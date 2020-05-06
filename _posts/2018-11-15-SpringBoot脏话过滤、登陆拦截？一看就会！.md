---
layout:     post
title:      SpringBoot脏话过滤、登陆拦截？一看就会！
subtitle:   SpringBoot登陆拦截、脏话过滤？一看就会！
date:       2018-11-15
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

### 前言

首先我们来说一下两者的应用场景。

1.相信大家有大部分人都往过英雄联盟或者是其他的什么游戏，他们都有这不尽相同的脏话过滤系统。当你打游戏输了气的要死的时候，总想骂队友或者是队友几句，但是更可气的事情发生了！别人只能看到被过滤后的**。快来了解一下在Java中脏话过滤是怎么实现的吧。

2.大家一定都上过博客、贴吧等等一系列的平台，当你觉得文章很好，或者很烂的时候，你可能会想要发布一下自己的评论。但是如果你没有登陆，不管你怎么点击、点击哪里，他都只会跳转到登陆或者是注册的页面，这就是拦截器的功劳了，防止为登陆用户或者是无权用户进行胡乱的操作影响系统的正常运行，也来了解一下拦截器的实现吧！

#### 首先我们来说一下脏话过滤

1.首先我们创建我们的过滤类**TestFilter.java**！

```java
import org.springframework.stereotype.Component;
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
@Component
// 过滤路径，过滤器名称
@WebFilter(urlPatterns = "/*", filterName = "testFilter")
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    // 过滤
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        // 获取request
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // 获取response
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        // 创建脏话过滤规则
        DirtyWordsHttpServletRequest dirtyWordsHttpServletRequest = new DirtyWordsHttpServletRequest(request);
        // 执行
        filterChain.doFilter(dirtyWordsHttpServletRequest, response);
    }
    @Override
    public void destroy() {
    }
    // 内部类脏话过滤规则
    class DirtyWordsHttpServletRequest extends HttpServletRequestWrapper {
        // 脏话字典、可以直接搜索脏话字典，然后通过io流进行读取和过滤
        private String[] words = {"傻", "禽", "畜"};
        // 够脏方法
        public DirtyWordsHttpServletRequest(HttpServletRequest request) {
            super(request);
        }
        // 充血getParameter方法
        @Override
        public String getParameter(String name) {
            // 获取传来的参数值
            String value = super.getParameter(name);
            // 判断
            if (value == null) return "没有值";
            // 执行脏话转换
            for (String dword : words) {
                if (value.contains(dword)) value = value.replace(dword, "**");
            }
            // 返回过滤有的值
            return value;
        }
    }
}
```

2.在我们的启动类中加入我们的过滤器！

```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
@ServletComponentScan("com.blogproject.utils.TestFilter")
public class BlogProjectApplication {
    public static void main(String[] args) {
        SpringApplication.run(BlogProjectApplication.class, args);
    }
}

```

3.编写、运行控制层代码查看我们的过滤结果！

```java

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping("/")
    public String test(HttpServletRequest request) {
        System.out.println(request.getParameter("name"));
        return request.getParameter("name");
    }
}

```

4.启动项目，打开浏览器在地址栏中输入**localhost:8080/test/?name=傻112233**就可以看到如图效果以及控制台结果打印，可以很清晰的看到我们输入的脏话被过滤掉了！![屏幕快照 2018-11-15 下午3.23.14](https://ws2.sinaimg.cn/large/006tNbRwly1fx8rypqhx0j30us06at93.jpg)

![屏幕快照 2018-11-15 下午3.23.27](https://ws2.sinaimg.cn/large/006tNbRwly1fx8ryuwsidj31wm056dim.jpg)

#### 接下来，我们就来说一下拦截器的实现

1.创建一个拦截器并实现HandlerInterceptor接口

```java
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
// 拦截器
public class MyHandlerInterceptor implements HandlerInterceptor {
    /**
     * 拦截（Controller方法调用之前）
     *
     * @param request  request
     * @param response response
     * @param o        o
     * @return 通过与否
     * @throws Exception 异常处理
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object
            o) throws Exception {
        // TODO 我这里是通过用户是否登陆进行拦截，我的用户信息存储在session中，名称为userSession，大家可以自行实现
        if (request.getSession().getAttribute("userSession") == null) {
            // 拦截至登陆页面
            request.getRequestDispatcher("/user/toLogin").forward(request, response);
            // false为不通过
            return false;
        }
        // true为通过
        return true;
    }

    // 此方法为处理请求之后调用（调用过controller方法之后，跳转视图之前）
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o,
                           ModelAndView modelAndView) throws Exception {

    }

    // 此方法为整个请求结束之后进行调用
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                                Object o, Exception e) throws Exception {

    }
}
```

2.创建一个配置类MyHandlerInterceptorConfig并继承WebMvcConfigurerAdapter类重写addInterceptors(InterceptorRegistry registry)方法

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
// 拦截器配置类
@Component
public class MyHandlerInterceptorConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    	/**
         * 这里的addPathPatterns("/**")为配置需要拦截的方法“/**”代表所有，而后excludePathPatterns("/user/toLogin")等方法为排除哪些方法不进行		 拦截
         */
        registry.addInterceptor(new MyHandlerInterceptor()).addPathPatterns("/**").excludePathPatterns("/user/toLogin").excludePathPatterns
                ("/user/login").excludePathPatterns("/user/toNewUser").excludePathPatterns("/user/newUser");
        super.addInterceptors(registry);
    }
}

```

启动项目后，就可以看到拦截效果了！！！