---
layout:     post
title:      Docker容器Link操作
subtitle:   Docker容器Link操作
date:       2020-04-04
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

### Docker容器Link操作

#### 开头说几句

1. 在上一章节中，我们介绍了**Docker容器**的一些基本操作。
2. 在这一章节中我们来说说**Docker容器**的Link操作。
3. 往后的章节中我们会说到**Docker编排工具**以及**集群**，希望大家多多支持。
4. 最近欠更啊，真的希望朋友们能跟我聊聊，最近压力很大。

#### 正式开始

##### 为什么使用Docker的Link操作

1. 无论是什么应用服务，一定都会依赖到外部的服务。比如**Mysql、Redis、Zookeeper**等等这些外部的服务。
2. 一般的非容器部署，各项服务都会有自己独立的**IP**，而且基本上都会有固定的**服务器地址**。
3. 当然容器中一样会有容器的独立IP，但是通常来说是动态分配的虚拟**IP**。
4. 那么在我们的应用服务中其实一样可以使用**IP**去进行服务的调用，但是每次容器重启会导致我们的服务根据新分配的**IP**去配置应用服务，很是麻烦。
5. 那通过我们这节说到的**Link**操作，相当于给主机加入了一个**主机名**或者是**域名**之类的配置，我们就可以根据定义的容器名称去调用服务了。
6. 如此依赖，即使依赖服务如何变化，都不会影响到我们应用服务。

##### Docker如何正确使用Link命令

1. 这里我们就用简单依赖**Mysql**服务的**Java**项目做演示。

2. 通常在我们的项目中，**Mysql**的相关参数会写到**Java**项目的相关配置当中：

   ```yml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://192.168.2.231:3306/wise_base_abutting?${JDBC_PAR}
       username: ${JDBC_USERNAME}
       password: ${JDBC_PASSWORD}
   ```

3. 如上方配置，我们通常使用IP进行配置，现在我们以字符`mysql`进行配置：

   ```yml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://mysql:3306/wise_base_abutting?${JDBC_PAR}
       username: ${JDBC_USERNAME}
       password: ${JDBC_PASSWORD}
   ```

4. 一般来说这样是肯定不能用的，我们先保持这样的配置，现在我们先启动**mysql**容器，并对外映射**3306端口**。`docker run -d -p 3306:3306 --name mysql mysql:latest`

5. 实际上如果只是其他容器通过**link**使用**mysql**服务，无需映射端口，这里我们方便可视化操作数据库，选择对应映射3306端口

6. 然后呢我们启动**tomcat服务器**容器，**Link**名为**mysql**的容器并对外映射**8080端口**。`docker run -d -p 8080:8080 --link mysql:mysql --name tomcat tomcat:latest`

7. 这样一来，两个容器之间就可以互相通信了，那接下来我们就把我们的**Java**项目部署到**tomcat**容器中进行测试。`docker cp test.war tomcat:/usr/local/tomcat/webapps`

8. 我们通过上面的命令将**war**包放入**webapps**目录下，**tomcat服务器**会自动执行热加载。

9. 接下来我们就只需要关注项目是否正常启动运行，观察日志中是否有**Mysql数据库**无法连接的异常信息即可。`docker logs -f tomcat`

#### 最后说两句

1. 在最后呢，我再唠叨两句。
2. 以上所讲呢，适用于单容器启动，且依赖服务不算太多的情况下使用。
3. 但是有很多的弊端，比如用起来麻烦、操作繁琐等等。
4. 后面我们会讲到容器编排以及集群方式，那个用起来就舒服的太多了。
5. 希望大家多多支持，谢谢大家。