---
layout:     post
title:      IDEA、Sonar集成使用
subtitle:   IDEA、Sonar集成使用
date:       2018-11-6
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - 自测
    - IDEA
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

# IDEA、Sonar集成使用

SonarQube是管理代码质量一个开放平台，可以快速的定位代码中潜在的或者明显的错误，下面将会介绍一下这个工具的安装、配置以及使用。

准备工作；

1、jdk（不再介绍）

2、sonarqube：http://www.sonarqube.org/downloads/

3、SonarQube+Scanner:https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-2.5.zip

4、mysql数据库（不再介绍）





# 一、安装篇

1.下载好sonarqube后，解压打开bin目录，启动相应OS目录下的StartSonar。如本文演示使用的是win的64位系统，则打开D:\sonar\sonarqube-5.3\sonarqube-5.3\bin\windows-x86-64\StartSonar.bat

2.启动浏览器，访问http://localhost:9000，如出现下图则表示安装成功。

![img](http://static.oschina.net/uploads/img/201610/21150614_SQyg.png)





# 二、配置篇

1.打开mysql，新建一个数据库。

2.打开sonarqube安装目录下的D:\sonar\sonarqube-5.3\sonarqube-5.3\conf\sonar.properties文件

3.在mysql5.X节点下输入以下信息（根据各自的数据库情况去填写）

```properties
sonar.jdbc.url=jdbc:mysql://172.16.30.228:3306/sonar
sonar.jdbc.username=root
sonar.jdbc.password=521410
sonar.sorceEncoding=UTF-8
sonar.login=admin
sonar.password=admin
```

url是数据库连接地址，username是数据库用户名，jdbc.password是数据库密码，login是sonarqube的登录名，sonar.password是sonarqube的密码

4.重启sonarqube服务，再次访问http://localhost:9000，会稍微有点慢，因为要初始化数据库信息

5.数据库初始化成功后，登录

6.按照下图的点击顺序，进入插件安装页面

![img](http://static.oschina.net/uploads/img/201610/21150614_GmPr.png)

7.搜索chinese Pack，安装中文语言包

![img](http://static.oschina.net/uploads/img/201610/21150614_rmM6.png)

8.安装成功后，重启sonarqube服务，再次访问http://localhost:9000/，即可看到中文界面

![img](http://static.oschina.net/uploads/img/201610/21150614_wB61.png)

三、idea使用篇

   \1. 打开File->Settings->Plugins,搜索sonar插件
![img](http://static.oschina.net/uploads/space/2016/1021/144831_B8bM_2395027.png)

2.点击图中第二个框起来的选项,在搜索框中输入sonar，出现一下界面
![img](http://static.oschina.net/uploads/space/2016/1021/145047_BDdt_2395027.png)

3.选择SonarLint,点击Install安装
![img](http://static.oschina.net/uploads/space/2016/1021/145119_qhhW_2395027.png)
![img](http://static.oschina.net/uploads/space/2016/1021/145151_jbUR_2395027.png)

4.安装完毕，点击Restart InteliJ IDEA
![img](http://static.oschina.net/uploads/space/2016/1021/145520_auru_2395027.png)

5.在maven中配置sonar：打开setting.xml配置文件，在其中加入如下代码：
​    

```xml
<profile>

      <id>sonar</id>

      <activation>

        <activeByDefault>true</activeByDefault>

      </activation>

      <properties>

        <sonar.jdbc.url>

               jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8

        </sonar.jdbc.url>

        <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>

        <sonar.jdbc.username>root</sonar.jdbc.username>

        <sonar.jdbc.password>521410</sonar.jdbc.password>

        <sonar.host.url>http://localhost:9000</sonar.host.url>

      </properties>

    </profile>
```

6.然后项目中就如出现如图所示，就能分析文件中的代码质量了
![img](http://static.oschina.net/uploads/space/2016/1021/145942_jUKX_2395027.png)

![img](http://static.oschina.net/uploads/space/2016/1021/150601_kMDw_2395027.png)

参考：http://www.cnblogs.com/qiaoyeye/p/5249786.html