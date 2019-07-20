# @Enable*

> @enable*是springboot中用来启用某一个功能特性的一类注解。其中包括我们常用的`@SpringBootApplication`注解中用于开启自动注入的annotation`@EnableAutoConfiguration`，开启异步方法的annotation`@EnableAsync`，开启将配置文件中的属性以bean的方式注入到IOC容器的annotation`@EnableConfigurationProperties`等。

## @EnableAspectJAutoProxy

@EnableAspectJAutoProxy注解 激活Aspect自动代理

```
 <aop:aspectj-autoproxy/>1
```

开启对AspectJ自动代理的支持。

在用到AOP的自动代理的时候用，如果你理解了Java的动态代理，很容易的就会熟悉AOP的自动代理的。

## @EnableAsync

@EnableAsync注解开启异步方法的支持。 
 这个相信大家都比较熟悉的。对于异步应该都理解的。 
 不太熟悉的，可以看这篇博客:-有示例 
 [【Spring】Spring高级话题-多线程-TaskExecutor ](http://blog.csdn.net/qq_26525215/article/details/53214185)

## @EnableScheduling

@EnableScheduling注解开启计划任务的支持。

也就是字面上的意思，开启计划任务的支持！ 
 一般都需要@Scheduled注解的配合。

详情见此博客: 
 [【Spring】Spring高级话题-计划任务-@EnableScheduling  ](http://blog.csdn.net/qq_26525215/article/details/53543816)

## @EnableWebMVC

@EnableWebMVC注解用来开启Web MVC的配置支持。

也就是写Spring MVC时的时候会用到。

## @EnableConfigurationProperties

@EnableConfigurationProperties注解是用来开启对@ConfigurationProperties注解配置Bean的支持。

也就是@EnableConfigurationProperties注解告诉Spring Boot 使能支持@ConfigurationProperties

## @EnableJpaRepositories

@EnableJpaRepositories注解开启对Spring Data JPA Repostory的支持。

Spring Data JPA 框架，主要针对的就是 Spring 唯一没有简化到的业务逻辑代码，至此，开发者连仅剩的实现持久层业务逻辑的工作都省了，唯一要做的，就只是声明持久层的接口，其他都交给 Spring Data JPA 来帮你完成！

简单的说，Spring Data JPA是用来持久化数据的框架。

## @EnableTransactionManagement

@EnableTransactionManagement注解开启注解式事务的支持。

注解@EnableTransactionManagement通知Spring，@Transactional注解的类被事务的切面包围。这样@Transactional就可以使用了。

## @EnableCaching

@EnableCaching注解开启注解式的缓存支持

通过这些简单的@Enable*可以开启一项功能的支持，从而避免自己配置大量的代码，很大程度上降低了使用难度。

我们一起来观察下这些@Enable*注解的源码，可以发现所有的注解都有一个@Import注解。

@Import注解是用来导入配置类的，这也就是说这些自动开启的实现其实是导入了一些自动配置的Bean。

这些导入配置方式主要分为以下三种类型。

  