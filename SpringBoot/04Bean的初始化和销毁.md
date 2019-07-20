# Bean的初始化和销毁

在我们开发的时候经常会遇到Bean在使用之前需要做的事情和使用之后需要做的事情，Spring对Bean的生命周期提供了支持。在使用Java的配置和注解的配置提供了如下的两种方式

- Java的配置方式：使用的是@Bean的initMethod和destoryMethod(相当于是xml配置文件中的init-method和destory-method的方式进行)
- 注解的方式进行配置：利用的是JSR-250的@PostConstruct和@PreDestory

具体的更加详细的可以查看Spring的生命周期

下面是关于上述的一个演示的示例

基于java配置方式进行的：

```java
package com.huanghe.chapter6;

/**
 * @Author: River
 * @Date:Created in  17:23 2018/10/7
 * @Description:
 */
public class BeanWayService {

    public void init(){
        System.out.println("@Bean-init-method");
    }

    public BeanWayService() {
        super();
        System.out.println("初始化构造函数-BeanWayService");
    }

    public void destory(){
        System.out.println("@Bean-destory-method");
    }

    public String testMethod(){
        return "BeanWay";
    }
}
```

基于注解的方式进行的：

```java
package com.huanghe.chapter6;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Author: River
 * @Date:Created in  17:25 2018/10/7
 * @Description:
 */
public class JSR250WayService {

    @PostConstruct
    public void init(){
        System.out.println("JSR250-init-method");
    }

    public JSR250WayService() {
        super();
        System.out.println("初始化构造函数-JSR250方式");
    }

    @PreDestroy
    public void destory(){
        System.out.println("JSR-250-destory-method");
    }

    public String testMethod(){
        return "JSR-way";
    }
}

```

配置文件

```JAVA
package com.huanghe.chapter6;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: River
 * @Date:Created in  17:28 2018/10/7
 * @Description:
 */
@Configuration
@ComponentScan("com.huanghe.chapter6")
public class PrePostConfig {

    @Bean(initMethod = "init", destroyMethod = "destory")
    BeanWayService beanWayService(){
        return new BeanWayService();
    }

    @Bean
    JSR250WayService jsr250WayService(){
        return new JSR250WayService();
    }

}


```

