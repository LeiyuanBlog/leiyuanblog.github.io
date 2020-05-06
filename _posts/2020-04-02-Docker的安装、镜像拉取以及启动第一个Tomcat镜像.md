---
layout:     post
title:      Docker的安装、镜像拉取以及启动第一个Tomcat镜像
subtitle:   Docker的安装、镜像拉取以及启动第一个Tomcat镜像
date:       2020-04-02
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

### Docker的安装、镜像拉取以及启动第一个Tomcat镜像

#### 开头说几句

1. 上一章节中，我们简单的介绍的**Dockerfile、Docker镜像、Docker容器**。
2. 在这一节当中，我们来说一下**Docker**在各个系统版本中的安装以及应用。
3. 后面我们还会提到外网端口映射等等有趣的知识点，希望大家多多支持哦！

#### 正式开始

##### 在这一章节中我们将介绍以下内容

1. **Linux**下**Docker**的安装。
2. **MacOS**下**Docker**的安装。
3. **Windows**下**Docker**的安装。
4. 拉取第一个实用镜像——**Tomcat**镜像。
5. 通过**Docker**启动第一个**Web**项目。
6. 最后说一下日志以及项目挂载。
7. 这里小小的透露一下，下一章我们来说一下**Docker占用空间特别大如何快速解决**？

#### Linus下Docker的安装

##### CentOS、Ubuntu安装

1. 首先切换到**root**用户，不然没权限安装以及卸载软件。

   ```shell
   $ su root
   # 或者
   $ sudo -i
   # 反正有很多种方式切换到root，大家自行抉择
   ```

2. 先别管以前装没装过，直接卸载一波。

   ```shell
   # CentOS
   $ yum -y remove docker docker-engine docker.io
   # Ubuntu
   $ sudo apt-get remove docker docker-engine docker.io
   ```

3. 这里我们就不说复杂的**apt、yum**安装了，针对这两个**Linux**发行版，官方推出了安装脚本。

4. 以下脚本安装方式随是官方推出，但同样不建议大家用于**生产环境**。

5. 下载安装脚本：

   ```shell
   $ curl -fsSL get.docker.com -o get-docker.sh
   ```

6. 执行安装脚本

   ```shell
   $ sh get-docker.sh
   # 国内访问安装源很慢，建议大家使用如下命令使用阿里源进行安装
   $ sh get-docker.sh --mirror Aliyun
   ```

7. 启动**Docker**服务并将其加入到开机启动

   ```shell
   # 以下为两种服务启动方式，根据不同Linux发行版自行选择
   
   # systemctl服务启动并加入开机启动
   $ systemctl start docker
   $ systemctl enable docker
   
   # service服务启动方式
   $ service docker start
   $ chkconfig docker on
   ```

8. 只有 `root` 用户和 `docker` 组的用户才可以访问 Docker。出于安全考虑，**Linux**系统上不会直接使用 `root` 用户。所以我们需要创建**docker**用户组并将需要使用**docker**服务的用户添加到组内。

   ```shell
   # 创建用户组
   $ groupadd docker
   
   # 添加用户到组内
   $ usermod -aG docker User
   ```

9. 需要关闭终端重新登录然后尝试使用User使用调用**Docker**服务。

#### **MacOS**下**Docker**的安装

1. 系统要求：**Docker For Mac** 要求系统最低版本为 **macOS Sierra 10.12**。

2. 如果你的Mac安装了**Homebrew**，那可以直接在终端进行安装。

   ```shell
   $ brew cask install docker
   ```

3. 可惜啊，我没有安装这个玩意，所以我选择了使用客户端进行安装。

4. 需要手动下载安装包，地址这里就不给大家贴出来了，大家自行去**Docker HUB**官方下载一下吧。

5. 下载好之后打开`.dmg`文件，将**Docker**图标拖入到**Applications**中即可完成安装。

6. 后面的操作就都是界面操作了，高端操作就靠大家自己摸索了。

#### **Windows**下**Docker**的安装

1. 系统要求：**Docker For Windows**要求系统版本必须为**Windows10**以及且必须为 **64** 位非家庭版，建议大家使用企业版或者专业版，虽然伟大的网友们出了Win7的安装方式，但是我这里不做推荐，实在太麻烦了，不如换个系统。
2. 系统服务要求：**Windows**中必须开启 **Hyper-V** 虚拟化服务。否则Docker服务正常启动。
3. 同样去**Docker HUB**官方下载**Windows**安装包。
4. 使用方式无他，启动后使用命令提示符（windows终端）使用就可以。

