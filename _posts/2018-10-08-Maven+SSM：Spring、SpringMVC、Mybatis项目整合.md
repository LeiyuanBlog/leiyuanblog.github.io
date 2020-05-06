---
layout:     post
title:      Maven+SSM：Spring、SpringMVC、Mybatis项目整合
subtitle:   ssm项目整合
date:       2018-10-08
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Spring
    - Mybatis
    - ssm
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

# Maven+IntelliJ IDEA、SSM：Spring、SpringMVC、Mybatis项目整合

本文将介绍如何搭建ssm项目框架

- **加入依赖**
- **编写配置**
- **链接数据库**
- **编写测试类**
- **项目结构图**

-------------------


## 加入依赖pom.xml

```

    <dependencies>

        <!-- 添加jackson依赖 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.8.8</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.8.8</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.8.8</version>
        </dependency>

        <!-- 添加json依赖 -->
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20180130</version>
        </dependency>

        <!-- 添加commons依赖 -->
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>20030211.134440</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>asm</groupId>
            <artifactId>asm-commons</artifactId>
            <version>3.3</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2</version>
        </dependency>
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.2</version>
        </dependency>

        <!-- 智能对话依赖 -->
        <dependency>
            <groupId>net.sf.ezmorph</groupId>
            <artifactId>ezmorph</artifactId>
            <version>1.0.6</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-util</artifactId>
            <version>9.3.7.v20160115</version>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpcore</artifactId>
            <version>4.4</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-util</artifactId>
            <version>9.3.7.v20160115</version>
        </dependency>

        <!-- 添加阿里巴巴依赖，json以及短信服务接口，智能对话接口，数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.15</version>
        </dependency>
        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>aliyun-java-sdk-core</artifactId>
            <version>3.7.1</version>
        </dependency>
        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>aliyun-java-sdk-dysmsapi</artifactId>
            <version>1.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>

        <!--junit单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>antlr</groupId>
            <artifactId>antlr</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>asm</groupId>
            <artifactId>asm</artifactId>
            <version>3.3</version>
        </dependency>
        <dependency>
            <groupId>asm</groupId>
            <artifactId>asm-tree</artifactId>
            <version>3.3</version>
        </dependency>
        <dependency>
            <groupId>aopalliance</groupId>
            <artifactId>aopalliance</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.22</version>
        </dependency>
        <dependency>
            <groupId>javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.11.0.GA</version>
        </dependency>
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>javax.transaction</groupId>
            <artifactId>jta</artifactId>
            <version>1.1</version>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.25</version>
        </dependency>
        <dependency>
            <groupId>ognl</groupId>
            <artifactId>ognl</artifactId>
            <version>3.0.6</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <!--Spring 4.3.3.release-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
        <!--jstl-->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!--mybatis工具包，反射数据库自动生成实体类等-->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator</artifactId>
            <version>1.3.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
        </dependency>
        <!-- ********************** Excel,jxl工具包********************** -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.14-beta1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml-schemas</artifactId>
            <version>3.14-beta1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.14-beta1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.2</version>
        </dependency>

    </dependencies>

```

##使用阿里巴巴数据源配置数据库jdbc.properties

```
url:jdbc:mysql://172.7.52.231:3306/first_ssm?useUnicode=true&characterEncoding=utf8
driverClassName:com.mysql.jdbc.Driver
username:root
password:123456
#------------------------------------------------------------------------------------------
#配置扩展插件 监控统计用filters:stat 日志用filters:log4j 防御sql注入用filters:wall
filters:stat
#最大连接池数量  初始化建立物理连接的个数  获取连接时最长的等待时间  最小连接池数量  maxIdle已经弃用
maxActive:20
initialSize:1
maxWait:60000
minIdle:10
maxIdle:15
#有两个含义 1.Destroy 线程会检测连接的时间 2.testWhileIdle的判断依据
timeBetweenEvictionRunsMillis:60000
#Destory线程中如果检测到当前连接的最后活跃时间和当前时间的差值大于minEvictableIdleTimeMillis，则关闭当前连接
minEvictableIdleTimeMillis:300000
#用来检测连接是否的sql，要求是一个查询语句。在mysql中通常设置为SELECT 'X'
validationQuery:SELECT 'x'
#申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery连接是否有效
testWhileIdle:true
#申请连接时执行validationQuery检测连接是否有效 这个配置会降低性能
testOnBorrow:false
#归还连接时执行validationQuery检测连接是否有效 这个配置会降低性能
testOnReturn:false
#要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true
maxOpenPreparedStatements:20
#对于建立连接超过removeAbandonedTimeout的连接强制关闭
removeAbandoned:true
#指定连接建立多长就被强制关闭
removeAbandonedTimeout:1800
#指定发生removeabandoned时，是否记录当前线程的堆栈信息到日志中
logAbandoned:true
```

##编写配置文件

