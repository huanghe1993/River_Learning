# profile

## 前沿

由于在项目中使用Maven打包部署的时候，经常由于配置参数过多(比如Nginx服务器的信息、ZooKeeper的信息、数据库连接、Redis服务器地址等)，导致实际现网的配置参数与测试服务器参数混淆，一旦在部署的时候某个参数忘记修改了，那么就必须重新打包部署，这确实让人感到非常头疼。因此就想到使用Spring中的Profile来解决上面描述的问题，并且在此记录一下其使用的方式，

本文从如下3方面探讨Spring的Profile：

- Spring中的Profile是什么
- 为什么要使用Profile
- 如何使用Profile

## 1.Spring中的Profile 是什么?

Spring中的Profile功能其实早在Spring 3.1的版本就已经出来，它可以理解为我们在Spring容器中所定义的Bean的**逻辑组名称**，只有当这些Profile被激活的时候，才会将Profile中所对应的Bean注册到Spring容器中。举个更具体的例子，我们以前所定义的Bean，当Spring容器一启动的时候，就会一股脑的全部加载这些信息完成对Bean的创建；而使用了Profile之后，它会将Bean的定义进行更细粒度的划分，将这些定义的Bean划分为几个不同的组，当Spring容器加载配置信息的时候，首先查找激活的Profile，然后只会去加载被激活的组中所定义的Bean信息，而不被激活的Profile中所定义的Bean定义信息是不会加载用于创建Bean的。

## 2.为什么要使用Profile

由于我们平时在开发中，通常会出现在开发的时候使用一个开发数据库，测试的时候使用一个测试的数据库，而实际部署的时候需要一个数据库。以前的做法是将这些信息写在一个配置文件中，当我把代码部署到测试的环境中，将配置文件改成测试环境；当测试完成，项目需要部署到现网了，又要将配置信息改成现网的，真的好烦。。。而使用了Profile之后，我们就可以分别定义3个配置文件，一个用于开发、一个用户测试、一个用户生产，其分别对应于3个Profile。当在实际运行的时候，只需给定一个参数来激活对应的Profile即可，那么容器就会只加载激活后的配置文件，这样就可以大大省去我们修改配置信息而带来的烦恼。

## 3.配置Spring profile

在介绍完Profile以及为什么要使用它之后，下面让我们以一个例子来演示一下Profile的使用，这里还是使用传统的XML的方式来完成Bean的装配。

### 3.1 例子需要的Maven依赖

由于只是做一个简单演示，因此无需引入Spring其他模块中的内容，只需引入核心的4个模块+测试模块即可。

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!--指定Spring版本，该版本必须大于等于3.1-->
        <spring.version>4.2.4.RELEASE</spring.version>
        <!--指定JDK编译环境-->
        <java.version>1.7</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>


        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 3.2 例子代码

```java
package com.panlingxiao.spring.profile.service;

/**
 * 定义接口,在实际中可能是一个数据源
 * 在开发的时候与实际部署的时候分别使用不同的实现
 */
public interface HelloService {

    public String sayHello();
}

```

> 定义生产环境使用的实现类

```java
package com.panlingxiao.spring.profile.service.produce;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import com.panlingxiao.spring.profile.service.HelloService;

/**
 * 模拟在生产环境下需要使用的类
 */
@Component
public class ProduceHelloService implements HelloService {
    
    //这个值读取生产环境下的配置注入
    @Value("#{config.name}")
    private String name;

    public String sayHello() {
        return String.format("hello,I'm %s,this is a produce environment!",
                name);
    }
}
```

> 定义开发下使用的实现类

```java
package com.panlingxiao.spring.profile.service.dev;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import com.panlingxiao.spring.profile.service.HelloService;

/**
 * 模拟在开发环境下使用类
 */
@Component
public class DevHelloService implements HelloService{
    
    //这个值是读取开发环境下的配置文件注入
    @Value("#{config.name}")
    private String name;
    
    public String sayHello() {
        return String.format("hello,I'm %s,this is a development environment!", name);
    }

}

```

> 定义配置Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">
    
    <!-- 定义开发的profile -->
    <beans profile="development">
        <!-- 只扫描开发环境下使用的类 -->
        <context:component-scan base-package="com.panlingxiao.spring.profile.service.dev" />
        <!-- 加载开发使用的配置文件 -->
        <util:properties id="config" location="classpath:dev/config.properties"/>
    </beans>

    <!-- 定义生产使用的profile -->
    <beans profile="produce">
        <!-- 只扫描生产环境下使用的类 -->
        <context:component-scan
            base-package="com.panlingxiao.spring.profile.service.produce" />
        <!-- 加载生产使用的配置文件 -->    
        <util:properties id="config" location="classpath:produce/config.properties"/>
    </beans>
