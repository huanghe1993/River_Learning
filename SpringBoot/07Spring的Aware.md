# Spring Aware

Spring的依赖注入的最大的亮点就是所有的Bean对Spring容器的存在是没有意识的。即你可以将你的容器替换成别的容器，如Google Guice ，这时Bean之间的耦合度很低。

但是在实际的项目中，你不可避免的要用到Spring容器本身的功能资源，这个时候你的Bean必须要意识到Spring容器的存在，才能调用Spring所提供的资源，这就是所谓的Spring Aware 。其实Spring Aware本来就是Spring设计用来框架内部使用的，若是使用了Spring Aware,你的Bean将会和Spring框架耦合在一起

| Aware接口                      | 作用                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| BeanNameAware                  | 获取容器中的Bean的实例的名字                                 |
| BeanFactoryAware               | 获取当前的bean factory,这样可以调用容器服务                  |
| ApplicationContextAware        | 获取到当前的application context，这样可以调用容器服务        |
| MessageSourceAware             | 获得message source，这样可以获得文本资源                     |
| ApplicationEventPublisherAware | 在bean中可以得到应用上下文的事件发布器，从而可以在Bean中发布应用上下文的事件。 |
| ResourceLoaderAware            | 在Bean中可以得到ResourceLoader，从而在bean中使用ResourceLoader加载外部对应的Resource资源。 |

Spring Aware的目的就是为了让Bean获得Spring容器的服务。因为ApplicationContext接口集成了MessageSource接口、ApplicationEventPublisher接口和ResourceLoader接口，所以Bean继承ApplicationContextAware可以获得Spring的所有的服务，但是原则上是我们用到什么，就实现什么样的接口

- 实现接口，获得到Bean的资源和加载的服务
- 实现ResourceLoaderAware需要重写setResourceLoader方法

```java
package com.huanghe.chapter9;

import com.sun.org.apache.xpath.internal.SourceTree;
import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

import java.io.IOException;

/**
 * @Author: River
 * @Date:Created in  21:20 2018/10/7
 * @Description:
 */
@Service
public class AwareService implements BeanNameAware, ResourceLoaderAware {

    private String beanName;
    private ResourceLoader loader;

    @Override
    public void setBeanName(String s) {
        this.beanName = s;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.loader = resourceLoader;
    }

    public void outputResult() {
        System.out.println("Bean的名称为：" + beanName);
        Resource resource = loader.getResource("classpath:chapter9/test.txt");
        try {
            System.out.println(IOUtils.toString(resource.getInputStream()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

