---
layout:     post
title:      依赖很多单个Docker镜像无法满足？该说说docker-compose编排了
subtitle:   依赖很多单个Docker镜像无法满足？该说说docker-compose编排了
date:       2020-04-09
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

## 依赖很多单个Docker镜像无法满足？该说说docker-compose编排了

### 习惯性开头说几句！

1. 知道今天为止嘞，我们已经更新了八个章节了。
2. 我们已经从简单的**Docker**镜像拉取、镜像运行说到了镜像的**黑箱**、**dockerfile**制作再到数据持久、动态配置以及容器**Link**。
3. 这个章节呢，我们就来说说重点了（PS：敲黑板），编排工具：**docker-compose**。
4. 不知不觉我们已经说了这么多啦，非常高兴大家能够有所收获。
5. 这个过程不仅仅是知识的分享，同样也是对知识技能的巩固。
6. 同样的非常重要的一点就是非常感谢大家可以积极的交流，这是我最喜欢的一点。
7. 为了方便大家的沟通交流呢我也是创建了圈子**被仰望的程序员研习圈**哦，大家可以在里面分享知识、生活、情感等等。

### 正式开始

#### 为什么我们会用到编排工具——**docker-compose**

1. 当我们的项目比较小的时候，我们用到的可能只有**Java、Tomcat、Mysql**。
2. 但是随着我们的项目用户数的增加，并发量、系统负载都会随之提高，相应的系统的反应以及处理速度就会大大受限。这个时候我们的架构就会发生变化啦。
3. 这时候我们一般会加入其他的中间件，比如**Redis、Zookeeper、MQ**等等这些中间件，还有很多很多我们都可能会依赖到。
4. 一般来说我们不使用容器部署的话，一样不会把这些服务全部都部署到一台宿主机，这样会给服务器带来很大的负载并且会大大的影响中间件的性能。
5. 如此说来，我们更加不会将所有的服务都部署到同一个容器中了。即使宿主机的配置极高可以做到，那容器的大小无法想象会有多大。
6. 所以啊，如果我们需要的依赖服务增多，那我们就需要使用编排工具快捷的启动所有的容器来供我们的项目使用。

#### 开始前的准备工作——安装docker-compose

1. 相信如果大家看到了这里，那你一定安装过**Docker**了，而且我们最开始的章节就是介绍各种系统版本的**Docker**安装方式。
2. 这里呢我们就简单的说一下怎么安装**docker-compose**编排工具吧。
3. 其实非常的简单啦，两个命令就可以解决！！！
4. 第一步下载：`sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose`
5. 第二步赋权：`sudo chmod +x /usr/local/bin/docker-compose`
6. 当你看到了这里，那你已经学会了一大半了，厉害了！

#### 开始编写编排文件docker-compose.yml

1. 这里值得注意的一点问题，这个编排文件必须要叫**docker-compose.yml**，也就是说你如果没给他起这个名字，他就不高兴工作。
2. **yml**的格式相信大家知道，这种格式的配置文件结构层次更加明显，这是个人非常喜欢的一种配置格式。
3. 我们简单的介绍一下**docker-compose.yml**的一些常用属性，更多的属性介绍就需要大家自行查看了，这里不做过多的赘述，详细信息最下方我有做总结。
4. version：版本
5. services：需要启动的服务
6. image：镜像名称
7. container_name：启动后的容器名称。等同与**docker run**命令中的**--name**参数。
8. ports：需要开放的端口
9. networks：网络设置
10. network_mode：网络模式。等同与**docker run**命令中的**--network**参数。
11. environment：指定环境变量
12. volumes：挂载目录。等同与**docker run**命令中的**-v**参数。
13. ipv4_address：指定容器IP
14. 简单的就说这些，详细的大家查看最下方详解喽。

#### 简单的实例

1. 这里嘞，给出一个简单的实例，大家看看如上的参数到底如何使用。
2. 注意喽，这里我没有使用link而是直接指定容器的虚拟网段并且指定IP，个人觉得这样用起来更方便一点。
3. 接下来我们就使用`/usr/local/bin/docker-compose up`启动一下我们编排好的容器们。需要注意的是，我们所在的目录必须是**docker-compose.yml**所在的目录。
4. 启动后呢我们可以事实的看到运行容器的日志，但是大家会发现如果我们**control+c**退出日志之后呢，容器也相应停止了，所以如果我们需要保持容器的后台运行，我们需要加上**-d**参数。
5. `/usr/local/bin/docker-compose up -d`

