# Spring对计划任务的支持

从Spring3.1开始，计划任务在Spring中的实现变得简单。首先只是需要在配置类上添加注解@EnableScheduling来开启对计划任务的支持，然后在要执行计划任务的方法上注解@Scheduled

声明这是一个计划任务。

 Spring通过@Scheduled支持多种类型的计算任务，包含cron,fixDelay,fixRate等等

计划任务执行类

```java
package com.huanghe.chaper11;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author: River
 * @Date:Created in  10:13 2018/10/8
 * @Description:
 */
@Service
public class ScheduleTaskService {
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime(){
        System.out.println("每隔5秒执行一次" + dateFormat.format(new Date()));
    }

    @Scheduled(cron = "0 25 10 ? * *")
    public void fixTimeExecution(){
        System.out.println("在指定时间"+dateFormat.format(new Date()+"执行"));
    }
}

```

- @Scheduled声明该方法是计划任务，使用fixedRate属性每隔固定的时间执行
- 使用cron属性可以按照指定时间执行，本例子是每天的10点25执行



配置类：

```java
package com.huanghe.chaper11;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * @Author: River
 * @Date:Created in  10:17 2018/10/8
 * @Description:
 */
@Configuration
@ComponentScan("com.huanghe.chaper11")
@EnableScheduling
public class TaskSchedulerConfig {

}

```

`@EnableScheduling`是开启对计划任务的支持

