# SpringMVC中的统一的异常处理

[TOC]




我们知道，系统中异常包括：编译时异常和运行时异常RuntimeException，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。在开发中，不管是dao层、service层还是controller层，都有可能抛出异常，在springmvc中，能将所有类型的异常处理从各处理过程解耦出来，既保证了相关处理过程的功能较单一，也实现了异常信息的统一处理和维护。这篇博文主要总结一下SpringMVC中如何统一处理异常。

## **1. 异常处理思路**

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/181009/Eghb9eg17e.png?imageslim)

如上图所示，系统的dao、service、controller出现异常都通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理。springmvc提供全局异常处理器（一个系统只有一个异常处理器）进行统一异常处理。明白了springmvc中的异常处理机制，下面就开始分析springmvc中的异常处理。

**Spring MVC处理异常有3种方式：** 
（1）使用Spring MVC提供的简单异常处理器SimpleMappingExceptionResolver； 
（2）实现Spring的异常处理接口HandlerExceptionResolver 自定义自己的异常处理器； 
（3）使用@ExceptionHandler注解实现异常处理； 

## **2. springmvc中自带的简单异常处理器**

springmvc中自带了一个异常处理器叫SimpleMappingExceptionResolver，该处理器实现了HandlerExceptionResolver 接口，全局异常处理器都需要实现该接口。我们要使用这个自带的异常处理器，首先得在springmvc.xml文件中配置该处理器：

```xml
<!-- springmvc提供的简单异常处理器 -->
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
     <!-- 定义默认的异常处理页面 -->
    <property name="defaultErrorView" value="/WEB-INF/jsp/error.jsp"/>
    <!-- 定义异常处理页面用来获取异常信息的变量名，也可不定义，默认名为exception --> 
    <property name="exceptionAttribute" value="ex"/>
    <!-- 定义需要特殊处理的异常，这是重要点 --> 
    <property name="exceptionMappings">
        <props>
            <prop key="ssm.exception.CustomException">/WEB-INF/jsp/custom_error.jsp</prop>
        </props>
        <!-- 还可以定义其他的自定义异常 -->
    </property>
</bean>
```

从上面的配置来看，最重要的是要配置特殊处理的异常，这些异常一般都是我们自定义的，根据实际情况来自定义的异常，然后也会跳转到不同的错误显示页面显示不同的错误信息。这里就用一个自定义异常CustomException来说明问题，定义如下：

```java
//定义一个简单的异常类
public class CustomException extends Exception {

    //异常信息
    public String message;

    public CustomException(String message) {
        super(message);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

}

```

接下来就是写测试程序了，还是使用查询的例子，如下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/181009/Jb13FlfmKE.png?imageslim)

然后我们在前台输入url来测试：<http://localhost:8080/SpringMVC_Study/editItems.action?id=11>，故意传一个id为11，我的数据库中没有id为11的项，所以肯定查不到，反正让它查不到即可。这样它就会抛出自定义的异常，然后被上面配置的全局异常处理器捕获并执行，跳转到我们指定的页面，然后显示一下该商品不存在即可。所以这个流程是很清晰的。 
从上面的过程可知，使用SimpleMappingExceptionResolver进行异常处理，具有集成简单、有良好的扩展性（可以任意增加自定义的异常和异常显示页面）、对已有代码没有入侵性等优点，但该方法仅能获取到异常信息，若在出现异常时，对需要获取除异常以外的数据的情况不适用。

## **3. 自定义全局异常处理器**

　　全局异常处理器处理思路：

> 1. 解析出异常类型
> 2. 如果该异常类型是系统自定义的异常，直接取出异常信息，在错误页面展示
> 3. 如果该异常类型不是系统自定义的异常，构造一个自定义的异常类型（信息为“未知错误”）

　　springmvc提供一个HandlerExceptionResolver接口，自定义全局异常处理器必须要实现这个接口，如下：

```java
public class CustomExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex) {

        ex.printStackTrace();
        CustomException customException = null;

        //如果抛出的是系统自定义的异常则直接转换
        if(ex instanceof CustomException) {
            customException = (CustomException) ex;
        } else {
            //如果抛出的不是系统自定义的异常则重新构造一个未知错误异常
            //这里我就也有CustomException省事了，实际中应该要再定义一个新的异常
            customException = new CustomException("系统未知错误");
        }

        //向前台返回错误信息
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("message", customException.getMessage());
        modelAndView.setViewName("/WEB-INF/jsp/error.jsp");

        return modelAndView;
    }
}
```

　　全局异常处理器中的逻辑很清楚，我就不再多说了，然后就是在springmvc.xml中配置这个自定义的异常处理器：

```xml
<!-- 自定义的全局异常处理器 
只要实现HandlerExceptionResolver接口就是全局异常处理器-->
<bean class="ssm.exception.CustomExceptionResolver"></bean> 123
```

　　然后就可以使用上面那个测试用例再次测试了。可以看出在自定义的异常处理器中能获取导致出现异常的对象，有利于提供更详细的异常处理信息。一般用这种自定义的全局异常处理器比较多。

