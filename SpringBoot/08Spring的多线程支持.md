# Spring对多线程的支持

 Spring通过任务执行器（TaskExecutor）来实现多线程和并发编程，使用ThreadPoolTaskExecutor可以实现一个基于线程池的TaskExecutor.而实际的开发中任务一般是非阻碍的也就是异步的，所以我们需要在配置类中通过@EnableAsys开启对异步任务的支持，并通过在实际执行的bean方法中使用@Async注解来声明其是一个异步的任务

代码实现

```java
package com.huanghe.chapter10;

import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

/**
 * @Author: River
 * @Date:Created in  9:41 2018/10/8
 * @Description:
 */
@Configuration
@ComponentScan("com.huanghe.chapter10")
@EnableAsync
public class TaskExecutorConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(25);
        taskExecutor.initialize();
        return taskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}

```

- 利用@EnableAsync注解开启异步任务支持
- 配置类的实现AsyncConfigurer接口重写getAsyncExecutor方法，并返回一个ThreadPoolTaskExecutor，这样我们就获得了一个基于线程池的TaskExecutor

任务类

```java
package com.huanghe.chapter10;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

/**
 * @Author: River
 * @Date:Created in  9:47 2018/10/8
 * @Description:
 */
@Service
public class AsyncTaskService {

    @Async
    public void executeAsyncTask(Integer i) {
        System.out.println("执行异步任务："+i);
    }

    @Async
    public void executeAsyncTaskPlus(Integer i){
        System.out.println("执行异步任务+1：" + (i+1));
    }
}

```

测试代码

```java
package com.huanghe.chapter10;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.*;

/**
 * @Author: River
 * @Date:Created in  9:51 2018/10/8
 * @Description:
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = TaskExecutorConfig.class)
public class AsyncTaskServiceTest {

    @Autowired
    private AsyncTaskService asyncTaskService;

    @Test
    public void testMethod(){
        for (int i = 0; i < 10; i++) {
            asyncTaskService.executeAsyncTask(i);
            asyncTaskService.executeAsyncTaskPlus(i);
        }
    }

}
```

![1538963899621](../../../../AppData/Roaming/Typora/typora-user-images/1538963899621.png)

但是已经能够看出了，假如是原来那样的，是同步执行，那么肯定偶数行的输出比前一个奇数行的输出是大1的。

 结果不是那样，它们是异步进行的，在这里由一个主线程(main线程)。  for循环里面，每运行一行调用方法的，就会开一个线程。  也就是说，你每次的运行结果可能会不一样！ 
 

 