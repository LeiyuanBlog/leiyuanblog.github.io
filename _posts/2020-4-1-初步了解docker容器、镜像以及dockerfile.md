---
layout:     post
title:      初步了解docker容器、镜像以及dockerfile
subtitle:   初步了解docker容器、镜像以及dockerfile
date:       2020-04-01
author:     LY
header-img: img/post-docker-bk.png
catalog: true
tags:
    - 专栏
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

### **开头说两句**

1. 从今天开始呢，我会从头开始详细的说明一下Docker的使用。
2. 虽说是详细的介绍，但是底层的东西我们不会介绍，想来大家也会明白为什么。（当然是因为不会了！！！）
3. 我们会从简介开始到安装、使用、映射、挂载以及镜像的制作包括，黑箱、Dockerfile。
4. 当然我们还会有更进一步的说明，环境变量、脚本、动态配置等等。
5. 最后我们会说一下compose编排以及如何利用Swarm实现集群以及集群服务的动态更新等等内容，如果大家感兴趣，可以持续关注哦！！！

### **正式开始**

#### **这一节我们将介绍以下内容**

1. 什么是Docker？
2. Docker镜像是什么？
3. Docker容器是什么？
4. 简单说一下Dockerfile。

### **什么是Docker？**

1. 首先，大家可以看一下Docker的图标。是一只大鲸鱼身上有很多集装箱。
2. 那通俗的讲，我们的服务器就像是那只大鲸鱼，而我们的Docker就像是集装箱。
3. 大家可以想一下集装箱解决了什么问题？他将每个集装箱中的环境相对隔离开来，这样我们的大货轮就可以运送不同种类的货物，而不用去根据不同的货物类型选择不同的货轮。
4. 对应到我们的开发中也就是说我们可以把我们的应用程序打包到”集装箱“Docker中，这样一来，我们可以在任何可以容纳”集装箱“Docker的环境中运行我们的应用，而不会收到其他应用的影响。

### **什么是Docker镜像？**

1. 一般来说，我们的操作系统分为内核以及用户空间。
2. 对于linux来说，内核启动后，会将root文件系统挂载到用户空间。
3. 而Docker镜像就相当于是一个root文件系统。
4. 比如较新的ubuntu18.04的官方镜像就包含了完整的一套ubuntu最小系统的root文件系统。
5. 由此可见Docker镜像就是一个特殊的文件系统，提供了容器运行时所需要的各项文件以及一些必备的参数。
6. 镜像的内容在构建之后不会发生任何改变。

### **什么是Docker容器？**

1. 在Docker中镜像与容器的关系，就像是Java中类与实例的关系一样。
2. 镜像是静态的定义而容器则是镜像运行的时候生成的实体。
3. 容器可以create、start、stop、remove等等，但是容器的基本操作是不会影响到镜像本身的，除非是commit操作覆盖了原有的镜像。
4. 容器实际上是系统中的一个进程，但是与直接在系统中执行的进程有所不同，容器进程拥有自己的独立的命名空间。因此容器可以拥有自己的root文件系统、网络、进程等等。所有初学的时候很容易将容器与虚拟机搞混。

### **什么是Dockerfile？**

1. Dockerfile是文本格式的配置文件，我们可以使用Dockerfile快速的构建我们自己的Docker镜像。

2. Dockerfile是由一行行的命令语句组成的，同样可以使用#添加相关注释提高可读性。

3. 一般而言Dockerfile的内容分为基础镜像、用户信息、操作指令以及容器启动时执行指令。

4. 例如，我们需要自己构建一个tomcat镜像：

   ```dockerfile
   # 第一行必须指定基于的容器镜像 
   FROM hub.c.163.com/public/centos:7.2-tools 
   # 维护者信息 
   MAINTAINER Lei emailAddress 
   # 下载tomcat到指定路径备用 
   ADD http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.21/bin/apache-tomcat-9.0.21.tar.gz /root/ 
   # 安装tomcat依赖环境java 
   RUN yum -y update \ 
   # 安装openjdk1.8 
   	&&yum -y install java-1.8.0-openjdk* \ 
   # 进入tomcat所在目录 
   	&&cd /root/ \ 
   # 解压tomcat 
   	&&tar -zxvf apache-tomcat-9.0.21.tar.gz \ 
   # 修改文件夹名称 
   	&&mv apache-tomcat-9.0.21 tomcat_8080 
   # 启动容器时执行脚本startup.sh 
   CMD ["bash","/root/startup.sh"]
   ```

   上述脚本内容如下

   ```shell
   #!/bin/bash # start tomcat bash /root/tomcat/bin/startup.sh tail -f /root/tomcat/logs/catalina.out
   ```

   

#### **最后说两句**

1. 作为程序员的我当然对于算法分析以及Java、Python、Go同样有着浓厚的兴趣，相信我们可以在技术的道路上走的更远。
2. 同样也希望大家多多关注我！！！
3. 谢谢大家！！！