```yaml
version: '2.1'
services:
  redis:
    image: hub.c.163.com/library/redis:latest
    container_name: redis-c
    command: redis-server --requirepass leiyuan
    ports:
      - "6379:6379"
    networks:
      extnetwork:
        ipv4_address: 172.19.0.2
  mqtt:
    image: eclipse-mosquitto:latest
    container_name: mqtt-c
    ports:
      - 1883:1883
      - 9001:9001
    networks:
      extnetwork:
        ipv4_address: 172.19.0.4
  zoo:
    image: zookeeper:latest
    container_name: zoo-c
    ports:
      - 2181:2181
    restart: always
    networks:
      extnetwork:
        ipv4_address: 172.19.0.5
networks:
  extnetwork:
    ipam:
      config:
        - subnet: 172.19.0.0/16
          gateway: 172.19.0.1
```

```verilog
Creating network "root_extnetwork" with the default driver
Creating mqtt-c ...
Creating zoo-c ...
Creating redis-c ...
Creating mqtt-c
Creating redis-c
Creating mqtt-c ... done
Attaching to redis-c, zoo-c, mqtt-c
redis-c  | 1:C 04 Jan 06:20:12.837 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-c  | 1:C 04 Jan 06:20:12.838 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis-c  | 1:C 04 Jan 06:20:12.838 # Configuration loaded
zoo-c    | ZooKeeper JMX enabled by default
redis-c  | 1:M 04 Jan 06:20:12.841 * Running mode=standalone, port=6379.
redis-c  | 1:M 04 Jan 06:20:12.842 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis-c  | 1:M 04 Jan 06:20:12.842 # Server initialized
redis-c  | 1:M 04 Jan 06:20:12.842 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix thisissue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to takeeffect.
mqtt-c   | 1578118813: mosquitto version 1.6.8 starting
zoo-c    | Using config: /conf/zoo.cfg
redis-c  | 1:M 04 Jan 06:20:12.842 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, andadd it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
mqtt-c   | 1578118813: Config loaded from /mosquitto/config/mosquitto.conf.
redis-c  | 1:M 04 Jan 06:20:12.842 * Ready to accept connections
mqtt-c   | 1578118813: Opening ipv4 listen socket on port 1883.
mqtt-c   | 1578118813: Opening ipv6 listen socket on port 1883.
zoo-c    | 2020-01-04 06:20:14,275 [myid:] - INFO  [main:QuorumPeerConfig@133] - Reading configuration from: /conf/zoo.cfg
zoo-c    | 2020-01-04 06:20:14,285 [myid:] - INFO  [main:QuorumPeerConfig@375] - clientPort is not set
zoo-c    | 2020-01-04 06:20:14,285 [myid:] - INFO  [main:QuorumPeerConfig@389] - secureClientPort is not set
zoo-c    | 2020-01-04 06:20:14,297 [myid:] - ERROR [main:QuorumPeerConfig@645] - Invalid configuration, only one server specified (ignoring)
zoo-c    | 2020-01-04 06:20:14,312 [myid:1] - INFO  [main:DatadirCleanupManager@78] - autopurge.snapRetainCount set to 3
zoo-c    | 2020-01-04 06:20:14,312 [myid:1] - INFO  [main:DatadirCleanupManager@79] - autopurge.purgeInterval set to 0
zoo-c    | 2020-01-04 06:20:14,312 [myid:1] - INFO  [main:DatadirCleanupManager@101] - Purge task is not scheduled.
```

#### 最后再说几句

1. 这一章我们说明了编排工具**docker-compose**怎么用，相信大家也同样觉得这玩意是真的好用。
2. 相对来说已经是非常的方便了，能够快速的实现批量的容器启动。
3. 下一章节中我们会讲到**docker swarm**集群式的编排，到时候大家会觉得简直就是神器啊！！！
4. 在这里呢非常的感谢各位小伙伴的持续关注，非常高兴能在这里跟大家分享知识。
5. 谢谢大家！！！