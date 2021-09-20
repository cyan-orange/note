# 什么是Redis


Redis 是用 C 语言开发的一个开源的高性能键值对（key-value）数据库。它通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止 Redis 支持的键值数据类型如下：


- 字符串类型
- 散列类型
- 列表类型
- 集合类型
- 有序集合类型



# Redis的使用场景


- 缓存（数据查询、短连接、新闻内容、商品内容等等）
- 分布式集群架构中的 session 分离
- 聊天室的在线好友列表
- 任务队列（秒杀、抢购、12306 等等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒）



# Redis高可用概述


在 `Web` 服务器中，**高可用** 是指服务器可以 **正常访问** 的时间，衡量的标准是在 **多长时间** 内可以提供正常服务（`99.9%`、`99.99%`、`99.999%` 等等）。在 `Redis` 层面，**高可用** 的含义要宽泛一些，除了保证提供 **正常服务**（如 **主从分离**、**快速容灾技术** 等），还需要考虑 **数据容量扩展**、**数据安全** 等等。


在 `Redis` 中，实现 **高可用** 的技术主要包括 **持久化**、**复制**、**哨兵** 和 **集群**，下面简单说明它们的作用，以及解决了什么样的问题：


- **持久化**：持久化是 **最简单的** 高可用方法。它的主要作用是 **数据备份**，即将数据存储在 **硬盘**，保证数据不会因进程退出而丢失。
- **复制**：复制是高可用 `Redis` 的基础，**哨兵** 和 **集群** 都是在 **复制基础** 上实现高可用的。复制主要实现了数据的多机备份以及对于读操作的负载均衡和简单的故障恢复。缺陷是故障恢复无法自动化、写操作无法负载均衡、存储能力受到单机的限制。
- **哨兵**：在复制的基础上，哨兵实现了 **自动化** 的 **故障恢复**。缺陷是 **写操作** 无法 **负载均衡**，**存储能力** 受到 **单机** 的限制。
- **集群**：通过集群，`Redis` 解决了 **写操作** 无法 **负载均衡** 以及 **存储能力** 受到 **单机限制** 的问题，实现了较为 **完善** 的 **高可用方案**。



# Redis 的安装


