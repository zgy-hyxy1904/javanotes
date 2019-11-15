# docker学习笔记（了解到掌握）

## 第一天  2019/11/13

### ―、Docker简介

#### 1.**Docker**是什么？

产生背景：

- 开发和运维之间因为环境不同而导致的矛盾（不同的操作系统、软件环境、应用配置等）DevOps
- 集群环境下每台服务器都配置相同的环境，太麻烦
- 解决“在我的机器上可以正常工作”的问题

Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器 中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

阿里云、百度云等都支持Docker技术

官网：https://www.docker.com/

中文官网：<http://www.docker-cn.com/>

#### 2.Docker 作用

Docker是一种容器技术，使用Docker可以：

- 将软件环境安装并配置好，打包成一个镜像Image,然后将该镜像发布出去（Docker仓 库）
- 其他使用者可以在仓库中下载获取这个镜像
- 通过Docker运行这个镜像，就可以获取同样的环境（容器）

Docker简化了环境部署和配置，实现“一次构建，处处运行”，避免了因运行环境不一致而导致的异常

可以将Docker简单的认为是一个虚拟机，可以运行各种软件环境的虚拟机，但与传统虚拟机技术有所不同

Docke r容器技术与传统虚拟机技术的区别：

- 传统虚拟机技术：模拟一个完整的操作系统，先虚拟出一套硬件，然后在其上安装操作系

  统，最后在系统上再运行应用程序

  缺点：资源占用多，启动慢

- Docker容器技术：不是模拟一个完整的操作系统，没有进行硬件虚拟，而是对进程进行隔离，封装成容器，容器内的应用程序是直接使用宿主机的内核，且容器之间是互相隔离 的，互不影响

  优点：更轻便、效率高、启动快、秒级

## 第二天  2019/11/14

#### 3.基础术语

**术语：**

- Docker主机

安装了 Docker程序的主机，运行Docker守护进程

- Docker 镜像(Image)

将软件环境打包好的模板，用来创建容器的，一个镜像可以创建多个容器

- Docker容器(Container)

运行镜像后生成的实例称为容器，每运行一次镜像就会产生一个容器，容器可以启动、停止或删除

容器使用是沙箱机制，互相隔离，是独立是安全的

可以把容器看作是一个简易版的Linux环境，包括用户权限、文件系统和运行的应用等

- Docker 仓库(Repository)

用来保存镜像的，仓库中包含许多镜像，每个镜像都有不同的标签Tag

官方仓库：https://hub.docker.com

**使用Docker的步骤：**

1. 安装Docker
2. 从Docker仓库中下载软件对应的镜像
3. 运行这个镜像，此时会生成一个Docker容器
4. 对容器的启动/停止就是对软件的启动/停止

### 二，准备Linux系统

1. **安装虚拟机软件**

VMware、VirtualBox

Ubuntu、RHEL、CentOS、Debian、SLES (SUSE)

2. **新建虚拟机**

版本：Red Hat (64-bit)

网络：桥接网卡

光驱：选择操作系统的is。文件

3. **安装** **CentOS**

4. **连接Linux服务器**

步骤：

1. 查看服务器ip地址、

在虚拟机中执行ip addr

2. 连接服务器

在客户端中执行ssh服务器账户@服务器地址，然后输入密码

注：SSH是Secure Shell的缩写，用于远程登陆访问的协议

**5.基本操作**

**5.1**常用命令

```
cat/proc/cpuinfo # 查看cpu信息

cat/proc/meminfo #查看内存信息

uname -r #查看内核信息，Docker要求CentOS必须是64位，且内核是3.10及以上 sudo reboot #重启,sudo表示以管理员root身份执行

sudo halt # 关机
```

**5.2**安装软件

使用yum,是一个基于rpm的软件包管理工具

用来安装软件包，可以自动解决软件包之间的依赖关系

```
yum install软件包名#安装

yum remove软件包名#卸载
```

### 三、**Docker**安装

1. **安装**

Docker版本：社区版CE、企业版EE

