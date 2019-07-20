# Spring的事件

Spring的事件（Application Event）为Bean和Bean之间的消息通信提供了支持。当一个Bean处理完一个任务后，希望另外的一个Bean知道并且能作出相应的处理，这个时候我们就需要另外的一个Bean监听当前Bean所发送的事件。

Spring事件需要遵循的流程：

- 自定义事件，继承ApplicationEvent
- 定义事件监听器，实现ApplicationListener
- 使用容器发布事件

Spring的文档对Event的支持翻译之后描述如下：

> ApplicationContext(使用publishEvent发布消息)通过ApplicationEvent类和ApplicationListener接口进行事件处理。 如果将实现ApplicationListener接口的bean注入到上下文中，则每次使用ApplicationContext发布ApplicationEvent时，都会通知该bean。 本质上，这是标准的观察者设计模式。

定义业务需求：

用户注册后，系统需要给用户发送邮件告知用户注册成功，需要给用户初始化积分；隐含的设计需求，用户注册后，后续需求可能会添加其他操作，如再发送一条短信等等，希望程序具有扩展性，以及符合开闭原则

如果不使用事件驱动，代码可能会像这样子：

```java
public class UserService {
   
    @Autowired
    EmailService emailService;
    @Autowired
    ScoreService scoreService;
    @Autowired
    OtherService otherService;
    public void register(String name) {
        System.out.println("用户：" + name + " 已注册！");
        emailService.sendEmail(name);
        scoreService.initScore(name);
        otherService.execute(name);
    } 
}
```

要说有什么毛病，其实也不算有，因为可能大多数人在开发中都会这么写，喜欢写同步代码。但这么写，实际上并不是特别的符合隐含的设计需求，假设增加更多的注册项service，我们需要修改register的方法，并且让UserService注入对应的Service。而实际上，register并不关心这些“额外”的操作，如何将这些多余的代码抽取出去呢？便可以使用Spring提供的Event机制。

### 定义用户注册事件

```java
public class UserRegisterEvent extends ApplicationEvent{
    public UserRegisterEvent(String name) { //name即source
        super(name);
    }
}
```

ApplicationEvent是由Spring提供的所有Event类的基类，为了简单起见，注册事件只传递了name（可以复杂的对象，但注意要了解清楚序列化机制）。

### 定义用户注册服务(事件发布者)

```java
@Service // <1>
public class UserService implements ApplicationEventPublisherAware { // <2>
    public void register(String name) {
        System.out.println("用户：" + name + " 已注册！");
        applicationEventPublisher.publishEvent(new UserRegisterEvent(name));// <3>
    }
    private ApplicationEventPublisher applicationEventPublisher; // <2>
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) { // <2>
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```

<1> 服务必须交给Spring容器托管
<2> ApplicationEventPublisherAware是由Spring提供的用于为Service注入ApplicationEventPublisher事件发布器的接口，使用这个接口，我们自己的Service就拥有了发布事件的能力。
<3> 用户注册后，不再是显示调用其他的业务Service，而是发布一个用户注册事件。

### 定义邮件服务，积分服务，其他服务(事件订阅者)

```java
@Service // <1>
public class EmailService implements ApplicationListener<UserRegisterEvent> { // <2>
    @Override
    public void onApplicationEvent(UserRegisterEvent userRegisterEvent) {
        System.out.println("邮件服务接到通知，给 " + userRegisterEvent.getSource() + " 发送邮件...");// <3>
    }
}
```

<1> 事件订阅者的服务同样需要托管于Spring容器
<2> ApplicationListener<E extends ApplicationEvent>接口是由Spring提供的事件订阅者必须实现的接口，我们一般把该Service关心的事件类型作为泛型传入。
<3> 处理事件，通过event.getSource()即可拿到事件的具体内容，在本例中便是用户的姓名。
其他两个Service，也同样编写，实际的业务操作仅仅是打印一句内容即可，篇幅限制，这里省略。

### 编写启动类

```java
@SpringBootApplication
@RestController
public class EventDemoApp {
    public static void main(String[] args) {
        SpringApplication.run(EventDemoApp.class, args);
    }
    @Autowired
    UserService userService;
    @RequestMapping("/register")
    public String register(){
        userService.register("kirito");
        return "success";
    }
```

当我们调用userService.register(“kirito”);方法时，控制台打印信息如下：

![1538917360565](../../../../AppData/Roaming/Typora/typora-user-images/1538917360565.png)

他们的顺序是无序的，如果需要控制顺序，需要重写order接口，这点不做介绍。其次，我们完成了用户注册和其他服务的解耦，这也是事件驱动的最大特性之一，如果需要在用户注册时完成其他操作，只需要再添加相应的事件订阅者即可。

## Spring 对Event的注解支持

上述的几个接口已经非常清爽了，如果习惯使用注解，Spring也提供了，不再需要显示实现

### 注解式的事件发布者

```java
@Service
public class UserService {
    public void register(String name) {
        System.out.println("用户：" + name + " 已注册！");
        applicationEventPublisher.publishEvent(new UserRegisterEvent(name));
    }
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;
}
```

### 注解式的事件订阅者

```java
@Service
public class EmailService {
    @EventListener
    public void listenUserRegisterEvent(UserRegisterEvent userRegisterEvent) {
        System.out.println("邮件服务接到通知，给 " + userRegisterEvent.getSource() + " 发送邮件...");
    }
}
```

@EventListener注解完成了ApplicationListener<E extends ApplicationEvent>接口的使命。

更多的特性可以参考SpringFramework的文档。

## Spring中事件的应用

在以往阅读Spring源码的经验中，接触了不少使用事件的地方，大概列了以下几个，加深以下印象：

- Spring Security中使用AuthenticationEventPublisher处理用户认证成功，认证失败的消息处理。

```java
public interface AuthenticationEventPublisher {
   void publishAuthenticationSuccess(Authentication authentication);
   void publishAuthenticationFailure(AuthenticationException exception,
         Authentication authentication);
}
```

- Hibernate中持久化对象属性的修改是如何被框架得知的？正是采用了一系列持久化相关的事件，如DefaultSaveEventListener，DefaultUpdateEventListener,事件非常多，有兴趣可以去org.hibernate.event包下查看。
- Spring Cloud Zuul中刷新路由信息使用到的ZuulRefreshListener