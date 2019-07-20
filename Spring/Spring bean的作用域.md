# Spring bean的作用域

1、singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域，也可以显示的将Bean定义为singleton模式，配置为：

- `<bean id="userDao" class="com.ioc.UserDaoImpl" scope="singleton"/>`

**说明：Spring中单例Bean与单例模式的区别**

Spring单例Bean与单例模式的区别在于它们关联的环境不一样，单例模式是指在一个JVM进程中仅有一个实例，而Spring单例是指一个Spring Bean容器(ApplicationContext)中仅有一个实例。 

**首先看单例模式**，在一个JVM进程中（理论上，一个运行的JAVA程序就必定有自己一个独立的JVM）仅有一个实例，于是无论在程序中的何处获取实例，始终都返回同一个对象 

**Spring的单例Bean**是与其容器（ApplicationContext）密切相关的，所以在一个JVM进程中，如果有多个Spring容器，即使是单例bean，也一定会创建多个实例，代码示例如下： 

```java
//  第一个Spring Bean容器
ApplicationContext context_1 = new FileSystemXmlApplicationContext("classpath:/ApplicationContext.xml");
Person yiifaa_1 = context_1.getBean("yiifaa", Person.class);
//  第二个Spring Bean容器
ApplicationContext context_2 = new FileSystemXmlApplicationContext("classpath:/ApplicationContext.xml");
Person yiifaa_2 = context_2.getBean("yiifaa", Person.class);
//  这里绝对不会相等，因为创建了多个实例
System.out.println(yiifaa_1 == yiifaa_2);
```

1. prototype:原型模式，每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。根据经验，对有状态的bean使用prototype作用域，而对无状态的bean使用singleton作用域。

2. request：在一次Http请求中，一个bean对应一个实例；即每次Http请求将会有各自的bean实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。

   `<bean id="loginAction" class="com.cnblogs.Login" scope="request"/>`,针对每一次Http请求，Spring容器根据该bean的定义创建一个全新的实例，且该实例仅在当前Http请求内有效，而其它请求无法看到当前请求中状态的变化，当当前Http请求结束，该bean实例也将会被销毁。

3. session：在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。

   `<bean id="userPreference" class="com.ioc.UserPreference" scope="session"/>`,同Http请求相同，每一次session请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的session请求内有效，请求结束，则实例将被销毁。

4. global Session：在一个全局的Http Session中，一个bean对应一个实例，典型的情况下，尽在使用protlet context的时候有效。该作用域仅仅在web的Spring ApplicationContext的情形下有效

