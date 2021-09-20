**注意：**这个问题是Windows上的发现的


1. 打开MySQL目录下的`my.ini`文件，在文件的最后添加一行`skip-grant-tables`，保存并关闭文件;
1. 重启MySQL服务;
1. 通过cmd行进入MySQL的bin目录，输入`mysql -u root -p`(不输入密码)，回车即可进入数据库;
1. 执行`use mysql;`，使用mysql数据库;
1. 执行`update user set password=PASSWORD("123456") where user='root';`（修改root的密码）;
1. 打开MySQL目录下的my.ini文件，删除最后一行的`skip-grant-tables`，保存并关闭文件;
1. `mysql> flush privileges;`
1. `mysql> quit;`
1. 在命令行中输入`mysql -u root -p 123456`，即可成功连接数据库。