#### 这里额外说一下镜像加速的问题

**Linux**下镜像加速

1. 在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

   ```json
   {
     "registry-mirrors": [
       "https://dockerhub.azk8s.cn",
       "https://reg-mirror.qiniu.com"
     ]
   }
   ```

2. 请注意**json**格式正确，否则服务将无法正常启动

3. 重启服务

   ```shell
   # systemctl重启
   $ systemctl restart docker
   
   # service重启
   $ service docker restart
   ```

**Windows,MacOS**下镜像加速

1. **Docker**服务启动之后，任务栏中会有对应图标。

2. 右键选择 `Settings`，然后选择 `Docker Engine`，同样修改**JSON**文件，修改完后保存并重启服务即可。

   ```json
   {
     "registry-mirrors": [
       "https://dockerhub.azk8s.cn",
       "https://reg-mirror.qiniu.com"
     ]
   }
   ```


#### 拉取第一个实用镜像——**Tomcat**镜像

1. 个人最常用的**web容器**就是**Tomcat**，所以今天我们的第一个镜像就以**Tomcat**为例，简单的说一下镜像的拉取等操作。

2. 这里我们以**Linux**版本为例，其实所有的版本均相同。

3. 首先我们去**docker hub**中查看镜像版本，并选择自己需要的版本。

   ![截屏2019-11-02下午12.55.18](/Users/leiyuan/Desktop/截屏2019-11-02下午12.55.18.png)

4. 通过如上步骤找到自己需要的版本并复制**pull**语句。

5. 在**Docker**中执行**pull**命令`docker pull tomcat:jdk8-adoptopenjdk-openj9`。

#### 通过**Docker**启动第一个**Web**项目并实现项目挂载以及日志挂载

1. 上面呢我们说了拉取**Tomcat**镜像，那接下来我们就说一下如何通过这个镜像运行我们自己的**Web**服务。

2. 首先我们准备一个打包好的**HelloWorld.war**，这是我们写的一个最最简单的**Web应用**，源码我就不贴出来了。

3. 接下来就是启动我们的镜像了，这里需要**注意了！！！**

4. 一般来说我们**tomcat**使用的端口默认是**8080**，如果有特殊需求会去改动，但是镜像中默认只有8080。

5. 而且**Docker**容器与宿主机是相对隔离的，所以容器内的端口我们从外部是无法访问的。

6. 所以我们需要对端口进行映射，也就是使用 `-p 宿主机端口:容器端口` 参数对容器端口进行映射，这样我们才能通过宿主机的**IP地址**对容器内的服务进行访问。

7. 其次我们的**HelloWorld.war**实在镜像外的，镜像内没有，那我们如何将其放入容器内呢，有两种办法。

8. 第一种就是先启动容器，后将**HelloWorld.war**复制到容器当中实现热加载或着对容器进行重启。

   ```shell
   # 1. 启动tomcat容器，这里我们将容器的8080端口映射到宿主机的8080端口
   $ docker run -d -p 8080:8080 --name tomcat tomcat:jdk8-adoptopenjdk-openj9
   # 2. 查看容器ID，第一列即为容器ID
   $ docker ps
   # 3. 复制HelloWorld.war到运行的容器当中
   $ docker cp ./HelloWorld.war 容器ID:/usr/local/tomcat/webapps/
   ```

9. 第二种就是在启动镜像的同时将**HelloWorld.war**挂载到`/usr/local/tomcat/webapps/`目录中。

   ```shell
   $ docker run -d -p 8080:8080 -v ./HelloWorld.war:/usr/local/tomcat/webapps/ --name tomcat tomcat:jdk8-adoptopenjdk-openj9
   ```

10. 同理我们一般会将日志文件挂载到宿主机以便**查看、排障**以及**日志切割**。这里我们就不细说了只需要挂载容器中`/usr/local/tomcat/logs/`目录到宿主机即可。

#### 最后说两句

1. Docker的应用远不止于此，希望大家持续关注，多多支持。
2. 后面我们还有很多要说的，编排啊集群等等。
3. 如果大家有什么问题可以私信我，一定回复大家。
4. 最后感谢各位的持续关注！！！我们下一章见。