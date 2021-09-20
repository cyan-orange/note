

# 配置 `yum` 数据源


1. `cd /etc` 进入/etc下创建一个目录`mkdir centOS-yum`用于保存原数据源。
1. `mv /etc/yum.repos.d/* /etc/centOS-yum/`将yum原数据源移动到新建的目录下。
1. `cd /etc/yum.repos.d/`进入yum数据源配置目录下，下载阿里的yum数据源 `wget http://mirrors.aliyun.com/repo/Centos-7.repo` 如果`wget`命令不生效，说明还没有安装`wget`工具，输入`yum -y install wget` 进行安装。
1. 依次执行yum源更新命令：
   - `yum clean all` #清理yum缓存
   - `yum makecache` #生成缓存
   - `yum update` #更新系统（可选）
   - 到此配置数据源完成。



# `rpm` 命令管理


## `rpm`安装软件包


- `rpm -ivh 包全名`
   - `-i`(install) 安装
   - `-v`(verbose)显示详细信息
   - `-h`(hash)显示进度
   - `--nodeps` 不检查依赖性(不推荐使用)
- 依赖性：安装当前包时显示依赖另一个包则要先安装另一个包
- 不推荐指定安装位置



## `prm`升级软件包


- `prm -Uvh 包全名`
   - `-U` (upgrade)升级
- 升级也需要软件包，当没有安装旧版本的软件包时则安装当前软件包，所以此命令也可以当作安装命令。



## `rpm` 卸载软件包


- `prm -e 包名`
   - `-e` (erase)卸载
   - `--nodeps` 不检查依赖性(不推荐使用)



## `rpm` 软件包查询


- `rpm -qa` #查询所有已安装的软件包。
   - `-q` (query)查询
   - `-a` (all)所有
- `rpm -q 软件包` #查询软件包是否安装
- `rpm -qi 包名` #查询软件包的信息
   - `-i` (information)信息
   - `-p` (package) 未安装的软件包信息
- `rpm -ql 包名` #查询软件包文件的安装位置。
   - `-l` (list)列表
   - `-p` (package) 未安装的软件包安装的文件位置
- `rpm -qf 系统文件` #查询该文件属于那个rpm软件包。
   - `-f` (file) 文件
- `rpm -V 软件包全名` #检验文件是否被修改



# `yum` 命令管理


## `yum` 查询


- `yum list` #查询`yum` 源中所有可用的软件包列表
- `yum search 关键字` #查询`yum` 源中和关键字相关的软件包列表
- `yum` 不支持本机安装的软件包查询，只能查询服务器上的软件包



## `yum`安装软件包


- `yum -y install 包名`
   - `-y` 自动回答yes
   - `install` 安装



## `yum`升级软件包


- `yum -y update 包名`
   - `-y` 自动回答yes
   - `update` 升级
- 注意：服务器一般不升级



## `yum` 卸载软件包


- `yum -y remove 包名`
   - `remove` 卸载



# 源码包的安装与卸载


## 源码包安装


- 源码包安装需要C语言编译器，所以要先安装gcc`yum -y install gcc`。
- 源码包必须指定安装位置，不然安装后文件会到处都是，不方便管理和卸载,源码包一般指定安装在`/usr/local/`下，源码包一般保存在`/usr/local/src/`下。
- 源码包安装流程：
   - 1、解压源码包并进入文件目录，执行`./configure --prefix=/usr/local/xxx`
      - `--prefix` 指定安装位置xxx，xxx会自动创建
   - 2、执行`make`，如果报错则执行`make clean` 清除缓存和临时文件，从头再来
   - 3、执行`make install` 编译安装，如果报错，执行`make clean` 清除缓存和临时文件，还要删除目录xxx，从头再来



## 源码包卸载


- 直接把安装的目录删除就可以了`rm -rf /usr/local/xxx`
