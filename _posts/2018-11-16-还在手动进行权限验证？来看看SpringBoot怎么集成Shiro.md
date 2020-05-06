---
layout:     post
title:      还在手动处理权限验证、无权异常？来看看SpringBoot怎么集成Shiro
subtitle:   SpringBoot怎么集成Shiro
date:       2018-11-16
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

1.在这里我依旧默认大家熟悉Springboot项目的搭建、maven可以熟练使用。

2.昨天有人喷我说我写的文章很低端，我想说的是：你要是行你也来，别说的好像自己什么都懂一样。

3.非常感谢各位的关注，我依旧日更一些java中常见的技术、知识点。

4.我们步入正题啦！

#### 配置

1.首先我们在maven中添加shiro依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

2.在shiro中有几种不同的认证方式，不过我们基本用不到，除非是非常简单的dome。所以我们一般都是通过继承**AuthorizingRealm**来实现自定义认证。我们这里就省略其他方式，直接带大家来写自定义的认证。首先创建**CustomRealm.java**

```java
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
public class CustomRealm extends AuthorizingRealm {
    @Autowired
    private IUserService userService; // your user service
    @Autowired
    private IRolesService userRolesService; // your roles service
    @Autowired
    private IPermissionsService userPermissionsService; // your permissions service

    /**
     * 获取授权信息
     * @param arg0
     * @return 授权信息
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection arg0) {
        String userName = (String) arg0.getPrimaryPrincipal();
        Set<String> roles = getRolesByEmail(userName);
        Set<String> permissions = getPermissionsByRoles(roles);
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setStringPermissions(permissions);
        authorizationInfo.setRoles(roles);

        return authorizationInfo;
    }

    /**
     * 根据角色集合获取角色对应的权限集合
     * @param roles 角色集合
     * @return 权限集合
     */
    private Set<String> getPermissionsByRoles(Set<String> roles) {
        List<String> list = new ArrayList<String>();
        for (String str : roles) {
            List<String> userPermissionsList = userPermissionsService.getByRoles(str);
            list.addAll(userPermissionsList);
        }
        Set<String> set = new HashSet<>(list);
        return set;
    }

    /**
     * 自己的方法，我这里是根据email获取角色集合
     * @param userName 用户名或者是用户邮箱
     * @return 角色集合
     */
    @SuppressWarnings("unchecked")
    private Set<String> getRolesByEmail(String userName) {
        List<String> list = userRolesService.getRolesByUsername(userName);
        Set<String> set = new HashSet<>(list);
        return set;
    }

    /**
     * 获取认证信息
     * @param arg0
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken arg0) throws AuthenticationException {
        // 从页面获取用户名;
        String username = (String) arg0.getPrincipal();
        // 通过用户名获取凭证；
        String password = getPasswordByUserName(username);
        if (password == null)
            return null;
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(username, password, "customRealm");
        return authenticationInfo;
    }

    /**
     * 自己的方法，根据用户名获取密码
     *
     * @param username 用户名
     * @return 密码
     */
    private String getPasswordByUserName(String username) {
        if (userService.getByUsername(username) == null) {
            return null;
        }
        return userService.getByUsername(username).getPassword();
    }
}
```

那么实现了该类以后呢，你就可以把它保存起来了，因为你以后一定会再次用到它的。

3.接下来就是编写我们的shiro配置类了，在类中你可以配置是否开启注解、拦截的url、不拦截的url、认证成功的跳转页面等等。那么我们创建**ShiroConfig.java**

```java

import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.apache.shiro.mgt.SecurityManager;

import java.util.LinkedHashMap;
import java.util.Map;
// shiro配置类
@Configuration
public class ShiroConfig {
    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        System.out.println("ShiroConfiguration.shirFilter()");
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        //拦截器.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 排除的页面（即为不被拦截的url或静态资源）
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/user/login", "anon");
        //authc表示必须认证通过才可以访问; anon表示可以匿名访问
        filterChainDefinitionMap.put("/**", "authc");
        // 如果拦截到的用户未登陆则自动跳转到以下url
        shiroFilterFactoryBean.setLoginUrl("/user/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * @return
     * @描述：开启Shiro的注解(如@RequiresRoles,@RequiresPermissions)
     * </br>不使用注解的话，可以注释掉这两个配置
     */
    @Bean
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new
                AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
        return authorizationAttributeSourceAdvisor;
    }

    /**
     * 自定义Realm
     * @return Realm
     */
    @Bean
    public CustomRealm myShiroRealm() {
        CustomRealm myShiroRealm = new CustomRealm();
        return myShiroRealm;
    }


    @Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(myShiroRealm());
        return securityManager;
    }
}

```

