# SpringBoot基础

SpringBoot可以以jar包的形式独立的运行，运行一个Spring Boot项目只需要通过java -jar xx.jar的方式来运行，SpringBoot可以选择内嵌的Tomcat，Jetty或者是Undertow，这样我们无须以war包的形式部署项目，SpringBoot提供了一系列的starter pom来简化Maven的依赖加载，例如当你使用了Spring-boot-starter-web时，会自动的加入一系列的包进去，SpringBoot会根据在类路径中的jar包、类，为jar包里的类自动的配置Bean,这样会极大的减少我们需要使用的配置

## SpringBoot的基本配置

SpringBoot通常是有一个名为Application的入口类，入口类里面有一个main方法，这个main方法其实就是一个标准的Java应用的入口方法，在main方法中使用SpringApplication.run就可以启动SpringBoot项目

@SpringBootApplication是Spring Boot的核心注解，它是一个组合的注解，源码如下所示：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "exclude"
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "excludeName"
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}

```

@SpringBootApplication是一个组合的注解：

- @Configuration:表示它是一个配置文件的类
- @EnableAutoConfiguration让SpringBoot根据类路径中的jar包依赖为当前的项目进行自动的配置
- @ComponentScan:是扫描指定表下的Component（@Componment，@Configuration，@Service) 等等,如果是没有其他的配置的情况下，默认是会扫描与配置类相同的包，

之前我们对大部分的配置文件的作用都进行了讨论，但是还有一个注解我们并不是特别的熟悉：

> 下面我们重点的讨论一下@EnaleAutoConfiguration是如何的让类路径中的jar包依赖为当前的项目进行自动的配置的

首先我们看一下@EnableAutoConfiguration的源码

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

可能有些人对@Import的用法不是特别的熟悉：下面就简单的补充一个@Import的用法

> 在应用中，有时没有把某个类注入到IOC容器中，但在运用的时候需要获取该类对应的bean，此时就需要用到@Import注解。
>
> 先创建两个类，不用注解注入到IOC容器中，在应用的时候在导入到当前容器中。 
> 1、创建Dog和Cat类 
> Dog类：
>
> ```java
> package com.example.demo;
> 
> public class Dog {
> 
> }
> ```
>
> Cat类：
>
> ```java
> package com.example.demo;
> 
> public class Cat {
> 
> }
> ```
>
> 2、在启动类中需要获取Dog和Cat对应的bean，需要用注解@Import注解把Dog和Cat的bean注入到当前容器中。
>
> ```java
> //@SpringBootApplication
> @ComponentScan
> /*把用到的资源导入到当前容器中*/
> @Import({Dog.class, Cat.class})
> public class App {
> 
>     public static void main(String[] args) throws Exception {
> 
>         ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
>         System.out.println(context.getBean(Dog.class));
>         System.out.println(context.getBean(Cat.class));
>         context.close();
>     }
> }
> ```
> **另外，也可以导入一个配置类** 
> 还是上面的Dog和Cat类，现在在一个配置类中进行配置bean，然后在需要的时候，只需要导入> >   这个配置就可以了，最后输出结果相同。
>
> MyConfig 配置类:
>
> ```java
> package com.example.demo;
> import org.springframework.context.annotation.Bean;
> 
> public class MyConfig {
> 
>     @Bean
>     public Dog getDog(){
>         return new Dog();
>     }
> 
>     @Bean
>     public Cat getCat(){
>         return new Cat();
>     }
> 
> ```
>
> 比如若在启动类中要获取Dog和Cat的bean，如下使用：
>
> ```java
> //@SpringBootApplication
> @ComponentScan
> /*导入配置类就可以了*/
> @Import(MyConfig.class)
> public class App {
> 
>     public static void main(String[] args) throws Exception {
> 
>         ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
>         System.out.println(context.getBean(Dog.class));
>         System.out.println(context.getBean(Cat.class));
>         context.close();
>     }
> }
> 
> ```
>
>

撇开上面的关于@Import的用法，我们可以看到@EnableAutoConfig的关键的功能是在@Import注解导入的配置功能，EnableAutoConfigurationImportSelector.class使用的是SpringFactoriesLoader.loadFactoryNames方法来扫描META-INF/Spring.factory文件包，而我们的Spring-boot-autoconfiguration-1.5.16.jar里面就有一个Spring.factories的文件，此文件中申明的配置如下所示

![1539487360491](../../../../AppData/Roaming/Typora/typora-user-images/1539487360491.png)

打开上面的任意的一个AutoConfiguration文件，一般都有下面的条件注解，在spring-boot-autoconfiguration-1.5.16.jar的org.springframework.boot.autoconfigure.condition包下，条件注解如下所示：

| 注解                          | 作用                                   |
| ----------------------------- | -------------------------------------- |
| @ConditionOnBean              | 当容器中有指定的bean的条件下           |
| @ConditionOnClass             | 当类路径下有指定的类的条件下           |
| @ConditionOnJava              | 基于JVM版本作为指定的条件              |
| @ConditionOnJndi              | 在JNDI存在的条件下查找指定的位置       |
| @ConditionOnMissingBean       | 在容器里没有指定Bean的条件下           |
| @ConditionOnMissingClass      | 在类路径中没有指定类的条件下           |
| @ConditionOnNotWebApplication | 当前的项目不是web项目的条件下          |
| @ConditionOnProperty          | 指定的属性中是否有指定的值             |
| @ConditionOnResource          | 类路径中是否存在指定的值               |
| @ConditionOnSingleCandidate   | 当指定的Bean在容器中只存在一个人的时候 |
| @ConditionOnWebApplication    | 只有时在web项目的前提下                |
|                               |                                        |
|                               |                                        |

这些注解都是组合了@Condition元注解，只是使用了不同的条件（condition），我们在之前已经知道了根据不同的条件创建不同的bean的实例，下面我们就简单的分析一下@ConditionOnWebApplication注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({OnWebApplicationCondition.class})
public @interface ConditionalOnNotWebApplication {
}
```

可以看出来这个注解使用的条件是OnWebApplicationCondition.class，下面我们来看一下这个条件是如何的构造：

```java
@Order(-2147483628)
class OnWebApplicationCondition extends SpringBootCondition {
    private static final String WEB_CONTEXT_CLASS = "org.springframework.web.context.support.GenericWebApplicationContext";

    OnWebApplicationCondition() {
    }

    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        boolean required = metadata.isAnnotated(ConditionalOnWebApplication.class.getName());
        ConditionOutcome outcome = this.isWebApplication(context, metadata, required);
        return required && !outcome.isMatch()?ConditionOutcome.noMatch(outcome.getConditionMessage()):(!required && outcome.isMatch()?ConditionOutcome.noMatch(outcome.getConditionMessage()):ConditionOutcome.match(outcome.getConditionMessage()));
    }

    private ConditionOutcome isWebApplication(ConditionContext context, AnnotatedTypeMetadata metadata, boolean required) {
        Builder message = ConditionMessage.forCondition(ConditionalOnWebApplication.class, new Object[]{required?"(required)":""});
        if(!ClassUtils.isPresent("org.springframework.web.context.support.GenericWebApplicationContext", context.getClassLoader())) {
            return ConditionOutcome.noMatch(message.didNotFind("web application classes").atAll());
        } else {
            if(context.getBeanFactory() != null) {
                String[] scopes = context.getBeanFactory().getRegisteredScopeNames();
                if(ObjectUtils.containsElement(scopes, "session")) {
                    return ConditionOutcome.match(message.foundExactly("\'session\' scope"));
                }
            }

            return context.getEnvironment() instanceof StandardServletEnvironment?ConditionOutcome.match(message.foundExactly("StandardServletEnvironment")):(context.getResourceLoader() instanceof WebApplicationContext?ConditionOutcome.match(message.foundExactly("WebApplicationContext")):ConditionOutcome.noMatch(message.because("not a web application")));
        }
    }
}

```

- GenericWebApplicationContext是否在类路径中；

- 容器中是否有名为session的scope；

- 当前容器的Enviroment是否为StandardServletEnvironment；

- 当前的ResourceLoader是否是WebApplicationContext（ResourceLoader是ApplicationContext的顶级接口之一）；

- 我们需要构建ConditionOutcome类的对象来帮助我们，最终通过ConditionOutcome.isMatch方法返回值来确定条件。

## 实例分析

我们在常规的配置http编码的时候是在web.xml里配置一个filter，如：

```xml
<filter>
		<filter-name>SetCharacterEncoding</filter-name>
    
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter
		</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>

```

forceEncoding用来设置是否理会 request.getCharacterEncoding()方法，设置为true则强制覆盖之前的编码格式。如果是false，当请求被提交之后，过滤器会判断request.getCharacterEncoding()是否为null，如果为null则会进行request.setCharacterEncoding("UTF-8")的操作，这个时候设置的编码就会生效，如果不为null,则在上面的设置中就无效，过滤器什么都不用做

在SpringBoot中则是这样配置的：

**1：配置参数**

SpringBoot是基于类型安全的配置来实现的，这里的配置类可以在application.properties中直接的设置，源码如下所示：

```java
@ConfigurationProperties(prefix = "spring.http.encoding")
public class HttpEncodingProperties {

	public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

	/**
	 * Charset of HTTP requests and responses. Added to the "Content-Type" header if not
	 * set explicitly.
	 */
	private Charset charset = DEFAULT_CHARSET;

	/**
	 * Force the encoding to the configured charset on HTTP requests and responses.
	 */
	private Boolean force;

	/**
	 * Force the encoding to the configured charset on HTTP requests. Defaults to true
	 * when "force" has not been specified.
	 */
	private Boolean forceRequest;

	/**
	 * Force the encoding to the configured charset on HTTP responses.
	 */
	private Boolean forceResponse;

	/**
	 * Locale to Encoding mapping.
	 */
	private Map<Locale, Charset> mapping;

	public Charset getCharset() {
		return this.charset;
	}

	public void setCharset(Charset charset) {
		this.charset = charset;
	}

	public boolean isForce() {
		return Boolean.TRUE.equals(this.force);
	}

	public void setForce(boolean force) {
		this.force = force;
	}

	public boolean isForceRequest() {
		return Boolean.TRUE.equals(this.forceRequest);
	}

	public void setForceRequest(boolean forceRequest) {
		this.forceRequest = forceRequest;
	}

	public boolean isForceResponse() {
		return Boolean.TRUE.equals(this.forceResponse);
	}

	public void setForceResponse(boolean forceResponse) {
		this.forceResponse = forceResponse;
	}

	public Map<Locale, Charset> getMapping() {
		return this.mapping;
	}

	public void setMapping(Map<Locale, Charset> mapping) {
		this.mapping = mapping;
	}

	boolean shouldForce(Type type) {
		Boolean force = (type != Type.REQUEST) ? this.forceResponse : this.forceRequest;
		if (force == null) {
			force = this.force;
		}
		if (force == null) {
			force = (type == Type.REQUEST);
		}
		return force;
	}

	enum Type {

		REQUEST, RESPONSE

	}

}
```

代码解释：

- 在application.properties配置的时候前缀是spring.http.encoding;
- 默认的编码的方式是UTF-8,如果修改可以使用spring.http.encoding.charset=编码
- 设置forceEncoding，默认是为true的，可以修改为false

**2：配置Bean**

通过上述的配置，并根据条件配置CharacterEncodingFilter的Bean，看源码

```java
@Configuration
@EnableConfigurationProperties(HttpEncodingProperties.class)//1
@ConditionalOnWebApplication
@ConditionalOnClass(CharacterEncodingFilter.class)//2
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)//3
public class HttpEncodingAutoConfiguration {

	private final HttpEncodingProperties properties;

	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}

	@Bean//4
	@ConditionalOnMissingBean(CharacterEncodingFilter.class)//5
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}

	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}

	private static class LocaleCharsetMappingsCustomizer
			implements EmbeddedServletContainerCustomizer, Ordered {

		private final HttpEncodingProperties properties;

		LocaleCharsetMappingsCustomizer(HttpEncodingProperties properties) {
			this.properties = properties;
		}

		@Override
		public void customize(ConfigurableEmbeddedServletContainer container) {
			if (this.properties.getMapping() != null) {
				container.setLocaleCharsetMappings(this.properties.getMapping());
			}
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}

}

```



- (1)、开启属性注入，@ConfigurationProperties注解主要用来把properties配置文件转化为bean来使用的，而@EnableConfigurationProperties注解的作用是@ConfigurationProperties注解生效。如果只配置@ConfigurationProperties注解，在IOC容器中是获取不到properties配置文件转化的bean的。[@ConfigurationProperties和@EnableConfigurationProperties配合使用](https://blog.csdn.net/u010502101/article/details/78758330)

  @EnableConfigurationProperties声明，使用@Auoreired注入

- @ConditionOnWebApplication在web的条件下的会后才会生效

- (2)、当CharacterEncodingFilter在类路径下会生效

- (3)、@ConditionOnProperty 可以通过配置文件中的属性值来判定configuration是否被注入,

  - 当value所对应配置文件中的值为false时，注入不生效，不为fasle注入生效；value有多个值时，只要有一个值对应为false,则注入不成功

  - matchIfMissing()该属性为true时，配置文件中缺少对应的value或name的对应的属性值，也会注入成功
  - prefix匹配前缀

  当配置spring.http.encoding=enable的条件下是可以的，如果没有配置默认的情况下是为true,即条件是符合的

- (4)、使用Java配置的方式配置CharacterEncodingFilter这个Bean

- (5)、当容器中没有这个Bean的时候新建Bean

## 实战

直到这里我们基本上Spring Boot是如何的完成自动的配置的功能的，其实我们完全的可以仿照上面http编码的配置的例子我们自己写一个配置文件，不过这里我们可以再做的更加的彻底一点，我们自己写一个start pom,这意味着我们不仅有自动配置的功能，而且具有跟加通用的耦合度更低的配置

　　为了更加的方便的理解，我们这里举一个非常简单的案例，包含某个类存在的时候，自动的配置这个类的Bean,并且可将bean的属性在applicaiotn.properties的文件中设置

pom.xml文件如下所示

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.huanghe</groupId>
  <artifactId>spring-boot-starter-hello</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>spring-boot-starter-hello</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
      <version>1.5.3.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.7.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.20.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

在此处增加SpringBoot自身的自动配置作为依赖

（2）属性配置，代码如下所示：

```java
package com.huanghe.spring_boot_starter_hello;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * @Author: River
 * @Date:Created in  21:22 2018/10/15
 * @Description:
 */
@ConfigurationProperties(prefix = "hello")
public class HelloServiceProperties {

    private static final String MSG = "world";

    private String msg = MSG;

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```

这里和上面的是一样的，属于类型安全的自动属性获取，在application.properties中通过hello.mag=可以用来设置属性的值，若是不设置默认的属性的值为hello.msg=world

(3):判断依据类

```java
package com.huanghe.spring_boot_starter_hello;

/**
 * @Author: River
 * @Date:Created in  21:25 2018/10/15
 * @Description:
 */
public class HelloService {

    private String msg;

    public String sayHello(){
        return "Hello" + msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

}

```

本例子就是根据此类的存在与否来创建这个类的Bean,这个类可以是第三方的类库，这个类也就是我们以后需要使用到的类

（4）自动配置类

```java
package com.huanghe.spring_boot_starter_hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: River
 * @Date:Created in  21:26 2018/10/15
 * @Description:
 */

@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class)
@ConditionalOnClass(HelloService.class)
@ConditionalOnProperty(prefix = "hello", value = "enable", matchIfMissing = true)
public class HelloServiceAutoConfiguration {

    @Autowired
    private HelloServiceProperties helloServiceProperties;

    @Bean
    @ConditionalOnMissingBean(HelloService.class)
    public HelloService helloService() {
        HelloService helloService = new HelloService();
        helloService.setMsg(helloServiceProperties.getMsg());
        return helloService;
    }

}


```

代码就不多作解释了，上面的案例已经说的是非常的明白了

（5）注册配置

在前面我们知道，若果想要配置文件生效，我们就需要注册配置类。在src/main/resources下建立文件

![1539612462967](../../../../AppData/Roaming/Typora/typora-user-images/1539612462967.png)

（6）maven clean install装载jar包到我自己的maven仓库里