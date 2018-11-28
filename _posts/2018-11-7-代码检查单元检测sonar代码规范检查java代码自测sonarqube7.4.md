---
layout:     post
title:      代码检查|单元检测|sonar代码规范检查|java代码自测|sonarqube7.4
subtitle:   代码检查|单元检测|sonar代码规范检查|java代码自测|sonarqube7.4
date:       2018-11-7
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - 代码规范
    - sonar
    - sonarqube
---

> 在这里小小推荐下我的个人博客
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>

#### 第一步下载最新版的sonarqube7.4

1.[官方下载地址](https://www.sonarqube.org/downloads/)，下载社区版，是开源免费的。

2.不知道什么原因我在官网下载特别慢，可能是因为资源不足的原因。[所以我在这里放一个备用链接](https://download.csdn.net/download/leiyuan2580/10769708)。当然这个链接是csdn的资源链接，需要收取一定的积分。

3.**<u>点击上方链接，关注我的csdn和简书，并私信截图发给我提供免费下载。</u>**

#### 第二步当然是解压并且启动服务

1.解压后呢，如图所示

[![iTDTvd.md.png](https://s1.ax1x.com/2018/11/07/iTDTvd.md.png)](https://imgchr.com/i/iTDTvd)

2.打开bin目录，选择适合当前操作系统的文件夹

![iTDOVP.png](https://s1.ax1x.com/2018/11/07/iTDOVP.png)

3.启动对应的服务启动程序即可

#### 第三步通过浏览器访问sonar服务器

1.打开浏览器进入http://locathost:9000

2.点击右侧的登陆按钮进行登陆，默认账号密码均为admin

3.到此算是服务启动完成

#### 第四步就是进行配置了

1.打开配置文件，它位于sonar目录下conf中，打开sonar.properties文件并加入如下代码

```properties
sonar.jdbc.url=jdbc:mysql://your_mysql_host:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.jdbc.username=your_username
sonar.jdbc.password=your_password
sonar.sorceEncoding=UTF-8
sonar.login=sonar的默认登录用户名为admin
sonar.password=sonar的默认登录密码为admin
```

2.对maven进行配置，打开maven的setting.xml文件，找到节点<profiles></profiles>并在该节点中加入如下代码

```xml
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.jdbc.url>
                    jdbc:mysql://your_mysql_url:3306/sonar
                </sonar.jdbc.url>
                <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
                <sonar.jdbc.username>your_mysql_username</sonar.jdbc.username>
                <sonar.jdbc.password>your_mysql_password</sonar.jdbc.password>
                <sonar.host.url>http://your_sonar_host:9000</sonar.host.url>
                <!-- your_sonar_host是你的服务器地址，如果你的服务在本机则使用localhost -->
            </properties>
        </profile>
```

#### 第五步打开项目，对代码进行检测

1.如果你使用的是idea的话按照下图进行操作即可

![操作流程](https://ws4.sinaimg.cn/large/006tNbRwly1fwzgg70zk3j31kw0zkwns.jpg)

2.如果不是的话，进入项目根目录，使用shell工具直接输入上图中的mvn命令效果也是相同的

#### 第六步就是查看结果了

1.命令运行完毕之后可以在服务器中查看进度，打开http://localhost:9000进入服务器，按图中步骤操作如下。

![查看结果](https://ws2.sinaimg.cn/large/006tNbRwly1fwzgossw28j31kw0zkal8.jpg)

2.接下来大家就可以畅快的查看自己代码中的bug、漏洞、异味啦。好好调整自己的代码吧！！！