---
layout: post
title:  安装及使用 docker（一）
date:   2018-09-01 17:09:00 +0800
categories: docker
tag: note
---

* content
{:toc}


## 安装

[docker 官网](https://www.docker.com/)

使用 [Toolbox](https://docs.docker.com/toolbox/overview/) 安装

[网易蜂巢镜像](https://c.163yun.com/hub#/m/home/)

## 创建一个 docker-mechine

```
# 默认是会创建一个 default 的 machine
docker-machine ls

# 创建一个名为 newmachine 的 machine 
docker-machine create --driver virtualbox newmachine

# 查看 machine 的环境变量
docker-machine env newmachine

# 根绝提示执行
eval $(docker-machine env newmachine)

# 运行一个 docker 验证是否成功 - 使用的是网易蜂巢的镜像 - 镜像名为 busybox
docker run hub-mirror.c.163.com/library/busybox:latest echo hello world

# 删除 machine
docker-machine rm newmachine
```

## 通过 nginx 搭建静态网页

```
# 拉取 Nginx 镜像
docker pull hub-mirror.c.163.com/library/nginx:latest 

# 查看所有镜像
docker images

# -it 进入容器
docker run -p 8080:80 --name nginx_web -it hub-mirror.c.163.com/library/nginx /bin/bash

# 查看 nginx 位置
whereis nginx 

# 启动 nginx
nginx
```

此时访问 `192.168.99.100:8080` 就可以看到 nigix 启动成功了

```
# 退出容器 ctrl + p , ctrl + q
# 查看容器
docker ps -a

# 停止容器
docker stop <CONTAINER ID>
```

此时访问 `192.168.99.100:8080` 就无法访问了

## 修改静态文件
```
# 重新连接 docker 容器
docker attach <CONTAINER ID>

# 更新来源
apt-get update

# 安装 vim
apt-get install -y vim

# 使用 vim 编辑 index.html 文件
vi index.html
```

## docker 容器和宿主机互通文件

```
# 宿主机到容器
docker cp /workSpace/html <CONTAINER ID or NAME>:/usr/share/nginx

# 容器到宿主机
docker cp <CONTAINER ID or NAME>:/file/path /host/path
```

## 挂载宿主机文件夹到容器里面

启动一个 web 静态服务器

```
# 使用之前的 nginx 镜像
docker run -d -p 9999:80 -v /Users/frank/workSpace/html:/usr/share/nginx/html --name test_web -it hub-mirror.c.163.com/library/nginx /bin/bash
```
