# Condition

Spring4提供了一个更加通用的基于条件的Bean的创建，即使用@Condition注解

@Condition根据满足某一个特定条件创建一个特定的Bean。比方说，当某一个jar包在一个类路径下的时候，自动配置一个或者是多个bean；或者是只有某个Bean在被创建才会去创建另外的一个Bean。总的来说，就是根据特定的条件来控制Bean的创建的方式，这样我们就可以利用这个特性进行一些自动的装配

下面演示的是在不同的操作系统中执行不同的方式

除了自己自定义Condition之外，Spring还提供了很多Condition给我们用

@ConditionalOnBean（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
 @ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean）
 @ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
 @ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
 @ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
 @ConditionalOnNotWebApplication（不是web应用）

**判断条件定义**

（1）判定Windows的条件

```java
package com.huanghe.chapter12;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * @Author: River
 * @Date:Created in  10:40 2018/10/8
 * @Description:
 */
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return conditionContext.getEnvironment().getProperty("os.name").contains("windows");
    }
}
```

(2):判断Linux的条件

```java
package com.huanghe.chapter12;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * @Author: River
 * @Date:Created in  10:41 2018/10/8
 * @Description:
 */
public class LinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return conditionContext.getEnvironment().getProperty("os.name").contains("Linux");
    }
}

```

**不同的系统下Bean的类**

接口

```java
package com.huanghe.chapter12;

/**
 * @Author: River
 * @Date:Created in  10:42 2018/10/8
 * @Description:
 */
public interface ListService {
    public String showListCmd();
}

```

(2):windows条件下需要创建的类

```java
package com.huanghe.chapter12;

/**
 * @Author: River
 * @Date:Created in  10:42 2018/10/8
 * @Description:
 */
public class WindowsListService implements ListService {

    @Override
    public String showListCmd() {
        return "dir";
    }
}

```

(3)Linux条件下需要创建的类

```java
package com.huanghe.chapter12;

/**
 * @Author: River
 * @Date:Created in  10:43 2018/10/8
 * @Description:
 */
public class LinuxListService implements ListService {
    @Override
    public String showListCmd() {
        return "ls";
    }
}

```

**配置类**

```java
package com.huanghe.chapter12;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: River
 * @Date:Created in  10:44 2018/10/8
 * @Description:
 */
@Configuration
public class ConditionConfig {
    @Bean
    @Conditional(WindowsCondition.class)
    public ListService windowsListService() {
        return new WindowsListService();
    }

    @Bean
    @Conditional(LinuxCondition.class)
    public ListService linuxListServie() {
        return new LinuxListService();
    }

}
```

- 通过Condition注解满足不同条件的类进行不同的初始化的动作

测试代码：

```java
package com.huanghe.chapter12;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @Author: River
 * @Date:Created in  10:52 2018/10/8
 * @Description:
 */
public class Main {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ConditionConfig.class);
        ListService service = context.getBean(ListService.class);
        System.out.println(context.getEnvironment().getProperty("os.name") + "系统的列表命令为：" + service.showListCmd());
    }
}

```



