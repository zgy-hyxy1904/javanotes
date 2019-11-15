# docker常用命令

**docker系统操作**

1. 查看版本 docker version
2. 启动 systemctl start docker
3. 停止 systemctl stop docker
4. 查看状态 systemctl status docker
5. 重启 systemctl restart docker

**镜像操作**

1. 查找 docker search 镜像的名字或者ID

2. 拉取 docker pull 镜像名字：版本号

3. 列表 docker images

   查看正在运行的容器 docker ps   加-a 查看历史记录 加-q 查看运行容器的ID

4. 获取元信息（详细信息）docker inspect 镜像的名字或ID

5. 删除 docker rmi -f  镜像的名字或者ID