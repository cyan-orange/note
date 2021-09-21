# 分页查询优化

当表中的数据特别多的时候，一次性查出所有数据就特别慢。这个时候就要使用分页查询

一般分页查询语句为：

```SQL
select table from column limit start pageSize;
```



简单的分页查询语句查询越靠后的数据时速度越慢。

**优化方法一：** 使用子查询优化，这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

```sql
SELECT * FROM `user` WHERE id>=(SELECT id FROM `user` LIMIT 1000000,1) LIMIT 20
```



**优化方法二：** 使用id限定优化，如果表的id是连续自增的，根据查询的页数和查询的记录数可以算出查询的id的范围，可以使用 id between and 来查询

```sql
SELECT * FROM `user` WHERE `id`  between 1000001 and 1000101 limit 100;
```

第二种写法

```sql 
select * from `user` where id >= 1000001 limit 100;
```



**优化方法三：** 连接临时表优化，这种方式适用于id没有递增的情况

```sql
SELECT
	* 
FROM
	`user` u1
	JOIN ( SELECT id FROM `user` LIMIT 20, 5 ) u2 ON u1.id = u2.id;
```



# Explain



explain能解释mysql如何处理SQL语句，表的加载顺序，表是如何连接，以及索引使用情况。是SQL优化的重要工具

在 SQL 语句前加 Explain 关键字就可以查看 SQL 的执行计划。

```sql
mysql> explain select * from user\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 116
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```



个字段的含义：

- id: SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符.
- select_type: SELECT 查询的类型.
- table: 查询的是哪个表
- partitions: 匹配的分区
- type: join 类型
- possible_keys: 此次查询中可能选用的索引
- key: 此次查询中确切使用到的索引.
- ref: 哪个字段或常数与 key 一起被使用
- rows: 显示此查询一共扫描了多少行. 这个是一个估计值.
- filtered: 表示此查询条件所过滤的数据的百分比
- extra: 额外的信息

## id

SELECT识别符。这是SELECT查询序列号。查询序号即为sql语句执行的顺序。

1. id相同，执行顺序从上之下
2. id不同，执行顺序从大到小
3. id有相同的和不同的，遵守1、2规则



## select_type



select类型，它有以下几种:

最常见的查询类别应该是 `SIMPLE` 了, 比如当我们的查询没有子查询, 也没有 UNION 查询时, 那么通常就是 `SIMPLE` 类型, 例如:

```sql
mysql> explain select * from user
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | user  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  116 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```



如果我们使用了 UNION 查询, 那么 EXPLAIN 输出 的结果类似如下:

```
mysql> EXPLAIN SELECT * FROM `user` WHERE id=1
    -> UNION
    -> SELECT * FROM `user` WHERE id=2;
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
| id | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
|  1 | PRIMARY      | user       | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL            |
|  2 | UNION        | user       | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL            |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
3 rows in set, 1 warning (0.00 sec)

mysql>
```



## table

表示查询涉及的表或衍生表



## type

`type`  字段比较重要, 它提供了判断查询是否高效的重要依据依据. 通过 `type` 字段, 我们判断此次查询是 `全表扫描` 还是 `索引扫描` 等.

type 常用的取值有：

1. `system`: 表中只有一条数据. 这个类型是特殊的 `const` 类型.

2. `const`: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.
   例如下面的这个查询, 它使用了主键索引, 因此 `type` 就是 `const` 类型的.

   ```sql
   mysql> explain select * from user where id = 2;
   +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
   | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
   +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
   |  1 | SIMPLE      | user  | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL  |
   +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
   1 row in set, 1 warning (0.00 sec)
   ```



3.  `eq_ref`: 此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 `=`, 查询效率较高. 例如:

   ```sql
   mysql> EXPLAIN SELECT * FROM `user` u,`order` o  WHERE u.id=o.user_id;
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   | id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref            | rows | filtered | Extra       |
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   |  1 | SIMPLE      | o     | NULL       | index  | NULL          | PRIMARY | 16      | NULL           |    1 |   100.00 | Using index |
   |  1 | SIMPLE      | u     | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | test.o.user_id |    1 |   100.00 | NULL        |
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   2 rows in set, 1 warning (0.00 sec)
   ```

   

