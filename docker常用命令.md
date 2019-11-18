# docker常用命令

## **docker系统操作**

1. 查看版本 docker version
2. 启动 systemctl start docker
3. 停止 systemctl stop docker
4. 查看状态 systemctl status docker
5. 重启 systemctl restart docker

## **镜像操作**

1. 查找 docker search 镜像的名字或者ID

2. 拉取 docker pull 镜像名字：版本号

3. 列表 docker images

   查看正在运行的容器 docker ps   加-a 查看历史记录 加-q 查看运行容器的ID

4. 获取元信息（详细信息）docker inspect 镜像的名字或ID

5. 删除 docker rmi -f  镜像的名字或者ID

## **容器操作**

1. 运行 docker run --name my名字 -it -p 8888:8080 -d 
2. 列表 docker ps -aq
3. 启动 docker start 容器的名字或者ID
4. 停止 docker stop 容器的名字或者ID
5. 删除 docker rm -f 容器的名字或者ID       docker rm -f $(docker ps -aq)
6. 日志 docker logs 容器的名字或者ID
7. 在容器中执行 docker exec -it 容器的名字或者ID /bin/bash
8. 拷贝文件（主机和容器之间）docker cp 文件 容器的名字或者ID：位置
9. 获取元信息（详细信息）docker inspect 容器的名字或者ID

## **Dockerfile操作**

### 1.创建镜像文件

FROM 设置基础镜像

MAINTAINET 指定作者

RUN 在构建镜像时执行的命令（下载vim wget软件）

ENV 设置环境变量

WORKDIR 设置进入容器时候的工作目录

VOLUME 设置挂载点（不用谢宿主机的目录，直接写容器的目录，宿主机的目录会自动创建）

CMD 在容器启动的时候执行的命令

ENTRYPOINT   在容器启动的时候执行的命令 和CMD有区别

COPY 拷贝文件/目录到镜像中

ADD   拷贝压缩包到镜像中，自动解压

EXPOSE 指定对外暴露的接口

### 2.**创建好Dockerfile文件后执行**

```
docker build -f Dockerfile -t bocloud/centos:v1 .
```

### 3.**查看镜像**

```
docker images
```

### 4.**创建并执行容器**

```
docker run --name centos_v1 -d 镜像名字或镜像id /bin/bash
```



