---
layout:     post
title:      Docker容器的常用命令——CRD、日志查询、伪终端
subtitle:   Docker容器的常用命令——CRD、日志查询、伪终端
date:       2020-04-03
author:     LY
header-img: img/post-docker-bk.png
catalog: true
tags:
    - docker
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

## Docker容器的常用命令——CSR、日志滚动查询、伪终端

### 开头说几句

1. 上一节中我们说了镜像的拉取以及启动我们的第一个web容器。
2. 相信大家由此已经可以大概的看出Docker的**方便**之处。
3. 在接下来的章节中这一**特点**会越来越明显的体现出来，大家多多期待。
4. 今天这节我们说一下容器中常用的几个命令，简单介绍如何使用，其他更多的操作方式我们会在后面逐步解析。

### Docker容器的CSR

1. 看到这里怕是有人要问了，什么是个**CSR**？
2. 我们这里的**C**表示**CREATE**，意为创建。也就是**创建容器**的意思。
3. 那**S**表示**STOP**，意为停止**运行中的容器**，但**保留容器内容**以及变更。
4. 最后的**R**表示**REMOVE**，意为删除容器，执行该操作后将**删除**容器以及其中的**所有内容**，且无法**恢复**需谨慎操作。

##### C——镜像启动、容器创建

1. 其实上一章节我们有说到镜像的启动，今天我们稍加详细的说一下有关参数。

2. 首先我们需要获取一下镜像列表`docker images`，然后使用列表中第一列的**镜像ID**创建并**启动容器**。

3. 当然我们也可以直接使用容器名称以及标签启动`docker run tomcat:latest`。

4. 接下来我们详细的说一下`docker run`的相关参数：

   ```shell
   -d, --detach=false         指定容器运行于前台还是后台，默认为false     
   -i, --interactive=false    打开STDIN，用于控制台交互    
   -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
   -u, --user=""              指定容器的用户    
   -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
   -w, --workdir=""           指定容器的工作目录   
   -c, --cpu-shares=0         设置容器CPU权重，在CPU共享场景使用    
   -e, --env=[]               指定环境变量，容器中可以使用该环境变量    
   -m, --memory=""            指定容器的内存上限    
   -P, --publish-all=false    指定容器暴露的端口    
   -p, --publish=[]           指定容器暴露的端口   
   -h, --hostname=""          指定容器的主机名    
   -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
   --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
   --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
   --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
   --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
   --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
   --device=[]                添加主机设备给容器，相当于设备直通    
   --dns=[]                   指定容器的dns服务器    
   --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
   --entrypoint=""            覆盖image的入口点    
   --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
   --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
   --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
   --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
   --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
   --net="bridge"             容器网络设置:  
                                bridge 使用docker daemon指定的网桥       
                                host    //容器使用主机的网络    
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
   --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
   --restart="no"             指定容器停止后的重启策略:  
                                no：容器退出时不重启    
                                on-failure：容器故障退出（返回值非零）时重启   
                                always：容器退出时总是重启    
   --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
   --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理    
   ```

5. 大家一看估计也要吓一跳了，这么多怎么记得住呢。其实完全不必要，我们只需要记住如下的一些常用参数即可，其他的了解认识一下。

6. 常用参数示例，这里我们以**Tomcat**镜像`058f892b0c46`举例说明：

   ```shell
   # 1.后台运行
   $ docker run -d 058f892b0c46
   # 2.后台启动并映射8080端口至宿主机80端口
   $ docker run -d -p 80:8080 058f892b0c46
   # 3.后台启动、映射8080至80、挂载/usr/local/tomcat/logs/至宿主机/root/tomcatLogs/:
   $ docker run -d -p 80:8080 -v /root/tomcatLogs/:/usr/local/tomcat/logs/ 058f892b0c46
   # 4.设置容器中环境变量
   $ docker run -d -e "DOCKER_TEST=我是环境变量测试，后续验证" 058f892b0c46
   # 5.通过环境变量文件指定容器的环境变量
   $ docker run -d --env-file=/root/envfile 058f892b0c46
   # 6.指定容器的主机名
   $ docker run -d -h docker_test 058f892b0c46
   # 7.链接其他容器，使容器间可以通信
   $ docker run -d -h test_001 058f892b0c46
   $ docker run -d -h test_002 --link=test_001 058f892b0c46
   # 8.指定容器名称
   $ docker run -d --name tomcat 058f892b0c46
   # 9.指定容器网络模式——使用宿主机网络，这样无需再映射端口，宿主机开放端口即位容器开放端口
   $ docker run -d --net="host" --name tomcat 058f892b0c46
   ```