4. `ref`: 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 `最左前缀` 规则索引的查询.
   例如下面这个例子中, 就使用到了 `ref` 类型的查询:

   ```sql
   mysql> EXPLAIN SELECT * FROM `user` u,`order` o  WHERE u.id=o.user_id AND o.id=1;
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   | id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref            | rows | filtered | Extra       |
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   |  1 | SIMPLE      | o     | NULL       | ref    | PRIMARY       | PRIMARY | 8       | const          |    1 |   100.00 | Using index |
   |  1 | SIMPLE      | u     | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | test.o.user_id |    1 |   100.00 | NULL        |
   +----+-------------+-------+------------+--------+---------------+---------+---------+----------------+------+----------+-------------+
   2 rows in set, 1 warning (0.00 sec)
   ```

   

5. `range`: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.
   当 `type` 是 `range` 时, 那么 EXPLAIN 输出的 `ref` 字段为 NULL, 并且 `key_len` 字段是此次查询中使用到的索引的最长的那个。例如下面这个例子

   ```sql
   mysql> EXPLAIN SELECT * FROM `user` WHERE id BETWEEN 2 AND 12;
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   |  1 | SIMPLE      | user  | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL |   11 |   100.00 | Using where |
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   1 row in set, 1 warning (0.00 sec)
   ```



6. `index`: 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.
   `index` 类型通常出现在: 所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据. 当是这种情况时, Extra 字段 会显示 `Using index`。例如：

   ```sql
   mysql> EXPLAIN SELECT id FROM `user`;
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   |  1 | SIMPLE      | user  | NULL       | index | NULL          | PRIMARY | 8       | NULL |  116 |   100.00 | Using index |
   +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
   1 row in set, 1 warning (0.00 sec)
   ```

   上面的例子中, 我们查询的 name 字段恰好是一个索引, 因此我们直接从索引中获取数据就可以满足查询的需求了, 而不需要查询表中的数据. 因此这样的情况下, type 的值是 `index`, 并且 Extra 的值是 `Using index`.



7. ALL: 表示全表扫描, 这个类型的查询是性能最差的查询之一. 通常来说, 我们的查询不应该出现 ALL 类型的查询, 因为这样的查询在数据量大的情况下, 对数据库的性能是巨大的灾难. 如一个查询是 ALL 类型查询, 那么一般来说可以对相应的字段添加索引来避免.
   下面是一个全表扫描的例子, 可以看到, 在全表扫描时, possible_keys 和 key 字段都是 NULL, 表示没有使用到索引, 并且 rows 十分巨大, 因此整个查询效率是十分低下的.

   ```sql
   mysql> EXPLAIN SELECT name FROM `user`;
   +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
   | id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
   +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
   |  1 | SIMPLE      | user  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  116 |   100.00 | NULL  |
   +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
   1 row in set, 1 warning (0.00 sec)
   ```

   

通常来说, 不同的 type 类型的性能关系如下:

`ALL < index < range ~ index_merge < ref < eq_ref < const < system`

`ALL` 类型因为是全表扫描, 因此在相同的查询条件下, 它是速度最慢的..而 `index` 类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比 ALL 类型的稍快.

后面的几种类型都是利用了索引来查询数据, 因此可以过滤部分或大部分数据, 因此查询效率就比较高了.



## possible_keys

`possible_keys` 表示 MySQL 在查询时, 能够使用到的索引. 注意, 即使有些索引在 `possible_keys` 中出现, 但是并不表示此索引会真正地被 MySQL 使用到. MySQL 在查询时具体使用了哪些索引, 由 `key` 字段决定.



## key

此字段是 MySQL 在当前查询时所真正使用到的索引.



## key_len

表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.
key_len 的计算规则如下:

- 字符串
  - char(n): n 字节长度
  - varchar(n): 如果是 utf8 编码, 则是 3 *n + 2字节; 如果是 utf8mb4 编码, 则是 4* n + 2 字节.
- 数值类型:
  - TINYINT: 1字节
  - SMALLINT: 2字节
  - MEDIUMINT: 3字节
  - INT: 4字节
  - BIGINT: 8字节
