# SpringBoot —— AOP注解式拦截与方法规则拦截]

AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法，所以它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件。

　　SpringBoot中AOP的使用方式主要有两种：注解式拦截与方法规则拦截，具体使用如下文所示。

一、创建一个简单springboot 2.03项目，添加aop依赖

```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

　此依赖已包含AspectJ相关依赖包。

二、编写拦截规则的注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Action {
    String name() default "";
}
```

三、编写使用注解的被拦截类

```java
@Service
public class DemoAnnotationService {

    @Action(name = "注解式拦截add操作")
    public void add(){
        System.out.println("目标方法add操作");
    }

}
```

四、编写切面

```java
@Aspect
@Component
public class LogAspect {

    //提高可扩展性
    @Pointcut("@annotation(com.huanghe.chapter3.Action)")
    public void annotationPointCut() {
    }

    @After("annotationPointCut()")
    public void after(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Action action = method.getAnnotation(Action.class);
        System.out.println("注解式拦截" + action.name());
    }
```

五、测试代码

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AopConfig.class)
public class DemoAnnotationServiceTest {

    @Autowired
    public DemoAnnotationService demoAnnotationService;

    @Autowired
    public DemoMethodService demoMethodService;

    @Test
    public void add() {
        demoAnnotationService.add();
        demoMethodService.add();
    }
}
```



