# 什么是容器？


容器是一个标准的软件单元，它将代码及其所有依赖关系打包，以便应用程序从一个计算环境快速可靠地运行到另一个计算环境。Docker容器映像是一个轻量级，独立的可执行软件包，包含运行应用程序所需的一切：代码，运行时，系统工具，系统库和设置。


容器映像在运行时成为容器，在Docker容器的情况下 - 映像在[Docker Engine](https://www.docker.com/products/docker-engine)上运行时成为容器。适用于基于Linux和Windows的应用程序，无论基础架构如何，容器化软件都将始终运行相同。容器将软件与其环境隔离开来，并确保它可以统一运行，尽管开发和分段之间存在差异。


在Docker Engine上运行的Docker容器：


- **标准：** Docker创建了容器的行业标准，因此它们可以随处携带
- **轻量级：**容器共享机器的操作系统内核，因此不需要每个应用程序的操作系统，从而提高服务器效率并降低服务器和许可成本
- **安全：**应用程序在容器中更安全，Docker提供业界最强大的默认隔离功能



# Docker容器无处不在


Docker容器技术于2013年作为开源[Docker Engine推出。](https://www.docker.com/products/docker-enterprise)


它利用围绕容器的现有计算概念，特别是在Linux世界中，称为cgroups和名称空间的原语。Docker的技术是独一无二的，因为它专注于开发人员和系统运营商将应用程序依赖性与基础架构分离的要求。


Linux世界的成功推动了与微软的合作，将Docker容器及其功能引入Windows Server（有时也称为[Docker Windows容器）。](https://www.docker.com/windows-containers)


Docker及其开源项目Moby提供的技术已被所有主要数据中心供应商和云提供商所利用。其中许多提供商正在利用Docker来提供容器本地IaaS产品。此外，领先的开源无服务器框架利用Docker容器技术。


# 安装Docker CE


您可以根据需要以不同方式安装Docker CE：


- 大多数用户 [设置Docker的存储库](https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository)并从中进行安装，以便于安装和升级任务。这是推荐的方法。
- 有些用户下载RPM软件包并 [手动安装](https://docs.docker.com/install/linux/docker-ce/centos/#install-from-a-package)并完全手动管理升级。这在诸如在没有访问互联网的气隙系统上安装Docker的情况下非常有用。
- 在测试和开发环境中，一些用户选择使用自动 [便捷脚本](https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-convenience-script)来安装Docker。



### 使用存储库安装


在新主机上首次安装Docker CE之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。


#### 设置存储库


1. 安装所需的包。`yum-utils`提供了`yum-config-manager` 效用，并`device-mapper-persistent-data`和`lvm2`由需要 `devicemapper`存储驱动程序。```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

2. 使用以下命令设置**稳定**存储库。```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

3. 安装_最新版本_的Docker CE和containerd ：



```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```


如果提示接受GPG密钥，请验证指纹是否匹配`060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果匹配，则接受该指纹 。


4. 启动Docker。



```shell
$ sudo systemctl start docker
```


5. 通过运行`hello-world` 映像验证是否正确安装了Docker CE 。



```
$ sudo docker run hello-world
```


此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条信息性消息并退出。


Docker CE已安装并正在运行。您需要使用它`sudo`来运行Docker命令。继续[Linux postinstall](https://docs.docker.com/install/linux/linux-postinstall/)以允许非特权用户运行Docker命令和其他可选配置步骤。


# 以非root用户身份管理Docker


Docker守护程序绑定到Unix套接字而不是TCP端口。默认情况下，Unix套接字由用户拥有`root`，其他用户只能使用它`sudo`。Docker守护程序始终以`root`用户身份运行。


如果您不想在`docker`命令前加上`sudo`，请创建一个名为的Unix组`docker`并向其添加用户。当Docker守护程序启动时，它会创建一个可由该`docker`组成员访问的Unix套接字。


要创建`docker`组并添加您的用户：


1. 创建`docker`组。```
$ sudo groupadd docker
```

2. 将您的用户添加到该`docker`组。```
$ sudo usermod -aG docker $USER
```

3. 注销并重新登录，以便重新评估您的组成员身份。
如果在虚拟机上进行测试，则可能需要重新启动虚拟机才能使更改生效。
在桌面Linux环境（如X Windows）上，完全注销会话，然后重新登录。
4. 验证您是否可以运行`docker`命令`sudo`。```
$ docker run hello-world
```

此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条信息性消息并退出。
如果`sudo`在将用户添加到`docker`组之前最初使用Docker CLI命令，则可能会看到以下错误，表明您的`~/.docker/`目录由于`sudo`命令而创建的权限不正确。```none
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

要解决此问题，请删除`~/.docker/`目录（它会自动重新创建，但任何自定义设置都会丢失），或使用以下命令更改其所有权和权限：```
$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R
```




# 配置Docker以在启动时启动


大多数当前的Linux发行版（RHEL，CentOS，Fedora，Ubuntu 16.04及更高版本）用于[`systemd`](https://docs.docker.com/install/linux/linux-postinstall/#systemd)管理系统启动时启动的服务。Ubuntu 14.10及以下使用[`upstart`](https://docs.docker.com/install/linux/linux-postinstall/#upstart)。


`systemd`


```
$ sudo systemctl enable docker
```


要禁用此行为，请`disable`改用。


```
$ sudo systemctl disable docker
```


# 镜像加速器配置


这里以阿里的镜像加速器为例


1. [上阿里官网](www.dev.aliyun.com)
2. 登录之后会或得镜像加速器得地址



**安装／升级Docker客户端**


推荐安装1.10.0以上版本的Docker客户端，参考文档 [docker-ce](https://yq.aliyun.com/articles/110806)


**配置镜像加速器**


针对Docker客户端版本大于 1.10.0 的用户


您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器


1. 创建文件夹```shell
sudo mkdir -p /etc/docker
```

2. 创建配置文件并加入内容```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://g66wjbqs.mirror.aliyuncs.com"]
}
EOF
```

3. 重新加载```shell
sudo systemctl daemon-reload
```

4. 重启Docker```shell
sudo systemctl restart docker
```




# Docker 镜像操作


### 1) 拉取镜像


```shell
docker pull 镜像名:版本号
```


### 2) 列出本地镜像


`docker images [options]`


- `-a, --all` 显示所有图像（默认隐藏中间图像）
- `--digests` 显示摘要
- `-f, --filter filter` 根据提供的条件过滤输出
- `--format string` 使用Go模板的漂亮图像
- `--no-trunc` 不截断输出
- `-q, --quiet` 仅显示数字ID
示例 :



```shell
[root@lemon ~]# docker images
REPOSITORY       TAG        IMAGE ID            CREATED             SIZE
tomcat           latest     5377fd8533c3        4 hours ago         506MB
hello-world      latest     fce289e99eb9        5 months ago        1.84kB
```


### 3) 删除镜像


`docker image rmi [OPTIONS] IMAGE [IMAGE...]`


- `-f， --force` 强制删除图像
- `--no-prune` 不要删除未标记的父母



可以用镜像的 **ID**来删除本地镜像


```shell
[root@lemon ~]# docker image rmi fce289e99eb9
Untagged: hello-world:latest
Untagged: hello-world@sha256:41a65640635299bab090f783209c1e3a3f11934cf7756b09cb2f1e02147c6ed8
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
```


# Docker 容器操作


### 1)启动容器


1. 有了镜像以后 , 以镜像为模板来启动容器
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]````shell
    --add-host list 添加自定义主机到IP映射（host：ip）
-a, --attach list 附加到STDIN，STDOUT或STDERR
    --blkio-weight uint16 块IO（相对权重），介于10和1000之间，或0表示禁用（默认为0）
    --blkio-weight-device list	阻塞IO权重（相对设备权重）（默认为[]）
    --cap-add list	添加Linux功能
    --cap-drop list	删除Linux功能
    --cgroup-parent string  容器的可选父cgroup
    --cidfile string  将容器ID写入文件
    --cpu-period int  限制CPU CFS（完全公平调度程序）期间
    --cpu-quota int限制CPU CFS（完全公平调度程序）配额
    --cpu-rt-period int限制CPU实时周期（以微秒为单位）
    --cpu-rt-runtime int以微秒为单位限制CPU实时运行时间
-c, --  cpu-shares int CPU份额（相对权重）
    --cpus decimal CPU数
    --cpuset-cpus 字符串允许执行的CPU（0-3,0,1）
    --cpuset-mems string允许执行的MEM（0-3,0,1）
-d, --detach 在后台运行容器并打印容器ID
```

2. 启动 **现有** 的容器
`docker start [OPTIONS] CONTAINER [CONTAINER...]````shell
选项：
-a, --attach   附加STDOUT / STDERR并转发信号
    --detach-keys string  覆盖用于分离容器的键序列
-i, --interactive  附加容器的STDIN
```




### 2) 停止容器


`docker stop [OPTIONS] CONTAINER [CONTAINER...]`


```shell
选项：
   -t， --time int  在杀死之前等待停止的秒数（默认为10）
```


**强制停止**


`docker kill [OPTIONS] CONTAINER [CONTAINER...]`


```shell
选项：
   -s， --signal string  信号字符串发送到容器的信号（默认为“KILL”）
```


### 2) 查看容器


`docker ps [OPTIONS]`


```shell
-a, --all 显示所有容器（默认显示刚刚运行）
-f, --filter filter 根据提供的条件过滤输出
    --format string  使用Go模板的漂亮打印容器
-n, --int int  显示最后创建的容器（包括所有状态）（默认为-1）
-l, --slast  显示最新创建的容器（包括所有状态）
    --no-trunc  不截断输出
-q, --quiet  仅显示数字ID
-s, --size  显示总文件大小
```


### 3) 删除容器


`docker rm [OPTIONS] CONTAINER [CONTAINER...]`


```shell
OPTIONS：
   -f， --force 强制删除正在运行的容器（使用SIGKILL）
   -l， --link 删除指定的链接
   -v， --volume 删除与容器关联的卷
```


# Dockerfile 容器定制


镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。


Dockerfile 是一个文本文件，其内包含了一条条的**指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。


#### 以交互的方式进入容器


```shell
[root@lemon ~]# docker exec -it 17232e5d3546 bash
root@17232e5d3546:/usr/local/tomcat#
```


我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 `docker diff` 命令看到具体的改动。


# Docker-Compose


- 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。



可见，一个项目可以由多个服务（容器）关联而成，`Compose` 面向项目进行管理。


### 1) 安装 Dockers -Compose


下载文件`Docker-Compose` 到 `/usr/lcoal/bin/`下,并给予执行的权限


您可以使用常用命令来安装或升级Compose：


```shell
curl -L https://github.com/docker/compose/releases/download/1.23.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```


安装之后执行 `docker-compose version` 可以看到版本信息


```shell
[root@lemon ~]# docker-compose version
docker-compose version 1.24.1, build 4667896b
docker-py version: 3.7.3
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```


### 2) 使用 Docker-Compose


一个简单的入门实验 , 用Docker-Compose 启动一个 tomcat


1. 在`/usr/local/` 下创建目录 `docker/tomcat` ,进入该目录创建`docker-compose.yml`配置文件,内容如下```yam
version: '3'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
```

2. 在该目录下输入命令`docker-compose up -d` 就能启动一个tomcat容器 , 在浏览器输入tomcat的访问地址就可访问到tomcat首页
3. 输入命令 `docker-compose logs -f tomcat` 可查看日志
4. 输入命令`docker-compose down` 可删除容器 .
