# 使用MySQL Yum存储库在Linux上安装MySQL


MySQL Yum存储库提供用于在Linux平台上安装MySQL服务器，客户端，MySQL Workbench，MySQL Utilities，MySQL路由器，MySQL Shell，Connector / ODBC，Connector / Python等的RPM软件包。 （并非所有软件包都可用于所有发行版; 有关详细信息，[请参阅 使用Yum安装其他MySQL产品和组件](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html#yum-install-components)）。


MySQL Yum存储库支持以下Linux发行版：


- 基于EL6和EL7的平台（例如，Oracle Linux，Red Hat Enterprise Linux和CentOS的相应版本）
- Fedora 28和29





## 一. 添加MySQL Yum存储库


首先，将MySQL Yum存储库添加到系统的存储库列表中。跟着这些步骤：

1. 访问(https://dev.mysql.com/downloads/repo/yum/)上的MySQL Yum存储库下载页面 。
2. 选择并下载适用于您的平台的发行包。
2. 使用以下命令安装下载的发行包，替换 platform-and-version-specific-package-name 为下载的包的名称：
```bash
sudo rpm -Uvh platform-and-version-specific-package-name.rpm
```


## 二. 选择发布系列
使用MySQL Yum存储库时，默认选择安装MySQL的最新GA版本。默认情况下启用最新GA系列（当前为MySQL 8.0）的子存储库，默认情况下禁用所有其他系列（例如，MySQL 5.7系列）的子存储库。


使用此命令查看MySQL Yum存储库中的所有子存储库，并查看哪些子存储库已启用或禁用（对于支持dnf的系统，请使用dnf替换 命令中的 yum）：
```
yum repolist all | grep mysql
```


要从最新的GA系列安装最新版本，无需进行任何配置。要从最新GA系列以外的特定系列安装最新版本，请在运行安装命令之前禁用最新GA系列的子存储库并启用特定系列的子存储库。如果您的平台支持 **_yum-config-manager_**或**_dnf config-manager_**命令，则可以通过发出以下命令来执行此操作，这些命令禁用8.0系列的子存储库并启用5.7系列的子存储库; 对于未启用dnf的平台：
```bash
sudo yum-config-manager --disable mysql80-community
sudo yum-config-manager --enable mysql57-community
```


对于支持dnf的平台：
```bash
sudo dnf config-manager --disable mysql80-community
sudo dnf config-manager --enable mysql57-community
```


除了使用**_yum-config-manager_**或 **_dnf config-manager_**命令外，您还可以通过手动编辑`/etc/yum.repos.d/mysql-community.repo` 文件来选择系列 。这是文件中发布系列的子存储库的典型条目：
```bash
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```


找到要配置的子存储库的条目，然后编辑该enabled选项。指定 enabled=0禁用子存储库，或 enabled=1启用子存储库。例如，要安装MySQL 5.7，请确保您具有enabled=0MySQL 8.0的上述子存储库条目，并且具有 enabled=1的5.7系列的条目：
```
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```


您应该只在任何时候为一个发布系列启用子存储库。当启用多个版本系列的子存储库时，Yum将使用最新的系列。


通过运行以下命令并检查其输出来验证是否已启用和禁用了正确的子存储库（对于启用dnf的系统，请使用dnf替换 命令中的 yum）：
```bash
yum repolist enabled | grep mysql
```


## 三. 安装MySQL
通过以下命令安装MySQL（对于支持dnf的系统，使用dnf替换命令中的 yum）：
```bash
yum install -y mysql-community-server
```


这将安装MySQL服务器的软件包以及其他所需的软件包。


## 四. 启动 MySQL 服务
使用以下命令启动MySQL服务器：
```
sudo service mysqld start
```


对于基于EL7的平台，这是首选命令：
```
sudo systemctl start mysqld.service
```


您可以使用以下命令检查MySQL服务器的状态：
```
sudo service mysqld status
```


对于基于EL7的平台，这是首选命令：
```
sudo systemctl status mysqld.service
```


## 五. 修改 root 本地账户密码
MySQL服务器初始化（从MySQL 5.7开始）：在服务器初始启动时，如果服务器的数据目录为空，则会发生以下情况：


- 服务器已初始化。
- 在数据目录中生成SSL证书和密钥文件。
- 该validate_password插件安装并启用。
- mysql 安装完成之后，生成的默认密码在 `/var/log/mysqld.log` 文件中。使用**_grep_** 命令找到日志中的密码。
```
sudo grep 'temporary password' /var/log/mysqld.log
```


通过使用生成的临时密码登录并为超级用户帐户设置自定义密码，尽快更改root密码：
```
mysql -uroot -p

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```


**注意**:
MySQL的 validate_password插件默认安装。
这将要求密码包含至少一个大写字母，一个小写字母，一个数字和一个特殊字符，并且密码总长度至少为8个字符。


## 设置开机启动
```
shell> systemctl enable mysqld
# 重载所有修改过的配置文件
shell> systemctl daemon-reload
```


## 添加远程登录用户
CentOS系统安装好MySQL后，默认情况下不支持用户通过非本机连接上数据库服务器，下面是解决方法：
1、在控制台执行
```bash
mysql -uroot -p
```
系统提示输入数据库root用户的密码，输入完成后即进入mysql控制台

2、选择数据库
```bash
use mysql;
```


开启远程连接 `root` 用户名 `%`所有人都可以访问 `password` 密码
```bash
--修改权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
--刷新权限
FLUSH PRIVILEGES;
```


3、重起mysql服务
```bash
service mysqld restart
```


如果执行完以上步骤，还是不能远程连接，那么我们需要查看服务器的防火墙是否开启
```bash
service iptables status
```


如果防火墙开启，请关闭
```bash
service iptables stop
```
到此就可以远程连接了！

## 设置默认编码为 utf8
mysql 安装后默认不支持中文，需要修改编码。
修改 `/etc/my.cnf` 配置文件，在相关节点下添加编码配置，如下：


```
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```


## 默认配置文件路径：
配置文件：`/etc/my.cnf`
日志文件：`/var/log/var/log/mysqld.log`
服务启动脚本：`/usr/lib/systemd/system/mysqld.service`
socket文件：`/var/run/mysqld/mysqld.pid`
