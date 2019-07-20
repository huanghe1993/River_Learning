# WebSocket

[TOC]

　　随着互联网的发展，传统的HTTP协议已经很难满足Web应用日益复杂的需求了。近年来，随着HTML5的诞生，WebSocket协议被提出，它实现了浏览器与服务器的全双工通信，扩展了浏览器与服务端的通信功能，使服务端也能主动向客户端发送数据。

　　我们知道，传统的HTTP协议是无状态的，每次请求（request）都要由客户端（如 浏览器）主动发起，服务端进行处理后返回response结果，而服务端很难主动向客户端发送数据；这种客户端是主动方，服务端是被动方的传统Web模式 对于信息变化不频繁的Web应用来说造成的麻烦较小，而对于涉及实时信息的Web应用却带来了很大的不便，如带有即时通信、实时数据、订阅推送等功能的应 用。在WebSocket规范提出之前，开发人员若要实现这些实时性较强的功能，经常会使用折衷的解决方法：**轮询（polling）**和**Comet**技术。其实后者本质上也是一种轮询，只不过有所改进。

　　Comet技术**又可以分为**长轮询**和**流技术**。**长轮询**改进了上述的轮询技术，减小了无用的请求。它会为某些数据设定过期时间，当数据过期后才会向服务端发送请求；这种机制适合数据的改动不是特别频繁的情况。**流技术**通常是指客户端使用一个隐藏的窗口与服务端建立一个HTTP长连接，服务端会不断更新连接状态以保持HTTP长连接存活；这样的话，服务端就可以通过这条长连接主动将数据发送给客户端；流技术在大并发环境下，可能会考验到服务端的性能

　　这两种技术都是基于请求-应答模式，都不算是真正意义上的实时技术；它们的每一次请求、应答，都浪费了一定流量在相同的头部信息上，并且开发复杂度也较大。

　　伴随着HTML5推出的WebSocket，真正实现了Web的实时通信，使B/S模式具备了C/S模式的实时通信能力。WebSocket的工作流程是这 样的：浏览器通过JavaScript向服务端发出建立WebSocket连接的请求，在WebSocket连接建立成功后，客户端和服务端就可以通过 TCP连接传输数据。因为WebSocket连接本质上是TCP连接，不需要每次传输都带上重复的头部数据，所以它的数据传输量比轮询和Comet技术小 了很多。本文不详细地介绍WebSocket规范，主要介绍下WebSocket在Java Web中的实现。



## Websocket是什么样的协议，具体有什么优点

首先，Websocket是一个持久化的协议，相对于HTTP这种非持久的协议来说

首先，Websocket是一个持久化的协议，相对于HTTP这种非持久的协议来说。简单的举个例子吧，用目前应用比较广泛的PHP生命周期来解释。

HTTP的生命周期通过 `Request` 来界定，也就是一个 `Request` 一个 `Response` ，那么在 `HTTP1.0` 中，这次HTTP请求就结束了。

在HTTP1.1中进行了改进，使得有一个keep-alive，也就是说，在一个HTTP连接中，可以发送多个Request，接收多个Response。但是请记住 `Request = Response`， 在HTTP中永远是这样，也就是说一个request只能有一个response。而且这个response也是被动的，不能主动发起

首先Websocket是基于HTTP协议的，或者说借用了HTTP的协议来完成一部分握手。

首先我们来看个典型的 `Websocket` 握手（借用Wikipedia的。。）

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

熟悉HTTP的童鞋可能发现了，这段类似HTTP协议的握手请求中，多了几个东西。我会顺便讲解下作用。

```
Upgrade: websocket
Connection: Upgrade
```

这个就是Websocket的核心了，告诉 `Apache` 、 `Nginx` 等服务器：注意啦，我发起的是Websocket协议，快点帮我找到对应的助理处理~不是那个老土的HTTP。

