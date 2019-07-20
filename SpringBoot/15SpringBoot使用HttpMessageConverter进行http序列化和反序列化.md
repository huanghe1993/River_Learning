# SpringBoot使用HttpMessageConverter进行http序列化和反序列化

对象的序列化/反序列化大家应该都比较熟悉：序列化就是将object转化为可以传输的二进制，反序列化就是将二进制转化为程序内部的对象。序列化/反序列化主要体现在程序I/O这个过程中，包括网络I/O和磁盘I/O。

那么什么是http序列化和反序列化呢？

在使用springmvc时，我们经常会这样写：

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public User getUserById(@PathVariable long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        System.err.println("create an user: " + user);
        return user;
    }
}
```

@RestController中有@ResponseBody，可以帮我们把User序列化到resp.body中。@RequestBody可以帮我们把req.body的内容转化为User对象。如果是开发Web应用，一般这两个注解对应的就是Json序列化和反序列化的操作。这里实际上已经体现了Http序列化/反序列化这个过程，只不过和普通的对象序列化有些不一样，Http序列化/反序列化的层次更高，属于一种Object2Object之间的转换。

Http协议的处理过程，TCP字节流 <---> HttpRequest/HttpResponse <---> 内部对象，就涉及这两种序列化。在springmvc中第一步已经由Servlet容器（tomcat等等）帮我们处理了，第二步则主要由框架帮我们处理。上面所说的Http序列化/反序列化就是指的这第二个步骤，它是controller层框架的核心功能之一，有了这个功能，就能大大减少代码量，让controller的逻辑更简洁清晰，就像上面示意的代码那样，方法中只有一行代码。

spirngmvc进行第二步操作，也就是Http序列化和反序列化的核心是HttpMessageConverter。用过老版本springmvc的可能有些印象，那时候需要在xml配置文件中注入MappingJackson2HttpMessageConverter这个类型的bean，告诉springmvc我们需要进行Json格式的转换，它就是HttpMessageConverter的一种实现。

![img](https://segmentfault.com/img/remote/1460000012658294?w=557&h=143)'



在Web开发中我们经常使用Json相关框架来进行第二步操作，这是因为Web应用中主要开发语言是js，对Json支持非常好。但是Json也有很大的缺点，大多数Json框架对循环引用支持不够好，并且Json报文体积通常比较大，相比一些二进制序列化更耗费流量。很多移动应用也使用Http进行通信，因为这是在手机app中，Json格式报文并没有什么特别的优势。这种情况下我们可能会需要一些性能更好，体积更小的序列化框架，比如Protobuf等等。

当前的SpringMVC 4.3版本已经集成了Protobuf的Converter，org.springframework.http.converter.protobuf.ProtobufHttpMessageConverter，使用这个类可以进行Protobuf中的Message类和http报文之间的转换。使用方式很简单，先依赖Protobuf相关的jar，代码中直接@Bean就行，像下面这样，springboot会自动注入并添加这种Converter。

```java
@Bean
    public ProtobufHttpMessageConverter protobufHttpMessageConverter() {
        return new ProtobufHttpMessageConverter();
    }
