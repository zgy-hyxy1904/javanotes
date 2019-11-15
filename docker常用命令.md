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

**容器操作**

1. 运行 docker run --name my名字 -it -p 8888:8080 -d 
2. 列表 docker ps -aq
3. 启动 docker start 容器的名字或者ID
4. 停止 docker stop 容器的名字或者ID
5. 删除 docker rm -f 容器的名字或者ID       docker rm -f $(docker ps -aq)
6. 日志 docker logs 容器的名字或者ID
7. 在容器中执行 docker exec -it 容器的名字或者ID /bin/bash
8. 拷贝文件（主机和容器之间）docker cp 文件 容器的名字或者ID：位置
9. 获取元信息（详细信息）docker inspect 容器的名字或者ID