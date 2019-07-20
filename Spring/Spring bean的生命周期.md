# Spring bean的生命周期

再谈生命周期之前有一点需要先明确： 

> Spring 只帮我们管理单例模式 Bean 的**完整**生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。 

![img](https://upload-images.jianshu.io/upload_images/3131012-0fdb736b21c8cc31.png?imageMogr2/auto-orient/) 

1：实例化

当程序加载springconfig.xml配置文件的时候，会对scope为singleton且非懒加载的bean进行实例化 

2：设置属性

按照Bean定义信息配置信息，注入所有的属性 （调用set方法）

3：如果实现了BeanNameAware接口

则可以通过setBeanName获取到实例化对象bean的id

4：如果实现了BeanFactoryAware接口

会回调该接口的setBeanFactory()方法，传入该Bean的BeanFactory，这样该Bean就获得了自己所在的BeanFactory 

5：如果Bean实现了ApplicationContextAware接口,会回调该接口的setApplicationContext()方法，传入该Bean的ApplicationContext，这样该Bean就获得了自己所在的ApplicationContext， 

6：如果bean与一个后置处理器BeanPostProcessors 相关联，则会自动的去调用BeanPostProcessors 的

postProcessBeforeInitialzation()方法，并不会立刻的去调用postProcessAfterInitialzation()方法，在调用完InitializingBean的afterpropertiesSet()方法，和调用完定制的初始化方法之后才会去调用postProcessAfterInitialzation()方法

7：如果Bean实现了InitializingBean接口，则会回调该接口的afterPropertiesSet()方法， 

8：如果Bean配置了init-method方法，也可以使用注解的方式进行配置@PostConstruct，则会执行init-method配置的方法， 

9：如果有Bean实现了BeanPostProcessor接口，则会回调该接口的postProcessAfterInitialization()方法， 

10：经过流程9之后，就可以正式使用该Bean了,对于scope为singleton的Bean,Spring的ioc容器中会缓存一份该bean的实例，而对于scope为prototype的Bean,每次被调用都会new一个新的对象，期生命周期就交给调用方管理了，不再是Spring容器进行管理了

11：当要销毁Bean的时候 ，如果Bean实现了DisposableBean接口，则会回调该接口的destroy()方法，在这里我们可以关闭数据连接，socket连接，文件流，释放该bean的资源

12：当要销毁Bean的时候 ，如果Bean配置了destroy-method方法，则会执行destroy-method配置的方法(注解的方式是@PreDestory)，至此，整个Bean的生命周期结束

 ### BeanFactory Bean生命周期

 ![img](https://upload-images.jianshu.io/upload_images/3131012-249748bc2b49e857.png?imageMogr2/auto-orient/) 

 BeanFactoty容器中, Bean的生命周期如上图所示，与ApplicationContext相比，有如下几点不同:

1.BeanFactory容器中，不会调用ApplicationContextAware接口的setApplicationContext()方法，

2.BeanPostProcessor接口的postProcessBeforeInitialzation()方法和postProcessAfterInitialization()方法不会自动调用，必须自己通过代码手动注册

3.BeanFactory容器启动的时候，不会去实例化所有Bean,包括所有scope为singleton且非懒加载的Bean也是一样，而是在调用的时候去实例化。