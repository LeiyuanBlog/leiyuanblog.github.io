---
layout:     post
title:      Springboot+ssm的application配置
subtitle:   springboot配置
date:       2018-10-08
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - springboot
    - spring
    - ssm
    - 配置
---

> 在这里小小推荐下我的个人博客
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>
> 某宝优惠：[优惠网站](www.innerstudent.group)

# Springboot+ssm的application配置

### 1.我使用的是yml格式的配置文件。
### 2.代码如下

```yaml
#端口号
server:
  port: 8080 
#Spring的配置
spring:
  datasource:
  #数据源
    type: com.alibaba.druid.pool.DruidDataSource
    #驱动
    driver-class-name: com.mysql.jdbc.Driver
    #数据库连接
    url: jdbc:mysql://localhost:3306/saomadiancan?characterEncoding=utf-8&autoReconnect=true&failOverReadOnly=false
    #用户名
    username: root
    #密码
    password: root
    #模板引擎配置
    thymeleaf:
    #配置返回值前缀以及后缀
      prefix: classpath:/templates/
      suffix: .html
      mode: HTML5
 #配置mybatis
mybatis:
#实体类包
  type-aliases-package: com.ambow.springboot.entity
  #配置mapping路径
  mapper-locations: classpath:mapping/*.xml


```