7. 相信有这些参数已经基本够用，后续我们还会讲到其他的，大家持续关注即可。

##### S——容器停止

1. 我们的容器启动后，难免也会有需要停止的时候，虽然不多，但也无可避免。

2. 这个操作没有什么复杂的地方，只需两步简单操作即可停止运行中的容器。

3. 具体操作：

   ```shell
   # 获取运行中的容器列表
   $ docker ps
   # 获取的结果中第一列CONTAINER ID即为容器编号
   $ docker stop ${CONTAINER ID}
   # 高端操作！！！！停止所有运行中的容器
   $ docker stop $(docker ps -aq)
   ```

##### R——容器删除

1. 这步操作也没什么特别之处，但是一定要谨慎操作。
2. 该操作会删除容器在运行期间的所有变更，但是挂载至宿住的文件是不会变动的。
3. 该操作的前置步骤为**S**操作，即为必须先停止运行中的容器才可执行删除操作。
4. 出了手动处理，也可以在容器启动时设置为容器停止自动删除`docker run --rm=true 058f892b0c46`。
5. 细心的朋友一定发现我们没有添加后台运行的`-d`参数，因为`--rm`与该参数无法并存。
6. 还有一个高危操作！！！删除所有容器`docker rm $(docker ps -aq)`，**慎用、慎用！**可以搭配上面的**停止所有容器**的命令结合使用，快速删除无用容器。

### Docker伪终端的使用

1. 初用**Docker**的朋友应该都有这样的困惑，启动的时候有些配置没有做好，但是又懒得**stop、rm、run**这样操作。

2. 进入伪终端去手动修改最为方便，可以使用如下命令通过容器ID进入伪终端：

   ```shell
   # 先查询容器ID
   $ docker ps
   # 进入容器伪终端
   $ docker exec -it ${CONTAINER ID} bash
   ```

3. 也可以使用启动时指定的容器名称或系统自动生成的容器名称进入伪终端：

   ```shell
   # 若启动时没有指定容器名称，则需要先查询容器名称
   $ docker ps
   # 通过名称进入
   $ docker exec -it ${name} bash
   ```

4. 两者其实相同。只是搜索**Docker容器**的方式不同。

5. 进入伪终端就跟**linux**差不多了。但是基础容器一般都是**极致**阉割后的，为了保持**Docker Images**的"苗条身材"。

6. 但是基础的操作基本都还是可以操作的**vi,tail,cat**等这些基础操作。

7. 需要推出的时候输入`exit`即可。

8. 需要注意的是，我们在运行中的容器进行修改，只要不删除容器，修改的内容会一直保持，但是**Docker Images**是不会修改的。所以如果执行了`rm`操作。那么修改内容就会跟着消失了。

9. 在后面的章节中我们会说到容器修改的保存操作。

### Docker容器日志的滚动查看

1. 容器日志的查看方式有很多种，我们简单的说几种。
2. 我们可以直接在宿主机查看**Docker**容器的日志信息
3. 一般情况下，**Docker**的数据文件均位于`/var/lib/docker/`目录中，而容器数据位于该目录下的`containers/`目录中。
4. 这个目录中会列出所有的容器ID，我们需要进入对应容器ID的目录中。查看对应日志文件即可。日志文件命名规则为`${CONTAINER ID}-json.log`。
5. 我们也可以使用**docker**命令查看对应容器日志`docker logs -f ${CONTAINER ID}`或者是`docker logs -f ${CONTAINER NAME}`。
6. 我们还可以通过上面的方式进入伪终端，查看我们需要查看的具体日志。
7. 相信大家滚动查看方式已经了解，但我这里还是贴一下查看日志的**Linux**命令，`tailf ${logFile}`或者是使用`tail -f ${logFile}`，这两个命令没有什么区别。
8. 我们还可以通过**grep**对内容进行过滤，这些就不在这里说了，后续会输出一些**linux**相关的内容。

### 最后说几句

1. 今天的内容就到这里了，希望大家有所收获。同时也希望大家多多支持。
2. 废话不多说了。我们下章节见！！！