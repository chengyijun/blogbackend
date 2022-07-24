---
title: docker常用命令
date: 2022-07-15 11:14:39
tags:
---



# docker常用命令

镜像加速

对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：

```json
{"registry-mirrors":["https://registry.docker-cn.com"]}
```

之后重新启动服务：

$ **sudo** systemctl daemon-reload
$ **sudo** systemctl restart docker

启动docker服务

> systemctl start docker

查找镜像

> docker search 镜像名

下载镜像

> docker pull 镜像名

查询已下载的镜像

> docker images

删除镜像

> docker rmi 镜像id

启动容器

> docker run --name centos-test -dit centos:latest

停止容器

> docker start 容器名/容器id

删除容器

> docker rm 容器名/容器id

查看容器

> docker ps -a

进入容器

> docker exec -it 容器名 /bin/bash