### spring-mybatis.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">

    <!-- 配置整合Mybatis -->
    <!-- 1.配置数据库相关参数 -->
    <context:property-placeholder

            location="classpath:jdbc.properties"/>
    <!-- 2.数据库连接池 --><!-- 阿里 druid数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <!-- 数据库基本信息配置 -->
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
        <property name="driverClassName" value="${driverClassName}"/>
        <property name="filters" value="${filters}"/>
        <!-- 最大并发连接数 -->
        <property name="maxActive" value="${maxActive}"/>
        <!-- 初始化连接数量 -->
        <property name="initialSize" value="${initialSize}"/>
        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="${maxWait}"/>
        <!-- 最小空闲连接数 -->
        <property name="minIdle" value="${minIdle}"/>
        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="${timeBetweenEvictionRunsMillis}"/>
        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="${minEvictableIdleTimeMillis}"/>
        <property name="validationQuery" value="${validationQuery}"/>
        <property name="testWhileIdle" value="${testWhileIdle}"/>
        <property name="testOnBorrow" value="${testOnBorrow}"/>
        <property name="testOnReturn" value="${testOnReturn}"/>
        <property name="maxOpenPreparedStatements" value="${maxOpenPreparedStatements}"/>
        <!-- 打开removeAbandoned功能 -->
        <property name="removeAbandoned" value="${removeAbandoned}"/>
        <!-- 1800秒，也就是30分钟 -->
        <property name="removeAbandonedTimeout" value="${removeAbandonedTimeout}"/>
        <!-- 关闭abanded连接时输出错误日志 -->
        <property name="logAbandoned" value="${logAbandoned}"/>
    </bean>
    <!-- 3.配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory"
          class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置mybatis全局配置文件： -->
        <property name="configLocation"
                  value="classpath:Configuration.xml"/>
        <!-- 扫描实体类包 -->
        <property name="typeAliasesPackage" value="com.ambow.first.entity"/>
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>
    <!-- 4.配置扫描Dao接口包,动态实现Dao接口并注入到Spring容器中 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName"
                  value="sqlSessionFactory"/>
        <property name="basePackage" value="com.ambow.first.dao"/>
    </bean>
</beans>
```

### spring-web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    <!-- 配置SpringMvc -->


    <!-- 1.开始SpringMvc注解模式 -->
    <!-- <mvc:annotation-driven /> 会自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter
        两个bean,是spring MVC为@Controllers分发请求所必须的。 -->
    <mvc:annotation-driven/>
    <!-- 2.静态资源默认Servlet配置
        1：加入对静态资源的处理
        2：允许使用"/"做整体映射
    -->
    <mvc:default-servlet-handler/>
    <!-- 3.配置jsp显示viewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/page"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!-- 4.扫描web相关的bean -->
    <context:component-scan base-package="com.ambow.first.controller"/>
    <!-- 5.配置文件上传 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"/>
    </bean>
</beans>
```

### spring-service.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
       <!-- 扫描service包下所有使用注解 的类型 -->
	<context:component-scan base-package="com.ambow.first.service"/>
	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	<!-- 配置基于注解的声明式事务 
		默认使用注解来管理事务
	-->
	<tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

### Configuration.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!--

       Copyright 2009-2016 the original author or authors.

       Licensed under the Apache License, Version 2.0 (the "License");
       you may not use this file except in compliance with the License.
       You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

       Unless required by applicable law or agreed to in writing, software
       distributed under the License is distributed on an "AS IS" BASIS,
       WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
       See the License for the specific language governing permissions and
       limitations under the License.

-->
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
   <settings>
   	<!-- 自动获取数据库自增主键 -->
    <setting name="useGeneratedKeys" value="true"/>
    <!-- 使用列别名代替列名 -->
    <setting name="useColumnLabel" value="true"/>
    <!-- 开启驼峰命名转换 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
</configuration>
```

## 编写junit单元测试类

```
import com.aliyuncs.exceptions.ClientException;
import com.ambow.first.dao.TypeMapper;
import com.ambow.first.util.HttpUtils;
import com.ambow.first.util.Send;
import org.apache.http.HttpResponse;
import org.apache.http.util.EntityUtils;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.HashMap;
import java.util.Map;

// SpringJUnit
@RunWith(SpringJUnit4ClassRunner.class)
// 加入配置
@ContextConfiguration({"/spring-mybatis.xml", "/spring-service.xml"})
public class Test {
    @Autowired
    private TypeMapper typeMapper;

    @org.junit.Test
    public void testType() {
        System.out.println(typeMapper.queryAll().toString());
    }
}
```
### junit运行结果如下

```
log4j:WARN No appenders could be found for logger (org.springframework.test.context.junit4.SpringJUnit4ClassRunner).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[Type{id='0e76f3b8-f576-4b58-9fc3-02ecacff0a5b', name='计算机', place='F1A13', bookNum=3}, Type{id='25b6c373-2d79-4ea2-8fa9-e976e74a05c7', name='小学题库', place='F2 A1', bookNum=0}, Type{id='494f33b3-6edd-4bdc-b4a0-a4c14de6c0e6', name='恐怖', place='F1 A3', bookNum=0}, Type{id='560aecb1-4f25-4738-843d-56c4a1b52957', name='数学', place='F2A13', bookNum=1}, Type{id='573651a8-9fc3-4a76-806b-3e63860af279', name='宗教', place='F1  B12', bookNum=0}, Type{id='67ab626a-c54d-4166-a6b3-6ead775bbca5', name='天文学', place='F1B13', bookNum=1}, Type{id='7de851a4-4b22-46e8-9b74-797d95c4033e', name='机械自动化', place='F2A22', bookNum=2}, Type{id='b0339bdb-d3a5-4a8e-8c8c-e16b5a978aea', name='英语', place='F2A1', bookNum=2}, Type{id='e051ac2e-87c9-4b30-8f31-72e432c67c13', name='自然科学', place='F1A13', bookNum=2}, Type{id='e9b20bbf-02e3-4855-9a2e-0ba01dfca81a', name='菜谱', place='F2A2', bookNum=2}]

```

## 项目结构图如下
![](https://img-blog.csdn.net/2018082811262969?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaXl1YW4yNTgw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