## **4. @ExceptionHandler注解实现异常处理**

　　还有一种是使用注解的方法，我大概说一下思路，因为这种方法对代码的入侵性比较大，我不太喜欢用这种方法。 
 　　首先写个BaseController类，并在类中使用@ExceptionHandler注解声明异常处理的方法，如：

```java
public abstract class BaseController
 {
     @ExceptionHandler
    public String exception(HttpServletRequest request, Exception e)
 { 
        // 根据不同的异常类型进行不同处理
        if(e instanceof SQLException)
 {
           String s = "数据库异常" ;
            request.setAttribute( "exceptionMessage", s);
            return "error";
     
        }else if(e instanceof IOException){
           String s = "IO异常";
            request.setAttribute( "exceptionMessage", s);
            return "error";
     
        }
        else return "error"; 
    }
}
```

　　然后将所有需要异常处理的Controller都继承这个BaseController，虽然从执行来看，不需要配置什么东西，但是代码有侵入性，需要异常处理的Controller都要继承它才行。 
 　　 该方法需要定义在Controller内部，然后创建一个方法并用@ExceptionHandler注解来修饰用来处理异常，这个方法基本和 @RequestMapping修饰的方法差不多，只是可以多一个类型为Exception的参数，@ExceptionHandler中可以添加一个或多个异常的类型，如果为空的话则认为可以触发所有的异常类型错误，返回值为视图名。

2、以后让每个controller类继承定义的BaseController即可；

```java
@Controller
@RequestMapping(value="/login" )
public class LoginController extends BaseController{
     private Logger logger =
 Logger.getLogger(LoginController.class);
     private JsonGenerator jsonGenerator = null;
     @Autowired
     IUserService userService;
     @RequestMapping(value= "/login")
     public String login() throws Exception{
            logger.info( "login....");
            //这里模拟抛出一个SQL异常信息
            throw new SQLException();
     }
}
```

3、最后定义一个error.jsp页面

```jsp
<%@ page language =*"java"* contentType=*"text/html;  charset=utf-8"*

    pageEncoding=*"utf-8"*%>

    <%@taglib prefix =*"c"* uri=*"http://java.sun.com/jsp/jstl/core"* %>

<html>
     <body >

     ${exceptionMessage}

     </body >

</html>
```

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/181009/K3K1EJbB6I.png?imageslim)

## 5.@ControllerAdvice + @ExceptionHandler 组合进行的 Controller 层上抛的异常全局统一处理

## 二、基本使用示例

上面的写法的第三种方法每次都是需要继承BaseController，虽然从执行来看，不需要配置什么东西，但是代码有侵入性，需要异常处理的Controller都要继承它才行。我们可以使用@Controller+@ExceptionHandler的方法来解决出现的异常

### 5.1.@ControllerAdvice 注解定义全局异常处理类

```java
@ControllerAdvice
public class GlobalExceptionHandler {
}
```

请确保此 GlobalExceptionHandler 类能被扫描到并装载进 Spring 容器中。

### 5.2. @ExceptionHandler 注解声明异常处理方法

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseBody
    String handleException(){
        return "Exception Deal!";
    }
}
```

方法 handleException() 就会处理所有 Controller 层抛出的 **Exception 及其子类的异常**，这是最基本的用法了。

被 **@ExceptionHandler** 注解的方法的参数列表里，还可以声明很多种类型的参数，[详见文档](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html)。其原型如下：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {

    /**
     * Exceptions handled by the annotated method. If empty, will default to any
     * exceptions listed in the method argument list.
     */
    Class<? extends Throwable>[] value() default {};

}
```

如果 @ExceptionHandler 注解中未声明要处理的异常类型，则默认为参数列表中的异常类型。所以上面的写法，还可以写成这样：

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler()
    @ResponseBody
    String handleException(Exception e){
        return "Exception Deal! " + e.getMessage();
    }
}

```

参数对象就是 Controller 层抛出的异常对象！

## 三、处理 Service 层上抛的业务异常

有时我们会在复杂的带有数据库事务的业务中，当出现不和预期的数据时，直接抛出封装后的业务级运行时异常，进行数据库事务回滚，并希望该异常信息能被返回显示给用户。

### 3.1 代码示例

封装的业务异常类：

```java
public class BusinessException extends RuntimeException {

    public BusinessException(String message){
        super(message);
    }
}123456
```

Service 实现类：

```java
@Service
public class DogService {

    @Transactional
    public Dog update(Dog dog){

        // some database options

        // 模拟狗狗新名字与其他狗狗的名字冲突
        BSUtil.isTrue(false, "狗狗名字已经被使用了...");

        // update database dog info

        return dog;
    }

}1234567891011121314151617
```

其中辅助工具类 BSUtil

```java
public static void isTrue(boolean expression, String error){
    if(!expression) {
        throw new BusinessException(error);
    }
}
```

那么，我们应该在 GlobalExceptionHandler 类中声明该业务异常类，并进行相应的处理，然后返回给用户。更贴近真实项目的代码，应该长这样子：

```java
/**
 * Created by kinginblue on 2017/4/10.
 * @ControllerAdvice + @ExceptionHandler 实现全局的 Controller 层的异常处理
 */
