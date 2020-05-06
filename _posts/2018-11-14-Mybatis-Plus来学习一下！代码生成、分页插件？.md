---
layout:     post
title:      Mybatis-Plus来学习一下！代码生成、分页插件？
subtitle:   Mybatis-Plus来学习一下！代码生成、分页插件？
date:       2018-11-14
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - spring
    - springboot
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

#### 今天我们来说一下Mybatis-Plus!

个人认为呢，Mybatis-Plus是Mybatis的增强版，他只是在Mybatis的基础上增加了功能，且并未对原有功能进行任何的改动。可谓是非常良心的一款开源产品，今天我就来给大家简单的说一下以下几个功能，以及代码实现。

#### 开始前的准备

1.我们需要创建一个springboot项目，当然mybatis-plus并不仅限于springboot项目，其他项目也可以正常使用。添加如下依赖：

```xml
        <!-- velocity 模板引擎, 默认 -->
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.0</version>
        </dependency>
        <!-- freemarker 模板引擎 -->
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.28</version>
        </dependency>
		<!-- lombok插件 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.4</version>
        </dependency>
		<!-- mybatis-plus-boot-starter -->
		<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.0.5</version>
        </dependency>
		<!-- druid 阿里巴巴数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>
```

2.我们可以在IDEA中添加如下两个插件方便我们后续开发！lombok以及MybatisX。进入**File -> Settings -> Plugins -> Browse Repositories**，然后分别搜索即可添加。

添加lombok插件以及依赖后，我们的实体类可以这样写：

```java
package com.leiyuan.entity;
import lombok.Data;
import lombok.experimental.Accessors;
/**
 * <p>
 * </p>
 * @author 来自底层程序员的仰望
 * @since 2018-11-14
 */
@Data 
public class User{
    private static final long serialVersionUID = 1L;
    private Integer id;
    private String name;
    private String email;
    private String phone;
    private String password;
    private Integer age;
    private String sex;
    private Integer state;
}

```

在添加**@Data**注解后呢我们可以省略所有封装方法，这样即使我们频繁的改动属性的类型或者名称，都无需重新生成封装方法。为我们节省了大量的时间。是不是很方便呢！

而MybatisX呢是用于Java 与 XML 调回跳转、Mapper 方法自动生成 XML 。以前我们在写mapper的时候呢，都是在mapper.java中定义好方法，然后手动跳转到mapper.xml中去写sql。那么在使用了MybatisX之后呢，我们在编写mapper.java的时候呢，在左侧可以看到Mybatis的小鸟标志。点击就可以进入到对应的方法中，也可以使用control+鼠标左键进行跳转了。并且它会像接口与实现类的关系，自动为你生成代码，非常方便。

![屏幕快照 2018-11-14 下午2.31.00](https://ws3.sinaimg.cn/large/006tNbRwly1fx7kvch41gj312e03wgmi.jpg)

![屏幕快照 2018-11-14 下午2.31.20](https://ws4.sinaimg.cn/large/006tNbRwly1fx7kvq1o32j312e05m758.jpg)

#### 代码生成

编写如下测试类或者main方法,并执行就可以了：

```java
	public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();
        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("来自底层程序员的仰望");
        gc.setOpen(false);
        mpg.setGlobalConfig(gc);
        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://yourHost:3306/test?useUnicode=true&useSSL=false&characterEncoding=utf8");
        dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("yourUsername");
        dsc.setPassword("yourPassword");
        mpg.setDataSource(dsc);
        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("yourPackageName");
        mpg.setPackageInfo(pc);
        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        List<FileOutConfig> focList = new ArrayList<>();
        focList.add(new FileOutConfig("/templates/mapper.xml.ftl") {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输入文件名称
                return projectPath + "/src/main/resources/mapper/" + tableInfo.getEntityName() + "Mapper" +
                        StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);
        mpg.setTemplate(new TemplateConfig().setXml(null));
        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // 是否启用lombok
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 需要自动生成的表名
        strategy.setInclude("user","center");
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
```

源码在官网可以找到，直接复制粘贴，改一下包名、表名就可以了，是不是非常简单呢，项目结构如下图![屏幕快照 2018-11-14 下午2.48.50](https://ws2.sinaimg.cn/large/006tNbRwly1fx7ld5ia63j30gu0pmtb2.jpg)

#### 自动分页

1.首先配置分页插件，编写配置类**PageConfig.java**

```java
package com.config;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;
//Spring boot方式
@EnableTransactionManagement
@Configuration
@MapperScan("com.leiyuan.mapper*")
public class PageConfig {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

2.在mapper.java中编写如下接口

```java
// 导包如下：
// import com.baomidou.mybatisplus.core.metadata.IPage;
// import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
IPage<User> queryAll(Page page);
```

3.在mapper.xml中实现普通的查询语句即可，这里以全查为例

```xml
<select id="queryAll" resultType="com.entity.User">
    select * from user
</select>
```

4.编写test类进行测试，在这里我们就省略service层的代码了。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestMybatisPlusApplicationTests {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void contextLoads() {
        Page<User> userPage = new Page<>();
        // 当前页
        userPage.setCurrent(2);
        // 每页数量
        userPage.setSize(2);
        System.out.println(userMapper.queryAll(userPage).getRecords().toString());
    }
}
```

5.如上便是分页的代码啦，与前台的交互，相信大家没有问题的，这里就不再赘述了。

#### 结束语

非常感谢大家的关注，文章有任何问题，大家都可以私信或者评论告诉我，小弟不才只能浅显的说一些应用的方式方法，底层的东西还要大家自己深入的研究！技术永无止境，各位努力！！！