```

这里就不演示protobuf相关的内容了。

首先需要了解一些HTTP的基本知识（不是强制的而是一种建议与约定）：

1、决定resp.body的Content-Type的第一要素是对应的req.headers.Accept属性的值，又叫做MediaType。如果服务端支持这个Accept，那么应该按照这个Accept来确定返回resp.body对应的格式，同时把**resp.headers.Content-Type**设置成自己支持的符合那个Accept的MediaType。服务端不支持Accept指定的任何MediaType时，应该返回错误**406 Not Acceptable**.
例如：req.headers.Accept = text/html，服务端支持的话应该让resp.headers.Content-Type = text/html，并且resp.body按照html格式返回。
例如：req.headers.Accept = text/asdfg，服务端不支持这种MediaType，应该返回**406 Not Acceptable**。

2、如果Accept指定了多个MediaType，并且服务端也支持多个MediaType，那么Accept应该同时指定各个MediaType的**QualityValue**，也就是q值，服务端根据q值的大小来决定这几个MediaType类型的优先级，一般是大的优先。q值不指定时，默认视为q=1.
Chrome的默认请求的Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8，表示服务端在支持的情况下应该优先返回text/html，其次是application/xhtml+xml.
前面几个都不支持时，服务器可以自行处理 */*，返回一种服务器自己支持的格式。

3、一个HTTP请求没有指定Accept，默认视为指定 Accept: */*；没有指定Content-Type，默认视为 null，就是没有。当然，服务端可以根据自己的需要改变默认值。

4、Content-Type必须是具体确定的类型，不能包含 *.

SpringMvc基本遵循上面这几点。

然后是启动时默认加载的Converter。在mvc启动时默认会加载下面的几种HttpMessageConverter，相关代码在 **org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport中的addDefaultHttpMessageConverters**方法中，代码如下。

```java
 protected final void addDefaultHttpMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
        StringHttpMessageConverter stringConverter = new StringHttpMessageConverter();
        stringConverter.setWriteAcceptCharset(false);

        messageConverters.add(new ByteArrayHttpMessageConverter());
        messageConverters.add(stringConverter);
        messageConverters.add(new ResourceHttpMessageConverter());
        messageConverters.add(new SourceHttpMessageConverter<Source>());
        messageConverters.add(new AllEncompassingFormHttpMessageConverter());

        if (romePresent) {
            messageConverters.add(new AtomFeedHttpMessageConverter());
            messageConverters.add(new RssChannelHttpMessageConverter());
        }

        if (jackson2XmlPresent) {
            messageConverters.add(new MappingJackson2XmlHttpMessageConverter(
                    Jackson2ObjectMapperBuilder.xml().applicationContext(this.applicationContext).build()));
        }
        else if (jaxb2Present) {
            messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
        }

        if (jackson2Present) {
            messageConverters.add(new MappingJackson2HttpMessageConverter(
                    Jackson2ObjectMapperBuilder.json().applicationContext(this.applicationContext).build()));
        }
        else if (gsonPresent) {
            messageConverters.add(new GsonHttpMessageConverter());
        }
    }
```

这段代码后面还有两个别的处理，一次是将Jaxb放在list最后面，第二次是将一个StringConverter和一个JacksonConverter添加到list中，所以打印出converter信息中这两个有重复的（第二次的那两个来自springboot-autoconfigure.web，重复了不影响后面的流程）。

接着我们在自己的MVC配置类覆盖extendMessageConverters方法，使用converter.add(xxx)加上上次自定义Java序列化的那个的和FastJson的（把自己添加的放在优先级低的位置）。最后的converters按顺序展示如下（下面的已经去掉重复的StringHttpMessageConverter和MappingJackson2HttpMessageConverter，后续的相应MediaType也去重）

| 类名                                    | 支持的JavaType  | 支持的MediaType                                        |
| --------------------------------------- | --------------- | ------------------------------------------------------ |
| ByteArrayHttpMessageConverter           | byte[]          | application/octet-stream, */*                          |
| StringHttpMessageConverter              | String          | text/plain, */*                                        |
| ResourceHttpMessageConverter            | Resource        | */*                                                    |
| SourceHttpMessageConverter              | Source          | application/xml, text/xml, application/*+xml           |
| AllEncompassingFormHttpMessageConverter | Map<K, List<?>> | application/x-www-form-urlencoded, multipart/form-data |
| MappingJackson2HttpMessageConverter     | Object          | application/json, application/*+json                   |
| Jaxb2RootElementHttpMessageConverter    | Object          | application/xml, text/xml, application/*+xml           |
| JavaSerializationConverter              | Serializable    | x-java-serialization;charset=UTF-8                     |
| FastJsonHttpMessageConverter            | Object          | */*                                                    |

这里只列出重要的两个属性，详细的可以去看**org.springframework.http.converter**包中的代码。