记住<u>filterChainDefinitionMap.put("/**", "authc");</u>这个一定要放到排除资源的下面，否则很有可能会出现资源即使排除掉也无法访问的问题！！！

4.当你启动项目后会发现，页面会自动跳转到定义的登陆页面。就此我们的配置算是完成了，那我们接下来就是进行我们的代码编写了。

#### 代码编写

1.为了方便测试我们使用返回json数据来进行判断系统是否正常！

2.首先是controller我们来实现UserController.java

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.annotation.RequiresRoles;
import org.apache.shiro.subject.Subject;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author 来自底层程序员的仰望
 * @since 2018-11-16
 */
@RestController
@RequestMapping("/user")
public class UserController {
    // 登陆方法
    @RequestMapping(value = "/login", method = RequestMethod.POST)
    public String login(User user) {
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getName(), user.getPassword());
        try {
            // 认证处理
            subject.login(token);
        } catch (Exception e) {
            return "密码错误";
        }
        return "登陆成功";
    }
    // @RequiresRoles注解测试方法
    @RequiresRoles("use111r")
    @RequestMapping("/testRoles")
    public String testRoles() {
        return "你有权访问啦";
    }
    // 异常处理测试
    @RequestMapping("/log")
    public String log() {
        return "你错啦，你是没有角色访问的！！！！";
    }
}

```

相信上面的代码大家都看得懂！那我们以post方式提交用户名以及密码到login方法中，通过shiro认证后即可看到返回的json数据是“密码错误”还是”登陆成功“！

3.我们再来看看通过注解的方式来拦截没有对应角色或者权限的用户。代码在上方Controller中可以看到，那我们在认证后通过提交get请求访问testRoles方法。如果你有对应的角色，那可以正常的获取到json数据“你有权访问啦”，但如果你恰好没有对应角色，情况是这样子的!

![屏幕快照 2018-11-16 下午5.24.59](https://ws2.sinaimg.cn/large/006tNbRwgy1fxa13iwl30j30pe09ujss.jpg)

看字面意思就明白，说的是没有对应的role：use111r。而且可以看到这是一个500的错误页面，给用户的体验会特别差，那我们该怎么处理呢？接着往下看！

4.这个时候，就需要我们来进行以下全局的异常处理了，我们来编写一下异常处理类**MyException.java**

```java
import org.apache.shiro.authz.AuthorizationException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@ControllerAdvice
public class MyException {
    // 对应处理shiro的AuthorizationException（认证异常）
    @ExceptionHandler(value = AuthorizationException.class)
    public void defaultErrorHandler(HttpServletRequest request, HttpServletResponse response, Exception e) throws Exception {
        System.out.println("无角色异常来了");
        // 重定向到我们的log方法，当然也可以转发到页面，也可以进行自己想要做的任何处理！！！
        response.sendRedirect("/user/log");
    }
}

```

log方法在上面的UserController中，为了方便大家看，我再单独摘出来一下。只是简单的返回错误信息为了方便调试！

```java
    // 异常处理测试
    @RequestMapping("/log")
    public String log() {
        return "你错啦，你是没有角色访问的！！！！";
    }
```

5.那么在进行异常处理后可以看到无角色时是这个样子的，页面不会有报错，因为我们已经对异常进行了处理。用户体验会增强不少！！！![屏幕快照 2018-11-16 下午5.32.28](https://ws1.sinaimg.cn/large/006tNbRwgy1fxa1b9sq1ej30qa0lawg2.jpg)

### 结束语

1.好！今天就说这么多，shiro在我看来是一款非常好用的安全框架，功能比较完善，那么在jsp中也有相对应的标签库供大家使用。非常简单易用，有兴趣的可以搜索一下，也可以私信问我。

2.非常感谢大家的关注和阅读，大家如果有什么问题或者是意见，可以私信或者是评论告诉我。