1. 去官网下载[redis源文件 ](https://redis.io/download) ,上传到Linux服务器上 , 解压到安装目录
2. 进入到安装目录执行`make`命令编译源文件
3. 执行命令`make install` 安装配置redis环境变量



## 启动Redis


前台启动方式


```shell
redis-server
```


后台启动方式


```shell
redis-server &
```


查看redis进程 `ps -ef | grep redis`


```shell
[root@lemon ~]# ps -ef | grep redis
root      11719   7346  0 19:47 pts/0    00:00:00 redis-server *:6379
root      11724   7346  0 19:48 pts/0    00:00:00 grep --color=auto redis
```


使用redis的命令行客户端连接redis服务器`redis-cli`


```shell
[root@lemon ~]# redis-cli
127.0.0.1:6379> set name xiejinchi
OK
127.0.0.1:6379> get name
"xiejinchi"
```


## 关闭Redis


- 关闭方式一 : 使用redis客户端向服务器发送命令关闭 (推荐使用)

```shel
redis-cli shutdown
```




- 关闭方式二 : 先用 命令`ps -ef | grep redis`查出进程号, 再用 `kill pid`关闭

```shell
[root@lemon ~]# ps -ef | grep redis
root      11798   7346  0 20:06 pts/0    00:00:00 redis-server *:6379
root      11803   7346  0 20:06 pts/0    00:00:00 grep --color=auto redis
[root@lemon ~]# kill 11798
[root@lemon ~]# 11798:signal-handler (1560082112) Received SIGTERM scheduling shutdown...
11798:M 09 Jun 2019 20:08:32.240 # User requested shutdown...
11798:M 09 Jun 2019 20:08:32.240 * Saving the final RDB snapshot before exiting.
11798:M 09 Jun 2019 20:08:32.241 * DB saved on disk
11798:M 09 Jun 2019 20:08:32.241 # Redis is now ready to exit, bye bye...
```


# Redis基本操作命令


## 1) 沟通命令,查看状态


输入`ping` , 返回PONG表示redis服务正常运行


```she
127.0.0.1:6379> ping
PONG
```


## 2) 查看当前数据库中 key 的数目 : dbsize


```shell
127.0.0.1:6379> set name xiejinchi
OK
127.0.0.1:6379> dbsize
(integer) 1
```


## 3) Reids默认使用16个库


Redis 默认使用16个库,从0到15. 对数据库个数的修改 , 在redis.conf 文件中


```tex
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```


## 4) 切换库


默认是使用第0个库, 可切换到其他库 `select index`


```shell
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]>
```


## 5) 清空当前库中的数据


flushdb将当前库中的数据全部删除


```shell
127.0.0.1:6379> flushdb
OK
```


## 6) 关闭当前redis连接


redis自带的客户端退出当前redis连接 : exit 或 quit


```shell
127.0.0.1:6379> exit
[root@lemon ~]#
```


## Key的操作命令


### 1) keys


语法: keys pattern


作用: 查找所有符合模式pattern的key , pattern可以使用通配符.


通配符 ;


1. * : 表示0~多个字符, 例如: keys * 查询所有的key .
2. ? : 表示单个字符, 例如 : wo?d ,匹配word , wood



### 2) exists


语法 : exists key[...]


作用 : 判断 key 是否存在


返回值 : 整数 , 存在key返回1 , 其他返回0 , 使用多个 key ,返回存在 key 的数量


### 3) expire


语法 : expire key seconds


作用 : 设置key的存活时间 , 超过时间的 key 自动删除 , 单位是秒


返回值 : 设置成功返回数字 1, 其他情况是 0


### 4) ttl


语法 : ttl key


作用 : 以秒为单位 , 返回key 的剩余生存时间(ttl : time to live)


返回值 :


1. -1 : 没有设置 key 的生存时间 , key 永不过期.
2. -2 : key 不存在
3. 数字 : key的剩余时间 , 秒为单位



### 5) type


语法 : type key


作用 : 查看 key 所存储值得数据类型


返回值 : 字符串表示数据类型


1. none (key不存在)
2. string(字符串)
3. list (列表)
4. set (集合)
5. zset (有序集)
6. hsh(哈希表)



### 6) del


语法 : del key [key...]


作用 : 删除存在得 key , 不存在的 key 忽略 .


返回值 : 数字, 删除 的 key 的数量


# Redis 的五种数据类型


## 1) 字符串 : String


字符串类型是redis 中最基本的数据类型 ,  它能存储任何形式的字符串 , 包括二进制数据, 序列化后的数据 , JSON化的对象甚至是一张图片 ,  最大**512M**


## 2) 哈希 hash


redis hash 是一个String类型的field 和 value 的映射表 , hash特别适合用于存储对象.


## 3) 列表 list


Redis 列表是简单的字符串列表 , 按照插入的顺序排序 ,  你可以添加一个元素到列表的头部(左边) 或者尾部 (右边)


## 4) 集合 set


Redis 的set 是String类型的无序集合 , 集合成员是唯一的, 即集合中不能出现重复的数据


## 5) 有序集合 zset


redis 有序集合zset 和集合 set 一样也是String类型元素的集合, 且不允许重复的成员 .  不同的是 zset 的每个元素都会关联一个分数 ( 分数可以重复) , redis 通过分数来为集合中的成员进行从小到大的排序 .


# 数据类型操作


## 1) 操作字符串类型


#### 基本命令


1. **set**



将字符串值 value 设置到 key 中


```shell
127.0.0.1:6379> set name xiejinchi
OK
```


2. get key 查询 key 对应的 value



```shell
127.0.0.1:6379> get name
"xiejinchi"
```


3. incr



将 key 中储存的数字值加 1 ,如果key 不存在则创建 , 初始值为 0 , 再执行incr 操作 (只对数字类型的数据操作) , 原子性的操作 , 是线程安全的


```shell
127.0.0.1:6379> set i 2
OK
127.0.0.1:6379> incr i
(integer) 3
```


4. decr



将key中存储的数字值减1 , 如果不存在则创建 ,初始值为 0 ,再执行 decr 操作 (只对数字类型的数据操作)


```shell
127.0.0.1:6379> decr de
(integer) -1
```


5. append



语法 : append key value
说明:  如果 key 存在, 则将 value 追加到 key 原来旧值的末尾 , 如果 key 不存在 , 则将 key 设置为value


返回值 : 返回追加后的字符长度


#### 常用命令


1. **strlen**



语法 : strlen key


说明 : 返回 key 所储存的字符串长度


返回值 : 如果 key 存在 ,返回字符的长度 , 如果不存在返回0


2. **getrange**



语法 : getrange key start end


作用 : 获取 key 中字符串值从start开始 到 end 结束 的子字符串 ,  包括start 和end , 负数 表示从字符串的末尾开始 , -1表示最后一个字符


返回值 : 截取的子字符串 .


3. **setrange**



语法 : **setrange key offset value**


说明 : 用value 覆盖(替换) key 的存储的值从offset开始 , 不存在的key 做空白字符串.


```shell
127.0.0.1:6379> set hello helloword
OK
127.0.0.1:6379> setrange hello 8 ld
(integer) 10
127.0.0.1:6379> get hello
"helloworld"
```


4. **mset**



语法 : mset key value [key value...]


说明 : 同时设置一个或多个 key-value 对


返回值 : ok


```shell
127.0.0.1:6379> mset k1 v1 k2 v2
OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
```


5. **mget**



语法 : mget key [key...]


作用 : 获取所有(一个或多个)给定 key 的值


返回值 :  包含所有key 对应的列表


## 2) 操作哈希类型


redis hash 是一个String类型的field 和value 的映射表. hash 特别合适用于存储对象.


#### 基本命令


1. **hset**



语法 : hset hash 表的key field value


作用 : 将哈希表key 中的域 field 的值设置为value , 如果key不存在, 则新建hash表, 再进行赋值, 如果有 field ,则覆盖值


返回值: 如果field是hash表中新field,且赋值成功 ,返回 1 . 如果field已经存在, 旧值被新值覆盖, 返回0


2. **hget**



语法 : hget key field


作用 : 获取哈希表 key 中给定域field 的值


返回值 : field 域的值, 如果key 不存在或者field不存在返回nil


3. **hmset**



语法 : hmset key field value [filed value...]


说明 :  同时将多个 filed-value(域-值) 设置到哈希表 key中, 此命令会覆盖已经存在的field, 如果hash 表 key 不存在 , 创建空的hash 表, 执行 hmset .


返回值 : 设置成功返回ok ,如果失败返回一个错误


4. **hmget**



语法 : hmget key field [field...]


作用 : 获取哈希表 key 中一个或多个给定域的值


返回值 : 返回和field顺序对应的值 , 如果field不存在 ,  返回nil


```shell
127.0.0.1:6379> hmget adriss imooc baidu
1) "www.imooc.com"
2) "www.baidu.com"
```


5. **hgetall**



语法 : hgetall key


作用 : 获取哈希表 kye 中所有的域和值


返回值: 以列表形式返回hash中域和域的值 , key 值不存在, 返回空hash


```shell
127.0.0.1:6379> hgetall adriss
1) "imooc"
2) "www.imooc.com"
3) "baidu"
4) "www.baidu.com"
```


6. hdel



语法 : hdel key field [field...]


作用 : 删除哈希表 key 中的一个或多个指定域field , 不存在field直接忽略


返回值 : 成功删除的field的数量


#### 常用命令


1. **hkeys**



语法 : hkeys key


作用 : 查看哈希表 key 中的所有filed 域


返回值 : 包含所有 field 的列表, key 不存在返回空列表


2. **hvals key**



语法 : hvals key


作用 : 返回哈希表中所有域的值


返回值 : 包含哈希表所有域值得列表 , key 不存在返回空列表


3. **hexists**



语法 : hexists key field


作用 : 查看哈希表 key 中, 给定 域 field 是否存在


返回值 : 如果field存在, 返回1 ,其他返回 0


## 3) 列表 list


redis 列表是简单得字符串列表 , 按照插入顺序排序 . 你可以添加一个元素列表的头部(左边)或者尾部(右边)


#### 基本命令


1. **lpush**



语法 : lpush key value [value ...]


作用: 将一个或多个值 value 插入到列表 key 的表头(最左边) , 从左边开始加入值,从左到右的顺序依次插入到表头


返回值 : 数字,新列表的长度


2. rpush



和 lpush 相对应


3. lrange



语法 : lrange key start stop


作用 : 获取列表 key 中指定区间内的元素 , 0 表示列表的第一个元素, 以 1 表示列表的第二个元素; start, stop 是列表的下标值, 也可以负数的下标, -1表示列表的最后一个元素 ,-2 表示列表的倒数第二个元素, 以此类推 . start, stop 超出列表的范围不会出现错误.


返回值 : 指定区间的列表


4. **lindex**



语法 : lindex key index


作用: 获取列表 key 中下标为指定 index 的元素 , 列表元素不删除, 只是查询, 0表示列表的第一个元素,  -1表示列表的最后一个元素


返回值 : 指定下标的元素 ; index不在列表范围, 返回nil


5. **llen**



语法 : llen key


作用 : 获取列表 key 的长度


返回值: 数值, 列表的长度; key 不存在返回 0


#### 常用命令


1. **lrem **



语法 : lrem key count value


作用 : 根据参数 count 的值 , 移除 列表中与参数 value相等的元素. count>0 , 从列表的左侧向右开始移除; count < 0 从列表的尾部开始移除; count=0 移除表中所有与value 相等的值


2. **lset**



语法 : lset key index value


作用 : 将列表key 下标为 index 的元素的值设置为value


返回值 : 设置成功返回 ok , key不存在或者 index 超出范围返回错误信息


3. **linsert**



语法 : linsert key Before|Alfter pivot value


作用 : 将值 value 插入到列表 key 当中位于 pivot之前或之后的位置 . key 不存在, pivot不存在列表中, 不执行任何操作


返回值 : 命令执行成功, 返回新列表的长度. 没有找到pivot 返回 -1 , key不存在返回 0


## 4) 集合类型 set


redis 的 set 的String类型的无序集合, 集合成员是唯一的, 即集合中不能出现重复的数据


#### 基本命令


1. **sadd**



语法 : sadd key member [member...]


作用 : 将一个或多个 member 元素加入到集合 key 当中 , 已经存在于集合的member元素将被忽略 ,不会再添加.


返回值 : 加入到集合的新元素的个数 . 不包括被忽略的元素.


2. **smembers**



语法 : smembers key


作用 : 获取集合 key 中的所有成员元素, 不存在的 key 视为空集合


3. **sismember**



语法 : sismember key member


作用 : 判断member 元素是否是集合 key 的成员


返回值 : member 是集合成员返回**1**, 其他返回 **0**


4. **scard**



语法 : scard key


作用 : 获取集合里面的元素个数


返回值 : 数字, key 的元素个数 . 其他情况返回 0


5. **srem**



语法 : srem key member [member...]


作用 : 删除集合 key 中的一个或多个 member 元素 , 不存在的元素被忽略 ;


返回值 : 数字,成功删除的元素个数 , 不包含忽略的元素


#### 常用命令


1. **srandmember**



语法 : srandmember key [count]


作用 : 只提供key , 随机返回集合中一个元素 , 元素不删除,依然再集合中 ; 提供了count 整正数, 返回包含count 个数元素的集合,  集合元素各不相同 ; count 是负数 ,返回一个count 绝对值的长度的集合 ,集合中元素可能会重复多次


返回值 : 一个元素 , 多个元素的集合


2. **spop**



语法 : spop key [count]


作用 : 随机从集合中删除一个元素, count 是删除元素的个数.


返回值 : 被删除的元素, key 不存在或空集合返回nil


## 5) 有序集合 zset


redis 有序集合zset 和集合 set 一样也是String类型元素的集合, 且不允许重复的成员 .  不同的是 zset 的每个元素都会关联一个分数 ( 分数可以重复) , redis 通过分数来为集合中的成员进行从小到大的排序 .


#### 基本命令


1. **zadd**



语法 : zadd key score member [score member...]


作用 : 将一个或多个member元素及其score 值加入到有序集合key 中, 如果member 存在集合中, 则更新值; score 可以是整数或浮点数;


返回值 : 数字 ,新添加的元素个数


2. **zrange**



语法 : zrange key start stop [Withscores]


作用 : 查询有序集合 , 指定区间内的元素. 集合成员按score值从小到大来排序. start 和stop都是从0 开始 . 0是第一个元素 , -1 是最后一个元素 , with scores选项让score 和value一同返回.


返回值 : 自定区间的成员集合


3. **zrevrange**



语法 : zrevrange key start stop [Withscores]


作用 : 返回有序集 key 中, 指定区间内的成员. 其中 成员的位置按score 值递减(从大到小)来排序 . 其他同zrange命令.


返回值 : 指定区间的成员集合


4. **zrem**



语法 : zrem key member [member ...]


作用 : 删除有序集合 key 中的一个或多个成员 , 不存在的成员被忽略


返回值 : 被成功删除的成员数量 , 不包括被忽略的成员 .


5. **zcard**



语法 : zcard key


作用 : 获取有序集 key 的元素成员的个数


返回值哦 : key 存在返回集合元素个数


#### 常用命令


1. **zreangebyscore**



语法 : zrangebyscore key min max [Withscores] [limit offset count]


作用 : 获取有序集 key 中 , 所有score值价于min 和max 之间 (包括min 和 max )的成员,有序成员是按递增(从小到大)排序.  min , max 是包括在内 , 使用符号 ' ( ' 表示不包括 . min , max 可以使用  -inf  , +inf 表示最大和最小    limit 用来限制返回结果的数量和区间  , Withscores用来显示 score和 value


返回值 : 指定区间的集合数据


2 . zrevreangebyscore


语法 : zrevrangebyscore key max min [Withscores] [limit offset count]


作用 : 和 **zreangebyscore** 一样 ,只是按大到小排序


3. zcount



语法 : zcount key min max


作用 : 返回有序集 key 中, score 值再 min 和 max 之间`(`默认包括score 值等于min 或max的成员数量


# Redis 事务


#### 1) multi


语法 : multi


作用 : 标记一个事务的开始 . 事务内的多条命令会按照先后顺序被放进一个队列当中 .


返回值 : 总是返回 ok


#### 2) exec


语法 : exec


作用 : 执行所有事务块内的命令


返回值 : 事务内的所有执行语句内容, 事务被打断(影响) 返回nil


#### 3) discard


语法 : discard


作用 :  取消事务 , 放弃执行事务块内的所有命令


返回值 : 总是返回  ok


# 数据持久化


Redis 对数据持久化有两种方式


## RDB 方式


**什么是 RDB 方式**


Redis Database (RDB) , 就是在指定的时间间隔内将内存中的数据集快照写入磁盘 , 数据恢复时将快照文件直接再读到内存,


RDB 保存了在某个时间点的数据集 (全部数据) 存储在一个二进制文件中 , 只有一个文件. 默认是 dump.rdb . RDB 技术非常适合做备份 , 可以保存最近一小时, 一天 , 一个月的全部数据. 保存数据是在单独的进程中写文件, 不影响redis 的正常使用 . RDB 恢复数据时比其他AOP速度快


**如何实现 RDB**


RDB方式的数据持久化 , 需要在redis.conf 文件中配置即可 ,默认配置是启用的 .


在配置文件redis.conf 中搜索 SNAPSHOTTING , 查找在注释开始和结束之间关于RDB的配置说明 . 配置 SNAPSHOTTING 地方有三处


1. 配置执行RDB生成快照文件的时间策略 . 例如 : 让它在 'N' 秒内数据集至少有 'M' 个 key 改动时自动保存一次数据



```tex
save N M
```


2. dbfilename: 设置RDB的文件名 ,默认文件名是 dump.rdb
3. dir : 指定RDB 文件的存储位置 , 默认是 ./  当前目录



## AOF 方式


**什么是AOF方式**


Append-Only file (AOF), Redis每次收到一条改变数据的命令时 , 它将把该命令写到一个AOF文件中(只记录写操作, 读操作不记录) . 当Redis重启动时, 它通过执行AOF文件中所有的 命令来恢复数据


**如何实现 AOF**


AOF方式的数据持久化 , 仅需要在redis.conf文件中配置,


配置项 :


1. appenddonly : 默认时no , 改成yes 开启了aof持久化
2. appendfilename : 指定AOF 文件名 ,默认文件名为appendonly.aof
3. dir : 指定RDB 和AOF文件的存放目录, 默认是 ./
4. appendfsync : 配置向aof文件写命令数据的策略;
   1. no : 不主动进行同步操作 , 而是完全交由操作系统来做(每30一次) , 比较块但不是很安全 .
   2. always : 每执行写入都会执行同步 , 慢一些但比较安全.
   3. exerysec : 每秒执行一次同步操作, 比较平衡 , 介于速度和安全之间 , 这是默认项
5. auto-aof-rewrite-min-size : 允许重重写的最小AOF文件大小, 默认64M , 当aof文件大于64M时, 开始整理aof文件, 去掉无用的操作命令,缩小aof文件.



# 悲观锁和乐观锁

在使用WATCH命令监控一个KEY后，当前队列中的命令会由于外部命令的执行而放弃，这是乐观锁的体现。


- 悲观锁
认为当前环境非常容易发生碰撞，所以执行操作前需要把数据锁定，操作完成后释放锁，其他操作才可以继续操作。
- 乐观锁
认为当前环境不容易发生碰撞，所以执行操作前不锁定数据，万一碰撞真的发生了，那么放弃自己的操作。