</beans>
```

> 开发使用的配置文件，dev/config.properties

```java
name =tomcat
```

> 生产使用的配置文件，produce/config.properties

```java
name =Jetty
```

> 编写测试类

```java
package com.panlingxiao.spring.profile.test;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.panlingxiao.spring.profile.service.HelloService;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:spring-profile.xml")
/*
 * 使用注册来完成对profile的激活,
 * 传入对应的profile名字即可,可以传入produce或者dev
 */
@ActiveProfiles("produce")
public class TestActiveProfile {

    @Autowired
    private HelloService hs;
    
    @Test
    public void testProfile() throws Exception {
        String value = hs.sayHello();
        System.out.println(value);
    }
}

```

### 4.激活Profile的其他几种方式

上面介绍了如何使用Profile以及在单元测试的环境下激活指定的Profile，除了使用@ActiveProfiles注解来激活profile外，Spring还提供了其他的几种激活Profile，这些方式在实际的开发中使用的更多。
 Spring通过两个不同属性来决定哪些profile可以被激活(注意：profile是可以同时激活多个的),一个属性是**spring.profiles.active和spring.profiles.default**。这两个常量值在Spring的AbstractEnvironment中有定义，查看AbstractEnvironment源码：

```java
/**
     * Name of property to set to specify active profiles: {@value}. Value may be comma
     * delimited.
     * <p>Note that certain shell environments such as Bash disallow the use of the period
     * character in variable names. Assuming that Spring's {@link SystemEnvironmentPropertySource}
     * is in use, this property may be specified as an environment variable as
     * {@code SPRING_PROFILES_ACTIVE}.
     * @see ConfigurableEnvironment#setActiveProfiles
     */
    public static final String ACTIVE_PROFILES_PROPERTY_NAME = "spring.profiles.active";

    /**
     * Name of property to set to specify profiles active by default: {@value}. Value may
     * be comma delimited.
     * <p>Note that certain shell environments such as Bash disallow the use of the period
     * character in variable names. Assuming that Spring's {@link SystemEnvironmentPropertySource}
     * is in use, this property may be specified as an environment variable as
     * {@code SPRING_PROFILES_DEFAULT}.
     * @see ConfigurableEnvironment#setDefaultProfiles
     */
    public static final String DEFAULT_PROFILES_PROPERTY_NAME = "spring.profiles.default";

```

如果当spring.profiles.active属性被设置时，那么Spring会优先使用该属性对应值来激活Profile。当spring.profiles.active没有被设置时，那么Spring会根据spring.profiles.default属性的对应值来进行Profile进行激活。如果上面的两个属性都没有被设置，那么就不会有任务Profile被激活，只有定义在Profile之外的Bean才会被创建。我们发现这两个属性值其实是Spring容器中定义的属性，而我们在实际的开发中很少会直接操作Spring容器本身，所以如果要设置这两个属性，其实是需要定义在特殊的位置，让Spring容器自动去这些位置读取然后自动设置,这些位置主要为如下定义的地方：

- 作为SpringMVC中的DispatcherServlet的初始化参数
- 作为Web 应用上下文中的初始化参数
- 作为JNDI的入口
- 作为环境变量
- 作为虚拟机的系统参数
- 使用@AtivceProfile来进行激活

我们在实际的使用过程中，可以定义默认的profile为开发环境，当实际部署的时候，主需要在实际部署的环境服务器中将spring.profiles.active定义在环境变量中来让Spring自动读取当前环境下的配置信息，这样就可以很好的避免不同环境而频繁修改配置文件的麻烦。

## 5、使用Java配置的方式

示例Bean

```java
package com.huanghe.chapter7;

/**
 * @Author: River
 * @Date:Created in  20:07 2018/10/7
 * @Description:
 */
public class DemoBean {

    private String content;

    public DemoBean(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}

```

Profile的配置

```java
package com.huanghe.chapter7;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

/**
 * @Author: River
 * @Date:Created in  20:08 2018/10/7
 * @Description:
 */
@Configuration
public class ProfileConfig {

    @Bean
    @Profile("dev")
    public DemoBean devDemoBean(){
        return new DemoBean("from dev profile");
    }

    @Bean
    @Profile("prod")
    public DemoBean prodDemoBean(){
        return new DemoBean("from production profile");
    }

}
```

测试代码：

```java
package com.huanghe.chapter7;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.*;

/**
 * @Author: River
 * @Date:Created in  20:11 2018/10/7
 * @Description:
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ProfileConfig.class)
@ActiveProfiles("dev")
public class DemoBeanTest {

    @Autowired
    private DemoBean demoBean;

    @Test
    public void testProfile() {
        System.out.println(demoBean.getContent());
    }

}
```

