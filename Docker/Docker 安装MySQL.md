Docker安装MySQL5.7

# 拉取MySQL 镜像
```shell
docker pull mysql:5.7
```

# 启动容器
```shell
docker run -p 3306:3306 --name mysql -v /mydata/mysql/log:/var/log/mysql -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=orange -d mysql:5.7
```
参数说明
-p 3306:3306：将容器的3306端口映射到主机的3306端口，左边是主机端口，右边是容器端口。
--name：给容器取名。
-v /mydata/mysql/log:/var/log/mysql：将日志文件挂载到主机。
-v /mydata/mysql/data:/var/lib/mysql：将数据文件挂载到主机。
-v /mydata/mysql/conf:/etc/mys：将配置文件挂载到主机。
-e MYSQL_ROOT_PASSWORD=orange：初始化root账户密码。
-d： 后台启动。


# 修改配置
在挂载的目录`/mydata/mysql/conf`下创建**my.cnf**文件，写入MySQL的相关配置
```shell
vim my.cnf
```


