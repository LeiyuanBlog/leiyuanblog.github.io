---
layout:     post
title:      Springboot项目怎么使用拦截器
subtitle:   拦截器的基本实现
date:       2018-10-08
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 拦截器
    - springboot
    - spring
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

# Springboot项目怎么使用拦截器

### 1. 创建一个拦截器并实现HandlerInterceptor接口
```java
package com.leiyuan.bs.interceptor;

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

### 2. 创建一个配置类MyHandlerInterceptorConfig并继承WebMvcConfigurerAdapter类重写addInterceptors(InterceptorRegistry registry)方法
```java
package com.leiyuan.bs;

import com.leiyuan.bs.interceptor.MyHandlerInterceptor;
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

### 3. 启动项目即可看到效果了