# Spring的Aware接口

　　容器管理的Bean一般不需要了解容器的状态和直接使用容器，但是在某些情况下，是需要在Bean中直接对IOC容器进行操作的，这时候，就需要在Bean中设定对容器的感知。Spring IOC容器也提供了该功能，它是通过特定的Aware接口来完成的。

　　Spring`的依赖注入的最大亮点就是你所有的`Bean`对`Spring`容器的存在是没有意识的。即你可以将你的容器替换成别的容器，例如`Goggle Guice`,这时`Bean`之间的耦合度很低。

　　但是在实际的项目中，我们不可避免的要用到`Spring`容器本身的功能资源，**这时候`Bean`必须要意识到`Spring`容器的存在，才能调用`Spring`所提供的资源，这就是所谓的`Spring ``Aware`**。其实`Spring` `Aware`本来就是`Spring`设计用来框架内部使用的，若使用了`Spring ``Aware`，你的`Bean`将会和`Spring`框架耦合。

### Spring提供的Aware接口

| 名称                           | 作用                                                  |
| ------------------------------ | ----------------------------------------------------- |
| BeanNameAware                  | 获取到当前容器中Bean的名称                            |
| BeanFactoryAware               | 获取到当前的BeanFactory,这样可以调用服务              |
| ApplicationContextAware*       | 获取当前的Application context，这样可以调用容器的服务 |
| MessageSourceAware             | 调用message source这样可以获得文本信息                |
| ApplicationEventPublisherAware | 应用事件发布器，可以发布事件                          |
| ResourceLoaderAware            | 获得资源加载器，可以获得外部资源文件                  |
|                                |                                                       |
|                                |                                                       |
|                                |                                                       |
|                                |                                                       |

**Spring Aware的目的是为了让Bean获得Spring容器的服务**。因为ApplicationContext接口集成了MessageSource接口，ApplicationEventPublisherAware接口和ResourceLoaderAware接口，所以Bean继承ApplicationContextAware可以获得Spring容器的所有服务，但原则上我们还是用到什么接口就实现什么接口。

## 二. 示例

## 1. 准备。

在`org.light4j.sping4.senior.aware`包下新建一个`test.txt`,内容随意，给下面的外部资源加载使用。

## 2. Spring Aware演示Bean

```java
package org.light4j.sping4.senior.aware;

import java.io.IOException;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

@Service
public class AwareService implements BeanNameAware,ResourceLoaderAware{//①

    private String beanName;
    private ResourceLoader loader;

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {//②
        this.loader = resourceLoader;
    }

    //该方法可以传递name属性 ，name:表示的是正在被实例化的bean的id
    @Override
    public void setBeanName(String name) {//③
        this.beanName = name;
    }

    public void outputResult(){
        System.out.println("Bean的名称为：" + beanName);

        Resource resource = 
                loader.getResource("classpath:org/light4j/sping4/senior/aware/test.txt");
        try{

            System.out.println("ResourceLoader加载的文件内容为: " + IOUtils.toString(resource.getInputStream()));

           }catch(IOException e){
            e.printStackTrace();
           }
    }
}
```

## 代码解释：

> ① 实现`BeanNameAware`,`ResourceLoaderAware`接口，获得`Bean`名称和资源加载的服务。
> ② 实现`ResourceLoaderAware`需要重写`setResourceLoader`方法。
> ③ 实现`BeanNameAware`需要重写`setBeanName`方法。

## 3. 配置类

```java
package org.light4j.sping4.senior.aware;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
@Configuration
@ComponentScan("org.light4j.sping4.senior.aware")
public class AwareConfig {

}
```

## 4. 运行

```java
package org.light4j.sping4.senior.aware;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {

        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AwareConfig.class);

        AwareService awareService = context.getBean(AwareService.class);
        awareService.outputResult();

        context.close();
    }
}
```

