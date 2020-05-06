---
layout:     post
title:      docker的安装、镜像拉取以及启动第一个镜像tomcat
subtitle:   docker的安装、镜像拉取以及启动第一个镜像tomcat
date:       2019-08-14
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - docker

---

> 欢迎大家关注我的以下主页，尤其是今日头条！！！谢谢🙏🙏🙏
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>
> 今日头条：[来自底层程序员的仰望](https://www.toutiao.com/c/user/6132192948/#mid=1616456407686158)

### 开头说两句

1. 上一节内容呢，我们简单的介绍了一下Docker相关知识点，没有仔细了解过Docker相关知识点的小伙伴可以先去看一下上节内容。
2. 这一节内容呢，我们开始介绍一下Docker的安装、镜像的拉取以及简单的应用。
3. 然后呢，我会介绍两个常用的镜像仓库给大家。



### 正式开始

##### 这一节我们将介绍以下内容

1. linux下Docker的安装
2. Windows10下Docker的安装
3. MacOS下Docker的安装
4. 拉取一个开源镜像（tomcat官方镜像）
5. 启动我们的第一个镜像（tomcat官方镜像）
6. 简单介绍两个镜像仓库的使用（Docker Hub、网易镜像仓库）



#### linux下Docker的安装

1. centos系统

   ```shell
   # 更新源，下载docker官方安装脚本，运行脚本
   $ sudo yum update
   $ curl -fsSL https://get.docker.com -o get-docker.sh
   $ sudo sh get-docker.sh
   # 启动docker服务
   $ sudo systemctl start docker
   # 设置开机自动启动
   $ sudo systemctl enable docker
   ```

2. debian系统

   ```shell
   # 更新源，下载docker官方安装脚本，运行脚本
   $ sudo apt-get update
   $ curl -fsSL https://get.docker.com -o get-docker.sh
   $ sudo sh get-docker.sh
   # 启动docker服务
   $ sudo systemctl start docker
   # 设置开机自动启动
   $ sudo systemctl enable docker
   ```

3. ubuntu系统

   ```shell
   # 更新源，下载docker官方安装脚本，运行脚本
   $ sudo apt-get update
   $ curl -fsSL https://get.docker.com -o get-docker.sh
   $ sudo sh get-docker.sh
   # 启动docker服务
   $ sudo systemctl start docker
   # 设置开机自动启动
   $ sudo systemctl enable docker
   ```

4. 以上为三种系统的脚本自动安装，其他发行版的linux系统也都大同小异，大家可以自由发挥一下。

5. 当然大家也可以使用yum或者apt进行安装，但是国内网络不太顺畅，所以建议大家更换软件源后再进行安装，以免安装失败。



#### MacOS下Docker的安装

1. 最低系统版本macOS El Capitan 10.11，若你的mac系统版本太低建议首先升级系统版本。
2. 若你的mac安装了**Homebrew Cask**，你可以使用以下命令直接安装

