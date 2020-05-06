---
layout:     post
title:      MQTT还在使用mosquitto？Vernemq集群搭建、监控是真的好用
subtitle:   MQTT还在使用mosquitto？Vernemq集群搭建、监控是真的好用
date:       2020-05-06
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 消息队列


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

### MQTT还在使用mosquitto？Vernemq集群搭建、监控是真的好用

#### 开头说几句

1. 近期出差了好久，一直没有功夫停下来写一篇文章。
2. 好容易今天有这么个机会，简单的给大家介绍一下**mosquitto**的代替者——**Vernemq**。
3. 专栏也欠更好就了，日后我一定会统统补上，很开心大家能继续支持和关注我。

#### 什么是MQTT？

1. **MQTT**是一种消息队列传输协议。
2. **MQTT**协议是基于TCP的。
3. **MQTT**的优点在于以极少的代码及带宽资源，为远程设备提供可靠的消息服务。
4. **MQTT**是一种低开销的即时通讯协议。

#### MQTT协议中三种身份

1. 发布者（Publish）
2. 代理（Broker）（服务器）
3. 订阅者（Subscribe）

#### MQTT传输的消息

1. 主题（Topic）：可以理解为消息的类型，订阅者订阅（Subscribe）后，就会收到该主题的消息内容（payload）。
2. 负载（payload）：可以理解为消息的内容，是指订阅者具体要使用的内容。

#### 关于消息队列

1. 关于消息队列我就不做过多的介绍了，就简单的说一下我个人的一些理解。
2. 个人在消息队列的使用上都是用来进行事务的异步处理，就好比注册账号。用户需要的只是系统告诉他注册成功可以登录了，但实际上后台服务有很多事情需要处理。比方说分析个人喜好、发送邮件通知你、发送短信通知你等等很多的事情。
3. 但是实际上处理这些需要一定的时间，用户是不想等待的。所以要做的就是验证账号可用之后就返回给用户结果，再通过消息队列的方式将任务下发到其他的模块中进行处理，而不是让用户去等待那些对他来说无关紧要的事务。
4. 还有就是一对多的通知下发，比如只能设备的通知、远程设备的操控等等。
5. 当然这些只是在我个人的应用中所用到的功能，他的能力绝不仅仅于此。

#### 今天的重点重点重点重点——Vernemq

1. 什么是**Vernemq？**

   VerneMQ是一个高性能的分布式MQTT代理。

2. 为什么使用**Vernemq？**

   它在普通硬件上水平和垂直扩展，以支持大量并发发布者和使用者，同时保持低延迟和容错能力。是物联网平台或智能产品的可靠消息中心。

3. 除了这些官方解释，还有什么值得推荐？

   部署构建简单、集群搭建快速稳定、权限配置简单明了、web页面监控状态更加直观。

4. 以上就是我为什么选择**Vernemq**的理由，接下来呢就让我们一起看看具体的安装步骤和集群搭建流程吧。

#### Vernemq的正式应用

1. 安装**Vernemq**

   ```shell
   $ wget https://github.com/vernemq/vernemq/releases/download/1.9.2/vernemq-1.9.2.centos7.x86_64.rpm
   $ yum -y install ./vernemq-1.9.2.centos7.x86_64.rpm
   ```

2. 安装完成后会自动生成配置文件**/etc/vernemq/vernemq.conf**，修改配置文件，仅修改而不是全部替换

   ```properties
   # 是否开启匿名访问，开启匿名访问后将不验证用户名以及密码on\off
   allow_anonymous = off
   # 若配置集群需要放开内部通讯
   allow_register_during_netsplit = on
   allow_publish_during_netsplit = on
   allow_subscribe_during_netsplit = on
   allow_unsubscribe_during_netsplit = on
   # 配置MQTT可连接地址以及端口0.0.0.0及代表任意IP可连接
   listener.tcp.default = 0.0.0.0:1883
   # 服务器内网ip，集群通讯端口。
   # 这里值得注意的是我测试发现如果配置外网的IP集群是无法正常通信的
   listener.vmq.clustering = 10.0.3.4:44053
   # VerneMQweb监控页面以及端口
   listener.http.default = 0.0.0.0:8888
   # 集群中节点名称，避免重复
   nodename = vmqNode1@10.0.3.4
   ```

3. 如果未开启匿名访问，我们就需要为**Vernemq**添加相关的账号密码

   ```shell
   $ vmq-passwd -c /etc/vernemq/vmq.passwd admin
   # 键入密码并验证
   ```

4. 接下来就是配置**topic**的读写权限，默认状态下允许所有用户对所有的**topic**可读写。但安全和规范起见，建议大家规定各任务之间不同的**topic**并对权限加以控制。修改配置文件：**/etc/vernemq/vmq.acl**

   ```properties
   # 添加如下内容
   topic read $SYS/#
   # ACL for user 'admin'
   user admin
   topic test/#
   ```

   这里的#代表统配，例如**test/#**代表**test**及以下所有**topic**。

5. 最后就是启动**vernemq**：`$ systemctl start vernemq`

6. 若为集群则使用如下命令加入任意集群节点：`$ vmq-admin cluster join discovery-node=vmqNode1@10.0.3.4 # 节点名称`

7. 查看节点状态：`vmq-admin cluster show`，或者通过web监控页面查看集群以及节点状态，访问如下地址：`http(s)://ip:8888/status`，这里的端口为上方配置文件中配置的`listener.http.default`信息中的端口。

   ![image-20191028182602815](/Users/leiyuan/Library/Application Support/typora-user-images/image-20191028182602815.png)

#### 还有几句废话

近期的生活并不如意，尽管工作还算顺利。值此之际，我也希望能好好的提升自己，同时也会给大家带来更多新的知识分享。

#### 关于读书

1. 一个陌生女人的来信，这是我最近读的一个作品，虽不赞同她的生活方式却也为她所谓的爱情所折服。在这里也推荐给大家。
2. 希望大家也能推荐给我一些好的书籍，谢谢大家！！！