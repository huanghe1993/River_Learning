# spring中用joinpoint来访问目标方法的参数

总结:访问目标方法参数,有三种方法(实际有四种,先说三种)

1.joinpoint.getargs():获取带参方法的参数

2.joinpoint.getTarget():.获取他们的目标对象信息

3..joinpoint.getSignature():(signature是信号,标识的意思):获取被增强的方法相关信息

**先看Signature方法**

```java
 @After("annotationPointCut()")
    public void after(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        System.out.println(signature.getName());
        System.out.println(signature.getDeclaringType());
        Method method = signature.getMethod();
        Action action = method.getAnnotation(Action.class);
        System.out.println("注解是拦截" + action.name());
    }
```

> ```
> signature.getName()获取的是目标类的方法名称 //add
> 
> signature.getDeclaringType()：返回的是目标方法的包名和类名 //class com.huanghe.chapter3.DemoAnnotationService
> 
> signature.getMethod();//public void com.huanghe.chapter3.DemoAnnotationService.add()
> 
> ```