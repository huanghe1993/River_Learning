# [第二章]Spring常用的配置

[TOC]

## 一、Bean的scope

Scope描述的是Spring容器如何新建Bean实例的。Spring的Scope有以下的几种形式，通过的是@Scope进行注解实现的

1、singleton作用域（**Spring默认的方式**）

当一个bean的作用域设置为singleton, **那么Spring IOC容器中只会存在一个共享的bean实例**，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。换言之，当把一个bean定义设置为singleton作用域时，Spring IOC容器只会创建该bean定义的唯一实例。这个单一实例会被存储到单例缓存（singleton cache）中，并且所有针对该bean的后续请求和引用都将返回被缓存的对象实例，这里要注意的是singleton作用域和GOF设计模式中的单例是完全不同的，单例设计模式表示一个ClassLoader中只有一个class存在，而这里的singleton则表示一个容器对应一个bean，也就是说当一个bean被标识为singleton时候，spring的IOC容器中只会存在一个该bean。

XML形式的配置

```xml
<bean id="role" class="spring.chapter2.maryGame.Role" scope="singleton"/> 
或者
<bean id="role" class="spring.chapter2.maryGame.Role" singleton="true"/>
```

注解的形式的配置

```java
@Service
@Scope("singleton")
public class DemoSingleService {
}
```

**2、prototype**

prototype作用域部署的bean，每一次请求（将其注入到另一个bean中，或者以程序的方式调用容器的`getBean()`方法）都会产生一个新的bean实例，相当与一个new的操作，<font color ='red'>对于prototype作用域的bean，有一点非常重要，那就是Spring不能对一个prototype bean的整个生命周期负责，容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就对该prototype实例不闻不问了。</font>不管何种作用域，容器都会调用所有对象的初始化生命周期回调方法，而对prototype而言，任何配置好的析构生命周期回调方法都将不会被调用。清除prototype作用域的对象并释放任何prototype bean所持有的昂贵资源，都是客户端代码的职责。（让Spring容器释放被singleton作用域bean占用资源的一种可行方式是，通过使用bean的后置处理器，该处理器持有要被清除的bean的引用。）

**3、request** 

request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效，配置实例：

request、session、global session使用的时候首先要在初始化web的web.xml中做如下配置：

如果你使用的是Servlet 2.4及以上的web容器，那么你仅需要在web应用的XML声明文件web.xml中增加下述ContextListener即可：

```xml
<web-app>   ... 
    <listener>
        <listener-class>       org.springframework.web.context.request.RequestContextListener
        </listener-class>  
    </listener>
   ...
</web-app>

 
```

```xml
接着既可以配置bean的作用域了：
<bean id="role" class="spring.chapter2.maryGame.Role" scope="request"/>
```

**4、session**

session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效，配置实例：

配置实例：

和request配置实例的前提一样，配置好web启动文件就可以如下配置：

```xml
<bean id="role" class="spring.chapter2.maryGame.Role" scope="session"/>
```

**5、global session**

global session作用域类似于标准的HTTP Session作用域，<font color='red'>不过它仅仅在基于portlet的web应用中才有意义</font>。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。如果你在web中使用global session作用域来标识bean，那么web会自动当成session类型来使用。

配置实例：

和request配置实例的前提一样，配置好web启动文件就可以如下配置：

```xml
<bean id="role" class="spring.chapter2.maryGame.Role" scope="global session"/>
```