```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

首先， `Sec-WebSocket-Key` 是一个 `Base64 encode` 的值，这个是浏览器随机生成的，告诉服务器：泥煤，不要忽悠窝，我要验证尼是不是真的是Websocket助理。

然后， `Sec_WebSocket-Protocol` 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。简单理解：今晚我要服务A，别搞错啦~

最后， `Sec-WebSocket-Version` 是告诉服务器所使用的 `Websocket Draft` （协议版本），在最初的时候，Websocket协议还在 `Draft` 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么Firefox和Chrome用的不是一个版本之类的，当初Websocket协议太多可是一个大难题。。不过现在还好，已经定下来啦~大家都使用的一个东西~ 脱水： **服务员，我要的是13岁的噢→_→**

然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功切换协议啦~

```
Upgrade: websocket
Connection: Upgrade
```

依然是固定的，告诉客户端即将升级的是 `Websocket` 协议，而不是mozillasocket，lurnarsocket或者shitsocket。

然后， `Sec-WebSocket-Accept` 这个则是经过服务器确认，并且加密过后的 `Sec-WebSocket-Key` 。 服务器：好啦好啦，知道啦，给你看我的ID CARD来证明行了吧。。

后面的， `Sec-WebSocket-Protocol` 则是表示最终使用的协议。

至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。具体的协议就不在这阐述了。

## WebSocket的广播

单播（Unicast）:点对点，私信私聊

广播（Broadcast）:游戏公告

多播，组播（Multicast）:多人聊天室，发布订阅

## 广播技术的应用

### 简单的webSocket游戏公告系统

webjar的介绍

js的框架管理的方式进行管理，web前端的工具打包成为一个jar包，方便统一的管理，主要是解决版本不一致的问题，文件混乱的问题，把前端的资源，打包成相应的jar进行开发配置

配置WebSocket，需要在配置类上使用@EnableWebSocketMessageBroker开启WebSocket的配置，并且继承AbstractWebSocketMessageBrokerConfigurer类重写其中的方法来配置webSocket

![1539912184036](../../../../AppData/Roaming/Typora/typora-user-images/1539912184036.png)

- 首先是用户需要订阅，有的是订阅的公告，有的是订阅的排行版
- 

```java
package com.huanghe.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

/**
 * @Author: River
 * @Date:Created in  11:11 2018/10/18
 * @Description:
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {



    /**
     * //注册一个端点,发布或者是订阅消息的时候需要小链接上这个端点
     * setAllowedOrigins :不是必须的，* 表示的是允许其他域进行连接
     * withSockJS ：表示开始socketJs的支持
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {①
        registry.addEndpoint("/endpoint-websocket").setAllowedOrigins("*").withSockJS();
    }

    /**
     * 配置消息代理（中介转发）
     * enableSimpleBroker：topic是服务端推送数据给客户端的路径前缀
     * setApplicationDestinationPrefixes：是客户端发送数据给服务端的一个前缀
     * @param registry
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/chat");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```

获取连接的代码：

```js
function connect() {
    var socket = new SockJS('/endpoint-websocket'); //连接上端点(基站) ①
    
    stompClient = Stomp.over(socket);			//用stom进行包装，规范协议
    stompClient.connect({}, function (frame) {	
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/game_chat', function (result) {
        	console.info(result)
        	showContent(JSON.parse(result.body));
        });
    });
}

function sendName() {
	
    stompClient.send("/app/v1/chat", {}, JSON.stringify({'content': $("#content").val()}));
}
```

服务端的代码controller

```java
/**
 * @Author: River
 * @Date:Created in  11:04 2018/10/18
 * @Description: 游戏公告Controller
 */
@Controller
public class GameInfoController {

    /**
     * @MessageMapping :当浏览器向服务器发送消息的时，通过@MessageMapping映射的地址，接收到数据
     *  @SendTo :当服务端有消息的时候，会对订阅了@SentTo中的路径的浏览器发送消息
     * @param inMessage ：管理员传输过来的信息
     * @return：直接的把管理员传输过来的信息传输出去
     */
    @MessageMapping("/v1/chat")
    @SendTo("/topic/game_chat")
    public OutMessage gameInfo(InMessage inMessage) {
        return new OutMessage(inMessage.getContent());
    }
}

