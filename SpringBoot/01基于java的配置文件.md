# 基于java的配置文件

####基于注解配置的方式

其中我们需要使用基于注解配置

```java
package com.huanghe.chapter1;
@Configuration
@ComponentScan("com.huanghe.chapter1")
public class DiConfig {

}

```

`@Configuration`: 代表这是一个配置文件

`@ComponentScan`:扫描指定表下的Component（@Componment，@Configuration，@Service) 等等,如果是没有其他的配置的情况下，默认是会扫描与配置类相同的包，在这个简单的示例中就是扫面的是`com.huanghe.chapter1;`这个包及其子包；这个在基于XML的配置文件中就是相当于是使用的是`<context:component-scan base-package ='com.huanghe.chapter1'>`

**设置扫描的基础包**

如果想扫描多个包，那该怎么办呢？或者是，如果你想扫描多个基础包，那又该怎么办呢？

可以通过设置@ComponentScan的value属性中指明包的名称，也可以使用`@ComponentScan(basePackages = {"com.huanghe.*"})`设置相应的配置文件



#### Spring与Junit4整和之后的测试

```java
package com.huanghe.chapter1;

import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * @Author: River
 * @Date:Created in  10:24 2018/9/25
 * @Description:
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = DiConfig.class)
public class USerFunctionServiceTest {
    @Autowired
    public USerFunctionService uSerFunctionService;

    @org.junit.Test
    public void sayHello() throws Exception {
        System.out.println(uSerFunctionService.sayHello());
    }

}
```

> @ContextConfiguration Spring整合JUnit4测试时，使用注解引入多个配置文件
>
> **单个文件** 
> @ContextConfiguration(Locations="../applicationContext.xml")  
>
> @ContextConfiguration(classes = SimpleConfiguration.class)
>
> **多个文件时，可用{}**
>
> @ContextConfiguration(locations = { "classpath*:/spring1.xml", "classpath*:/spring2.xml" })

#### 使用基于java配置

```java
@Configuration
public class JavaConfig {

    @Bean
    public FunctionService functionService() {
        return new FunctionService();
    }

    @Bean
    public UseFunctionService useFunctionService(){
        UseFunctionService useFunctionService = new UseFunctionService();
        useFunctionService.setService(functionService());
        return useFunctionService;
    }

    @Bean
    public UseFunctionService useFunctionService2(FunctionService functionService){
        UseFunctionService useFunctionService = new UseFunctionService();
        useFunctionService.setService(functionService);
        return useFunctionService;
    }
}
```

前面的是开启注解扫描，然后使用的是基于注解的配置，现在使用的是基于java的配置，使用的是@Bean的方式进行配置的，@Bean注解使用在方法上，声明当前方法返回的是一个bean

@Bean有value和name属性，这两个属性的含义是相同的，在代码中定义了别名。为bean起一个名字，如果默认是没有写该属性，那么就使用方法的名称作为该bean的名称