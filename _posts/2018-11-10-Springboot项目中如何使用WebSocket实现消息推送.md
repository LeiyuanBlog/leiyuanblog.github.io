---
layout:     post
title:      Springboot项目中如何使用WebSocket实现消息推送
subtitle:   Springboot项目中如何使用WebSocket实现消息推送
date:       2018-11-10
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - springboot
    - 消息推送
    - websocket
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

# Springboot项目中如何使用WebSocket实现消息推送

#### 首先，我们来说一下消息推送的应用场景

1.我们现在在饭店吃饭，好多饭店都有扫码点餐自助下单的服务，那么后厨或者是前台是如何收到我们下单的信息，并且能够及时的进行处理呢？

2.我们在网吧，你登录英雄联盟的时候，整个网吧总是会响起“坐在233号的玩家，是来自德玛西亚的钻石大神”。

3.还有等等一系列的推送服务。那么消息推送到底是如何实现的呢？我们今天就来小小的探究一番。

#### 接下来我们进入主题

1.首先我们需要在pom.xml中添加websocket依赖，打开pom：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
```

2.因为我们使用的是springboot项目，不使用配置文件，所以我们需要在项目启动类同级目录创建一个配置类WebSocketConfig.java

```java
package com.ambow.springboot;

import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Component
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}

```

3.接下来就是编写实现类WebSocket.java，通过该类对视图层HTML、JSP进行消息推送，当然功能并不仅仅限制与此。

```java
package com.ambow.springboot;

import groovy.util.logging.Slf4j;
import org.springframework.stereotype.Component;

import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
import java.util.concurrent.CopyOnWriteArraySet;

@Component
// 你的WebSocket访问地址
@ServerEndpoint("/webSocket")
@Slf4j
public class WebSocket {
    private Session session;
    //定义Websocket容器，储存session
    private static CopyOnWriteArraySet<WebSocket> webSocketSet = new CopyOnWriteArraySet<>();

    //对应前端的一些事件
    //建立连接
    @OnOpen
    public void opOpen(Session session) {
        this.session = session;
        webSocketSet.add(this);
    }

    //关闭连接
    @OnClose
    public void onClose() {
        webSocketSet.remove(this);
    }

    //接收消息
    @OnMessage
    public void onMessage(String message) {
    }

    //发送消息
    public void sendMessage(String message) {
        //遍历储存的Websocket
        for (WebSocket webSocket : webSocketSet) {
            //发送
            try {
                webSocket.session.getBasicRemote().sendText(message);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

```

4.接下来我们需要定义在何时出发消息推送方法，一般我们将此类代码放置在service业务逻辑层，例如：在饭店我们下单成功后，逻辑层接收到数据访问层返回成功数据后，调用消息推送方法，将订单信息等等所需数据推送至后厨或者是前台。我在这里以订单为例：OrderServiceImpl.java中创建订单的方法，	webSocket可以像注入Dao层一样注入，因为在上面我们已经进行了bean配置。

```java

	@Autowired
    private WebSocket webSocket;

	@RequestMapping("/createOrder")
    @ResponseBody
    public String createOrder(HttpServletRequest request, HttpServletResponse response) {
        //获取cookie中的订单号
        String order_numberCookie = getorder_numberCookie(request);
        long order_number;
        if (order_numberCookie != null) {
            order_number = Long.parseLong(order_numberCookie);
            orderDao.createOrder(order_number);
            webSocket.sendMessage("有新的订单");
            // 获取名为"cart"的cookie
            Cookie cookie = getCookie(request);
            if (cookie != null) {
                // 设置寿命为0秒
                cookie.setMaxAge(0);
                // 设置路径
                cookie.setPath("/");
                // 设置cookie的value为null
                cookie.setValue(null);
                // 更新cookie
                response.addCookie(cookie);
            }
            return "success";
        }
        return "error";
    }
```

如此我们就将“有心的订单”这条消息发送到了WebSocket.java中，那么在webSocket中就会将我们的消息推送到接收消息的客户端。

#### 那么接下来我们就看看在HTMl、JSP这类视图中是如何接受推送来的消息的。webSocket.js

```javascript
var websocket = null;
//浏览器是否支持
if ('WebSocket' in window) {
    // 上面我们给webSocket定位的路径
    websocket = new WebSocket('ws://localhost:8080/webSocket');
} else {
    alert('该浏览器不支持websocket!');
}
//建立连接
websocket.onopen = function (event) {
    console.log('建立连接');
}
//关闭连接
websocket.onclose = function (event) {
    console.log('连接关闭');
}
//消息来的时候的事件
websocket.onmessage = function (event) {
    // 这里event.data就是我们从后台推送过来的消息
    console.log('收到消息:' + event.data);
    // 在这里我们可以在页面中放置一个音乐，例如“您有新的订单了！”这样的提示音
    document.getElementById("newOrderMp3").play();
}

//发生错误时
websocket.onerror = function () {
    alert('websocket通信发生错误！');
}
//窗口关闭时，Websocket关闭
window.onbeforeunload = function () {
    websocket.close();
}
```

#### 到现在，当有人下单时，你就可以在网页f12的控制台中看到“收到消息：有新的订单啦！”这样的消息。如果你放置了音乐，那么你就可以听到提示了。

# 结束语

1.webSocket的用途很广泛，可以用来做简单的消息推送，可以用来做一个即时的聊天通讯，新闻推送，公告发布等等。

2.非常感谢大家的关注，往后同样，干货不断，大家多多支持关注我！！！感谢🙏🙏🙏！！！