# @ControllerAdice

`@ControllerAdvice`，是Spring3.2提供的新注解，从名字上可以看出大体意思是控制器增强。让我们先看看`@ControllerAdvice`的实现：

```java
package org.springframework.web.bind.annotation;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {

    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<?>[] assignableTypes() default {};

    Class<? extends Annotation>[] annotations() default {};
}
```

- `@ControllerAdvice`是一个`@Component`，用于定义`@ExceptionHandler`，`@InitBinder`和`@ModelAttribute`方法，适用于所有使用`@RequestMapping`方法。

- Spring4之前，`@ControllerAdvice`在同一调度的Servlet中协助所有控制器。Spring4已经改变：`@ControllerAdvice`支持配置控制器的子集，而默认的行为仍然可以利用。

- 在Spring4中， `@ControllerAdvice`通过`annotations()`, `basePackageClasses()`, `basePackages()`方法定制用于选择控制器子集。

  不过据经验之谈，只有配合`@ExceptionHandler`最有用，其它两个不常用。