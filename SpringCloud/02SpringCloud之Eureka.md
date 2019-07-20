# SpringCloud-Eureka

[TOC]

## 1、Eureka是什么


Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了。功能类似于dubbo的注册中心，比如Zookeeper。

## 2、Eureka基本架构

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现(请对比Zookeeper)。

Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

![mark](http://man.hhaxmm.cn/blog/20190619/au2Tp6zwcxGb.png)

Eureka包含两个组件：Eureka Server和Eureka Client

**Eureka Server提供服务注册服务**
各个节点（每个微服务）启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到

**EurekaClient是一个Java客户端**

用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

**三大角色：**

1、Eureka Server 提供服务注册和发现

2、Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到

3、Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务

## 3、Eureka的构建

### 1、Eureka服务注册中心

1、创建SpringBoot的Maven工程，新增POM文件

```xml
<!--eureka-server 服务端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
```

2、配置application.yml

```yml
server:
   port: 7001  #端口号
   
eureka:
   instance:
      hostname: localhost   #eureka服务端的实例名称
   client:
     registerWithEureka: false  #false表示不向注册中心注册自己。
     fetchRegistry: false  #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
     serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka #设置与eureka server 交互的地址查询服务和注册服务都需要依赖的地址
```

3、启动一个服务注册中心，只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加：

```java
@SpringBootApplication
@EnableEurekaServer // EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer7001 {
    public static void main(String[] args){
        SpringApplication.run(EurekaServer7001.class, args);
    }
}
```

4、eureka server 是有界面的，启动工程,打开浏览器访问（[http://localhost:7001](http://localhost:7001/) ,界面如下：

![mark](http://man.hhaxmm.cn/blog/20190620/730ruMJ29mLA.png)

### 2、创建一个服务提供者 (eureka client)

当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

1、创建过程同server类似,创建完pom.xml如下：

```xml
<!--将微服务provider注册进eureka-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
```



2、配置文件中注明自己的服务注册中心的地址，application.yml配置文件如下：

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001
    prefer-ip-address: true     #访问路径可以显示IP地址
```

3、通过注解@EnableEurekaClient 表明自己是一个eurekaclient.

```java
@SpringBootApplication
@EnableEurekaClient
public class DeptProvider8001App {

    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001App.class,args);
    }
}
```

需要指明<font color='red'>spring.application.name)</font>>,这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name 启动工程，打开http://localhost:7001 ，即eureka server 的网址：

![mark](http://man.hhaxmm.cn/blog/20190620/7HIP7jcGqfIS.png)

你会发现一个服务已经注册在服务中了，服务名为**MICORSERVICECLOUD-DEPT** ,端口为7001

## 4、actuator与注册微服务信息完善

### 4.1使用别名的处理微服务的实例，

修改Ppom.xml中的文件

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://localhost:7001/eureka
       #defaultZone: 
  instance:
    instance-id: microservicecloud-dept8001  # 别名的处理
    prefer-ip-address: true     #访问路径可以显示IP地址
```

### 4.2微服务info内容详细信息

> 超链接点击服务报告ErrorPage

1、修改pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2、总的父工程microservicecloud修改pom.xml添加构建build信息

```xml
<build>
   <finalName>microservicecloud</finalName>
   <resources>
     <resource>
       <directory>src/main/resources</directory>
       <filtering>true</filtering>
     </resource>
   </resources>
   <plugins>
     <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-resources-plugin</artifactId>
       <configuration>
         <delimiters>
          <delimit>$</delimit>
         </delimiters>
       </configuration>
     </plugin>
   </plugins>
  </build>
```