```
#更新yum库
yum update

#设置yum源
sudo yum install -y yum-utils device-mapper-persistent-data Ivm2

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 

sudo yum makecache fast

# 安装Docker-CE

yum -y install docker-ce
```

2. **启动/停止**

```
docker version # 查看版本

systemctI start docker # 启动

systemctI stop docker # 停止

systemctI status docker # 查看状态

systemctI restart docker # 重启

systemctI enable docker # 设置开机自动启动

# 验证，运行hello-world

docker run hello-world # 下载hello-world镜像并运行
```

3. **配置Docker的镜像加速**

使用阿里云提供的镜像加速（镜像仓库），也可以使用网易云等

步骤：

1. 注册并登陆"阿里云的开发者平台"[http://dev.aliyun.com](http://dev.aliyun.com/)

2. 查看专属的加速器地址
3. 配置自己的D ocker加速器

```
vi /etc/docker/daemon.json

{

"registry-mirrors": ["https://sswv6yx0.mirror.aliyuncs.com"]

}

systemctl daemon-reload

systemctl restart docker

```



### 四，docker的操作

输入docker可以查看Docker的命令用法，输入docker COMMAND --help查看指定命令的详细用法

1. **镜像操作**

| 操作        | 命令                             | 说明                                                      |
| :---------- | -------------------------------- | --------------------------------------------------------- |
| 查找        | docker search 关键字             | 可以在D ocker Hub网站查看镜像的详细信息，如镜像的 tag标签 |
| 抽取        | docker pull 镜像名:tag           | :tag表示软件的版本，如果不指定默认是latest                |
| 列表        | docker images                    | 查看所有本地镜像                                          |
| 获取元信 息 | docker inspect 镜像 id           | 获取镜像的元信息，详细信息                                |
| 删除        | docker rmi -f镜像id或镜像 名:tag | 删除指定的本地镜像，-f表示强制删除                        |

## 第三天  2019/11/15

1. **容器操作**

   | 操作             | 命令                                                         | 说明                                                         |
   | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | 运行             | docker run --name 容器 名-i -t -p主机端口 :容器 端口 -d -v主机目录:容 器目录:ro镜像id或镜像 名称:tag | --name指定容器名，名称自定义，如果不指定会自动命名；-i以交 互模式运行，即以交互模式运行容器；-t分配一个伪终端，即命令 行，通常组合使用-it ； -p指定端口映射，将主机端口映射到容器内 的端口； -d表示后台运行，即守护式运行容器；-v指定挂载主机目 录到容器目录，默认为rw读写模式，r。表示只读 |
   | 列 表            | docker ps -a -q                                              | 查看正在运行的容器，-a表示显示所有容器，-q表示只显示容器id   |
   | 启动             | docker start 容器 id 或容                                    | 启动容器                                                     |
   | 停 止            | docker stop容器id或容 器名称                                 | 停止正在运行的容器                                           |
   | 删 除            | docker rm -f容器id或容器名称                                 | 删除容器，-f表示强制删除                                     |
   | 日 志            | docker logs容器id或容 器名称                                 | 获取容器的日志                                               |
   | 在容 器 中 执 行 | docker exec -it 容器 id 或容器名称/bin/bash                  | 进入正在运行的容器中并开启一个交互模式的终端，可以在容器中执 行操作 |
   | 拷 贝 文 件      | docker cp主机中的文 件路径容器id或容器名 称:容器路径；docker cp容器id或容器名称:容 器中的文件路径主机路 径 | 将文件中的文件拷贝到容器中；将容器中的文件拷贝到主机中       |
   | 获 取 元 信 息   | docker inspect 容器 id                                       | 获取容器的元信息                                             |

以CentOS为例子：

```
docker search centos
docker pull centos
docker run --name mycentos -it centos:latest # 根据 centos:latest 镜像运行容器，并以交互模式进入 容器中
#实际上是在Docker容器中运行一个精简版的CentOS系统
exit #退出并关闭容器
docker ps -a
docker start mycentos
docker stop mycentos
docker rm mycentos
docker rm -f $(docker ps -aq) # 强制删除所有容器
```

**注意：docker容器内实际上是运行着一个精简版的Linux系统**