```





①：代码中的index.html(用户端)和admin.html(管理员)都需要通过`var socket = new SockJS('/endpoint-websocket');` 连接上端点(基站),此时的双方才是可以进行通信的

②：现在的客户端是有index(普通的用户)和admin（管理员），`registry.enableSimpleBroker("/topic", "/chat");`是服务端推送数据给客户端的路径前缀，客户端需要订阅`stompClient.subscribe('/topic/game_chat'`才是可以收到消息的，这个路径是在服务端的@SenTo的注解里面写的，这个路径的前缀就是在`registry.enableSimpleBroker("/topic", "/chat");`中定义的前缀

③： `registry.setApplicationDestinationPrefixes("/app");`是客户端发送数据给服务端的前缀，现在是需要admin给服务端发送数据发送的路径就是前缀+`@MessageMapping("/v1/chat")`组合成最终的路径

④：Controller中的代码主要的作用就是把admin中的数据发送到客户端中去

## WebSocket整合SpringBoot监听器

### webSocket推送方法讲解

@SendTo注解和SimpleMessagingTemplate的区别：

1、@SentTo是不同用的，固定发给指定的订阅者

2、SimpleMessagingTemplate灵活，支持的  多种方式

```java
/**
 * @Author: River
 * @Date:Created in  10:10 2018/10/19
 * @Description： 简单消息模板，用来推送消息
 */
@Service
public class WebSocketService {


    @Autowired
    private SimpMessagingTemplate template;


    public void sendTopicMessage(InMessage message) {
        for (int i = 0; i < 20; i++) {
            template.convertAndSend("/topic/game_rank",new OutMessage(message.getContent()));
        }
    }


    //客户端需要进行订阅的
    public void sendChatMessage(InMessage message) {
        template.convertAndSend("/chat/single/"+message.getTo(),new OutMessage(message.getFrom()+"发送"+message.getContent()));
    }

    /**
     * 获取JVM的信息，推送给客户端
     */
    public void sendServerInfo() {
        //获取JVM的可用核数
        int processor = Runtime.getRuntime().availableProcessors();
        long freeMemory = Runtime.getRuntime().freeMemory();
        long maxMemory = Runtime.getRuntime().maxMemory();
        String message = String.format("服务器可用的处理器：%s;虚拟机的空闲的内存大小：%s;最大的内存的大小%s", processor, freeMemory, maxMemory);
        template.convertAndSend("/topic/server_info", new OutMessage(message));
    }

    /**
     * 获取股票的数据
     */
    public void sendStockInfo() {
        Map<String, String> map = StockService.getStockInfo();
        String message = String.format("股票的最低的价格是：%s", map.get("min_price"));
        template.convertAndSend("/topic/stack_info",new OutMessage(message));
    }
}

```



SessionSubscribeEvent时间监听器，监听订阅事件

```java
/**
 * @Author: River
 * @Date:Created in  10:44 2018/10/19
 * @Description: 订阅事件的监听器
 * SessionSubscribeEvent ：是此监听器监听的事件
 */
@Component
public class ConnnectEventListener implements ApplicationListener<SessionConnectEvent> {

    //当事件触发的时候我们需要做的事情
    //StompHeaderAccessor:简单消息传递协议中处理消息头的基类，可以得到消息的类型（例如：发布订阅，建立连接断开连接，会话id等）
    @Override
    public void onApplicationEvent(SessionConnectEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        System.out.println("[ConnnectEventListener监听器事件 类型]"+headerAccessor.getCommand().getMessageType());
    }
}

```

监听WebSocket的建立连接的

```java
**
 * @Author: River
 * @Date:Created in  10:44 2018/10/19
 * @Description: 订阅事件的监听器
 * SessionSubscribeEvent ：是此监听器监听的事件
 */
@Component
public class SubscribeEventListener implements ApplicationListener<SessionSubscribeEvent> {

    //当事件触发的时候我们需要做的事情
    @Override
    public void onApplicationEvent(SessionSubscribeEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        System.out.println("[监听器事件 类型]"+headerAccessor.getCommand().getMessageType());
    }
}
```



## WebSocket和Spring的拦截器的使用

HandleshakeInterceptor（握手拦截器）的介绍