@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 处理所有不可知的异常
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    @ResponseBody
    AppResponse handleException(Exception e){
        LOGGER.error(e.getMessage(), e);

        AppResponse response = new AppResponse();
        response.setFail("操作失败！");
        return response;
    }

    /**
     * 处理所有业务异常
     * @param e
     * @return
     */
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    AppResponse handleBusinessException(BusinessException e){
        LOGGER.error(e.getMessage(), e);

        AppResponse response = new AppResponse();
        response.setFail(e.getMessage());
        return response;
    }
}
```

Controller 层的代码，就不需要进行异常处理了：

```java
@RestController
@RequestMapping(value = "/dogs", consumes = {MediaType.APPLICATION_JSON_UTF8_VALUE})
public class DogController {

    @Autowired
    private DogService dogService;

    @PatchMapping(value = "")
    Dog update(@Validated(Update.class) @RequestBody Dog dog){
        return dogService.update(dog);
    }
}
```

### 3.2 代码说明

 Logger 进行所有的异常日志记录。

@ExceptionHandler(BusinessException.class) 声明了对 BusinessException 业务异常的处理，并获取该业务异常中的错误提示，构造后返回给客户端。

@ExceptionHandler(Exception.class) 声明了对 Exception 异常的处理，起到**兜底**作用，不管 Controller 层执行的代码出现了什么未能考虑到的异常，都返回统一的错误提示给客户端。

备注：以上 GlobalExceptionHandler 只是返回 Json 给客户端，更大的发挥空间需要按需求情况来做。

## 四、处理 Controller 数据绑定、数据校验的异常

在 Dog 类中的字段上的注解数据校验规则：

```java
@Data
public class Dog {

    @NotNull(message = "{Dog.id.non}", groups = {Update.class})
    @Min(value = 1, message = "{Dog.age.lt1}", groups = {Update.class})
    private Long id;

    @NotBlank(message = "{Dog.name.non}", groups = {Add.class, Update.class})
    private String name;

    @Min(value = 1, message = "{Dog.age.lt1}", groups = {Add.class, Update.class})
    private Integer age;
}12345678910111213
说明：@NotNull、@Min、@NotBlank 这些注解的使用方法，不在本文范围内。如果不熟悉，请查找资料学习即可。

其他说明：
@Data 注解是 **Lombok** 项目的注解，可以使我们不用再在代码里手动加 getter & setter。
在 Eclipse 和 IntelliJ IDEA 中使用时，还需要安装相关插件，这个步骤自行Google/Baidu 吧！
12345
```

Lombok 使用方法见：[Java奇淫巧技之Lombok](http://blog.csdn.net/ghsau/article/details/52334762)

SpringMVC 中对于 RESTFUL 的 Json 接口来说，数据绑定和校验，是这样的：

```java
/**
 * 使用 GlobalExceptionHandler 全局处理 Controller 层异常的示例
 * @param dog
 * @return
 */
@PatchMapping(value = "")
AppResponse update(@Validated(Update.class) @RequestBody Dog dog){
    AppResponse resp = new AppResponse();

    // 执行业务
    Dog newDog = dogService.update(dog);

    // 返回数据
    resp.setData(newDog);

    return resp;
}1234567891011121314151617
```

使用 `@Validated + @RequestBody` 注解实现。

当使用了 `@Validated + @RequestBody` 注解但是没有在绑定的数据对象后面跟上 `Errors` 类型的参数声明的话，Spring MVC 框架会抛出 `MethodArgumentNotValidException` 异常。

所以，在 GlobalExceptionHandler 中加上对 `MethodArgumentNotValidException` 异常的声明和处理，就可以全局处理数据校验的异常了！加完后的代码如下：

```java
/**
 * Created by kinginblue on 2017/4/10.
 * @ControllerAdvice + @ExceptionHandler 实现全局的 Controller 层的异常处理
 */
@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 处理所有不可知的异常
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    @ResponseBody
    AppResponse handleException(Exception e){
        LOGGER.error(e.getMessage(), e);

        AppResponse response = new AppResponse();
        response.setFail("操作失败！");
        return response;
    }

    /**
     * 处理所有业务异常
     * @param e
     * @return
     */
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    AppResponse handleBusinessException(BusinessException e){
        LOGGER.error(e.getMessage(), e);

        AppResponse response = new AppResponse();
        response.setFail(e.getMessage());
        return response;
    }

    /**
     * 处理所有接口数据验证异常
     * @param e
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    AppResponse handleMethodArgumentNotValidException(MethodArgumentNotValidException e){
        LOGGER.error(e.getMessage(), e);

        AppResponse response = new AppResponse();
                response.setFail(e.getBindingResult().getAllErrors().get(0).getDefaultMessage());
        return response;
    }
}
```

注意到了吗，所有的 Controller 层的异常的日志记录，都是在这个 GlobalExceptionHandler 中进行记录。也就是说，Controller 层也不需要在手动记录错误日志了。

 