- 时间类型
  - DATE: 3字节
  - TIMESTAMP: 4字节
  - DATETIME: 8字节
- 字段属性: NULL 属性 占用一个字节. 如果一个字段是 NOT NULL 的, 则没有此属性.



举个例子：

```sql
mysql> EXPLAIN SELECT * FROM order_info WHERE user_id < 3 AND product_name = 'p1' AND productor = 'WHH' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: order_info
   partitions: NULL
         type: range
possible_keys: user_product_detail_index
          key: user_product_detail_index
      key_len: 9
          ref: NULL
         rows: 5
     filtered: 11.11
        Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```



上面的例子是从表 order_info 中查询指定的内容, 而我们从此表的建表语句中可以知道, 表 `order_info` 有一个联合索引:

```sql
KEY `user_product_detail_index` (`user_id`, `product_name`, `productor`)
```

不过此查询语句 `WHERE user_id < 3 AND product_name = 'p1' AND productor = 'WHH'` 中, 因为先进行 user_id 的范围查询, 而根据 `最左前缀匹配` 原则, 当遇到范围查询时, 就停止索引的匹配, 因此实际上我们使用到的索引的字段只有 `user_id`, 因此在 `EXPLAIN` 中, 显示的 key_len 为 9. 因为 user_id 字段是 BIGINT, 占用 8 字节, 而 NULL 属性占用一个字节, 因此总共是 9 个字节. 若我们将user_id 字段改为 `BIGINT(20) NOT NULL DEFAULT '0'`, 则 key_length 应该是8.

上面因为 `最左前缀匹配` 原则, 我们的查询仅仅使用到了联合索引的 `user_id` 字段, 因此效率不算高.

在举个例子：

```sql
mysql> EXPLAIN SELECT * FROM order_info WHERE user_id = 1 AND product_name = 'p1' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: order_info
   partitions: NULL
         type: ref
possible_keys: user_product_detail_index
          key: user_product_detail_index
      key_len: 161
          ref: const,const
         rows: 2
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.00 sec)
```

这次的查询中, 我们没有使用到范围查询, key_len 的值为 161. 为什么呢? 因为我们的查询条件 `WHERE user_id = 1 AND product_name = 'p1'` 中, 仅仅使用到了联合索引中的前两个字段, 因此 `keyLen(user_id) + keyLen(product_name) = 9 + 50 * 3 + 2 = 161`



## rows

rows 也是一个重要的字段. MySQL 查询优化器根据统计信息, 估算 SQL 要查找到结果集需要扫描读取的数据行数.
这个值非常直观显示 SQL 的效率好坏, 原则上 rows 越少越好.



## Extra

EXplain 中的很多额外的信息会在 Extra 字段显示, 常见的有以下几种内容:

1. Using filesort
   当 Extra 中有 `Using filesort` 时, 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果. 一般有 `Using filesort`, 都建议优化去掉, 因为这样的查询 CPU 资源消耗大.

例如下面的例子:

```sql
mysql> EXPLAIN SELECT * FROM order_info ORDER BY product_name \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: order_info
   partitions: NULL
         type: index
possible_keys: NULL
          key: user_product_detail_index
      key_len: 253
          ref: NULL
         rows: 9
     filtered: 100.00
        Extra: Using index; Using filesort
1 row in set, 1 warning (0.00 sec)
```

索引是

```sql
KEY `user_product_detail_index` (`user_id`, `product_name`, `productor`)
```

但是上面的查询中根据 `product_name` 来排序, 因此不能使用索引进行优化, 进而会产生 `Using filesort`.

如果我们将排序依据改为 `ORDER BY user_id, product_name`, 那么就不会出现 `Using filesort` 了. 例如:

```sql
mysql> EXPLAIN SELECT * FROM order_info ORDER BY user_id, product_name \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: order_info
   partitions: NULL
         type: index
possible_keys: NULL
          key: user_product_detail_index
      key_len: 253
          ref: NULL
         rows: 9
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.00 sec)
```

2. Using index
   "覆盖索引扫描", 表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, 往往说明性能不错

3. Using temporary
   查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化.
