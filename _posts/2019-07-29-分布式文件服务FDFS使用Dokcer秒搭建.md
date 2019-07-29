---
layout:     post
title:      分布式文件服务FDFS使用Dokcer秒搭建！
subtitle:   Docker搭建FDFS分布式文件服务
date:       2019-07-29
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - docker
---

> 在这里小小推荐下我的个人博客
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)

**关于FastDFS分布式文件服务器**

1. 相信大家在点开这篇文章的时候就已经对Docker有一些理解并能简单的应用。
2. 说实话，这东西我并不是很了解。
3. 不过还是按照文档一步一步的可以搭建起来这个服务器并实现上传下载的功能。
4. 但是在搭建的过程中会有很多的问题。
5. 因此耗费了不少的时间在系统的搭建上面。
6. 然后为了方便下次更换服务器时能够快速的搭建起我们的文件服务器，我选择使用docker的centos镜像将fdfs搭建起来并生成我的fdfs镜像，并将他开源发布到了DockerHub。

**以下是关于fdfs的开源文档**

这是一个简单pull即可使用的fdfs分布式文件系统镜像，内置运行nginx配合fdfs可实现http下载。

**相关目录**

> **fdfs相关配置挂载目录**
>
> /etc/fdfs/tracker.conf
>
> /etc/fdfs/storage.conf
>
> **nginx相关配置挂载目录**
>
> /etc/fdfs/mod_fastdfs.conf
>
> /opt/nginx/conf/nginx.conf
>
> **fdfs相关数据挂载目录**
>
> /fastdfs/storage/data
>
> /fastdfs/tracker

**拉取方式**

```shell
$ docker pull ly15326047083/fastdfs
```

**相关环境变量**

```properties
# nginx 监听ip
NGINX_IP=127.0.0.1
# nginx 中监听端口
FDFS_PORT=80 
# tracker服务ip
TRACKER_IP=127.0.0.1
# tracker服务端口
TRACKER_PORT=22122
# strage服务端口
STORAGE_PORT=23000
# 超时时间
CONNECT_TIMEOUT=10
```

**使用方式**

```shell
# 挂载配置
$ docker run -d -v /宿主机路径/tracker.conf:/etc/fdfs/tracker.conf -v /宿主机路径/storage.conf:/etc/fdfs/storage.conf --name 自定义容器名称 ly15326047083/fdfs:1.0
# 挂载数据
$ docker run -d -v /宿主机路径/storage/data:/fastdfs/storage/data --name 自定义容器名称 ly15326047083/fdfs:1.0
# 使用环境变量
$ docker run -d -e "FDFS_PORT=80" --name fdfs ly15326047083/fdfs:1.0

# 使用示例
$ docker run -d -p 80:80 -p 22122:22122 -p 23000:23000 -e "NGINX_IP=192.168.2.23" -e "TRACKER_IP=192.168.2.23" -e "FDFS_PORT=80" -e "TRACKER_PORT=22122" -e "STORAGE_PORT=23000" -e "CONNECT_TIMEOUT=10" --name fdfs
```

**最后说两句**

1. 作者对Docker有很浓厚的兴趣，那希望同样感兴趣的朋友们可以私我或者评论，我们多交流多沟通，互相促进，互相学习。
2. 除此之外呢，作为程序员的我当然对于算法分析以及Java、Python、Go同样有着浓厚的兴趣，相信我们可以在技术的道路上走的更远。
3. 对于Docker还要多说两句，作者最近在学习和应用docker-compose编排以及docker swarm集群部署，手头也有很多限制的服务器用来练手，希望同样感兴趣的同学们可以私我或者评论我们多多交流学习心得。
4. fdfs是我开源的第一个Docker Image如果大家有好的开源项目或者产品可以推荐给我哦。如果我的镜像中有什么做的不好的地方也希望大家可以指正。
5. 谢谢大家！！！