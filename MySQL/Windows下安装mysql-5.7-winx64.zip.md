# `下载mysql-5.7-winx64.zip`


- 官网下载并解压`mysql-5.7.25-winx64.zip`，然后`配置环境变量`
- 在解压的目录下创建`data`文件夹用于存放数据，再创建`my.ini`配置文件，内容如下:



```properties
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=F:\\software-school\\mysql-5.7.25
# 设置mysql数据库的数据的存放目录
datadir=F:\\software-school\\mysql-5.7.25\\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```


# 初始化MySQL
以管理员权限打开`cmd`窗口，输入以下命令
`mysqld --initialize --user=mysql --console` 初始化数据库，在data目录里生成数据文件，会生成一个默认密码, 记住这个密码用来修改密码：
```shell
  C:\WINDOWS\system32>mysqld --initialize --user=mysql --console
2019-02-20T08:15:24.348929Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-02-20T08:15:24.728509Z 0 [Warning] InnoDB: New log files created, LSN=45790
2019-02-20T08:15:24.815555Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2019-02-20T08:15:24.914895Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: ac968873-34e7-11e9-b7be-54e1ade440b0.
2019-02-20T08:15:24.949780Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2019-02-20T08:15:24.958325Z 1 [Note] A temporary password is generated for root@localhost: yDaJeiam:8<p
```


# 安装MySQL服务
`mysqld -install` 安装MySQL服务


```shell
C:\WINDOWS\system32>mysqld -install
Service successfully installed.
```


# 启动MySQL服务
`net start mysql` 启动MySQL服务


```shell
C:\WINDOWS\system32>net start mysql
MySQL 服务正在启动 .
MySQL 服务已经启动成功。
```


# 修改密码
`mysql -u root -p` 输入刚才的复制的默认密码，登陆mysql


```shell
C:\WINDOWS\system32>mysql -uroot -p
Enter password: ************
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.25

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('garcon');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>
```


`SET PASSWORD FOR 'root'@'localhost' = PASSWORD('garcon');` 修改密码


```shell
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('garcon');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```


