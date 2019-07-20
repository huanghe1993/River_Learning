# 组合注解和元注解

所谓的元注解就是基础的注解，可以注解在其他的注解上的注解，组合注解具备了元注解的功能。Spring的很多的注解都是可以作为元注解进行使用的，而且Spring本身具有非常多的组合注解，比如说@Configuration就是一个组合@Component的注解，表明 这个类其实也是一个Bean

```java
package com.huanghe.chapter13;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.lang.annotation.*;

/**
 * @Author: River
 * @Date:Created in  15:15 2018/10/8
 * @Description:
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@ComponentScan
public @interface WislyConfiguration {
    String[] value() default {};
}

```

```java
package com.huanghe.chapter13;

import org.springframework.stereotype.Service;

/**
 * @Author: River
 * @Date:Created in  15:17 2018/10/8
 * @Description:
 */
@Service
public class DemoService {

    public void outputResult(){
        System.out.println("从组合注解里面配置");
    }
}

```

```java
package com.huanghe.chapter13;

/**
 * @Author: River
 * @Date:Created in  15:18 2018/10/8
 * @Description:
 */
@WislyConfiguration("com.huanghe.chapter13")
public class DemoConfig {
}

```

