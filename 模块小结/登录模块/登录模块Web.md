# 登录模块Web

[TOC]

**前言**

　　登录模块是我们在开发应用中基本都是会开发的一个模块，这个模块的开发有简单也有比较复杂，所以在这里准备好好的总结一下各种情况下的登录，以便以后的开发中可以进行快速的复用。在现代的应用中，后端往往不直接的和浏览器进行交互，如果后端是APP，一般是不会在请求中待JSESSION ID的，因为APP端没有和PC端一样的session机制；如果是PC端，对于单一的应用而言可以使用简答的session机制进行校验，但是在前后端分离的架构中，浏览器把请求发送给Web服务器，由Web服务器发送请求到后端服务器，此时也是不带JSESSION ID的。下面先从APP和Web的简单登录着手，到复杂的SSO。

-----

![img](https://upload-images.jianshu.io/upload_images/6073535-d776f7d5cf307675.png?imageMogr2/auto-orient/)

## 1. Web端Session的登录机制

首先想一个问题，我们为什么需要使用cookie、Session？

> 其实到了后面才反过来想这个问题，当时把自己给迷糊了。当时自己想法是这样的：我们登陆之后把用户信息放在session中，这个sesson也就是获取用户的信息的，说的再简单一点就是获取当前登录用户的id，然后通过当前用户的id进行一系列的操作。那么我们为什么不在进行一系列操作的时候手动传用户的id呢？这样也就用不到这个session了。

反过头来又看了一下cookie和session的出现，以及它们的作用

> 由于 Http 协议的请求过程，是基于 TCP/IP 的，当客户端请求服务器，服务器处理后，进行响应，这个过程是无状态的。通俗的解释就是，当用户在发送一个请求得到返回信息之后，客户端与服务器端之间的网络连接就已经断开了，在下一个请求发送时，服务器无法确定这次请求和上次的请求是否来自同一个客户端。也就是说，服务器不能记住"记住"用户，这是http协议的限制。

为什么不用手动传值呢？

> 当时的假设是在用户登录后，然后把用户的id保存起来，在需要使用用户id的时候再把这个id传给后台。哈哈哈这样想其实是对的，也就是session,cookie的思路，就是把用户的状态信息保存起来，当然用户的id也是用户的状态信息

　　早期互联网以 web 为主，客户端是浏览器，所以 Cookie-Session 方式最那时候最常用的方式，直到现在，一些 web 网站依然用这种方式做认证。具体的实现方式如下所示：

　　当浏览器 第一次发送请求时，服务器自动创建了一个Session对象，**并将通过特殊算法算出一个session的ID**，用来标识该session对象，并将其通过响应发送到浏览器。当浏览器第二次发送请求，会将前一次服务器响应中的Session ID放在cookie中(cookie里面的值存放的是Session ID)一并发送到服务器上，服务器从请求中提取出Session ID，并和保存的所有Session ID进行对比，找到这个用户对应的Session。

　　一般情况下，服务器会在一定时间内<font color='red'>（默认30分钟）</font>保存这个 Session，过了时间限制，就会销毁这个Session。在销毁之前，程序员可以将用户的一些数据以Key和Value的形式暂时存放在这个 Session中。当然，也有使用数据库将这个Session序列化后保存起来的，这样的好处是没了时间的限制，坏处是随着时间的增加，这个数据 库会急速膨胀，特别是访问量增加的时候。一般还是采取前一种方式，以减轻服务器压力。

**首先了解一下session的特点：**

- 由于 `Session` 是以文本文件形式存储在服务器端的，所以不怕客户端修改 Session 内容。
- `Session`对象是有生命周期的
- `Session`实例是轻量级的，所谓轻量级：是指他的创建和删除不需要消耗太多资源

**认证过程大致如下：**

1. 用户输入用户名、密码或者用短信验证码方式登录系统；
2. 服务端验证后，创建一个 Session 信息，并且将 SessionID 存到 cookie，发送回浏览器；
3. 下次客户端再发起请求，自动带上 cookie 信息，服务端通过 cookie 获取 Session 信息进行校验；

**弊端**

- 只能在 web 场景下使用，如果是 APP 中，不能使用 cookie 的情况下就不能用了；
- 即使能在 web 场景下使用，也要考虑跨域问题，因为 cookie 不能跨域；
- cookie 存在 CSRF（跨站请求伪造）的风险；
- 如果是分布式服务，需要考虑 Session 同步问题；

**具体的代码如下所示**：

　　基本上在做登录的时候都是会传用户名和密码的，有的用户名是使用的邮箱，这个情况是在Web的登录中比较多，也有的是使用电话号只作为用户名，有的系统的登录名是在上传的时候用户上传的，有的登录名是之后进行用户的资料进行修改的时候才添加上去的，所以这个最简单的方式是使用邮箱和密码或者是手机号和密码这两种方式登录，现在以这两种方式为例子来说明：

先从Controller的入口进行说明：

```java
	@RequestMapping("/loginSubmit")
    @ResponseBody
    public JsonResult<?> login (@RequestParam("email") String email,
            @RequestParam("password") String password, 
            @RequestParam(value = "gourl", required = false, defaultValue = "") String gourl,
            HttpServletRequest request, HttpServletResponse response) {
        
        if (email == null || "".equals(email.trim())) {
            return JsonResult.error("请输入用户名");
        }
        
        if (password == null || "".equals(password.trim())) {
            return JsonResult.error("请输入密码");
        }
        
        User user = userService.getActiveUserByEmail(email);
        
        if (user == null) {
            return JsonResult.error("用户名不存在");
        }
        
        if (!EncryptHelper.encryptMD5(password).equals(user.getPassword())) {
            return JsonResult.error("密码输入不正确");
        }
        
        request.getSession().setAttribute(SessonConstants.CURRENT_LOGIN_USER, user);
        
        super.addCookie(request, response, CookieConstants.LOGIN_USER_ID, String.valueOf(user.getId()), Integer.MAX_VALUE);
        
        super.addCookie(request, response, CookieConstants.LOGIN_EMAIL, user.getEmail(), Integer.MAX_VALUE);
        
        super.addCookie(request, response, CookieConstants.LOGIN_STATUS, String.valueOf(1), -1);
        
        logger.info("--------------------------------------------user: "+email+" , pwd: "+password+" login successfully!");
        
        return JsonResult.success(user);
```

说明：这个Controller继承了BaseController，这个BaseController里面的方法是protetced的，也即是就在这个包内是有效的

```java
/**
     * 设置cookie
     * 
     * @param response
     * @param name cookie名字
     * @param value cookie值
     * @param maxAge acookie生命周期 以秒为单位 (-1浏览器关闭立即失效)
     */
    protected void addCookie(HttpServletRequest request, HttpServletResponse response, String name, String value,
            int maxAge) {
        Cookie cookie = new Cookie(name, value);
        cookie.setPath("/");
        cookie.setMaxAge(maxAge);
        response.addCookie(cookie);
    }

	protected User getCurrentLoginUser(HttpServletRequest request){
        User user = (User)               request.getSession().getAttribute(SessionConstant.CURRENT_LOGIN_USER);
        return user;
    }


```

**认证过程：**

> 在代码的第26行可以看到把当前的用户保存在session中，session是以Key和Value的方式保存起来的，其中Key是SessonConstants.CURRENT_LOGIN_USER，Value是当前的获取到的用户。在28,30,32行分别把用户ID，用户登录邮箱，登录状态设置到Cookie中。

## 2. Web端Session获取用户状态信息

用户登录完成之后，会把用户的信息存储在session中，之后再进行session的调用的时候就会session中获取数据，举个简单的例子**用户登录之后需要显示用户的信息和修改信息**：

这个时候获取用户的信息都是从session中进行获取的

```java
 	@RequestMapping(value = "/show")
    @ResponseBody
    public JsonResult<?> showPersonInfo(HttpServletRequest request,@RequestParam("id") Integer id) {
        //这里的获取用户的信息是从session中获取的，getCurrentLoginUser也是在BaseController
        User currentLoginUser = getCurrentLoginUser(request);
        currentLoginUser = userService.getByPrimaryKey(currentLoginUser.getId());
        return JsonResult.success(currentLoginUser);
    }

    @RequestMapping("/updateSave")
    @ResponseBody
    public JsonResult<?> saveData(@RequestParam("nameCn") String nameCn,
                                  @RequestParam("nameEn") String nameEn,
                                  @RequestParam("gender") Integer gender,
                                  @RequestParam("homeAddress") String homeAddress,
                                  HttpServletRequest request, Model model) {
		//这里的获取用户的信息是从session中获取的，getCurrentLoginUser也是在BaseController
        User user = getCurrentLoginUser(request);

        user = userService.getByPrimaryKey(user.getId());
        user.setNameCn(nameCn);
        user.setNameEn(nameEn);
        user.setGender(gender);
        user.setHomeaddress(homeAddress);
        userService.insertOrUpdate(user);
        
		//需要在这一步更新session
        request.getSession().setAttribute(SessonConstants.CURRENT_LOGIN_USER, user);

        return JsonResult.success(user);
    }
```

## 3. 登录拦截认证的过程

在做开发时，遇到 防止未登录访问时需要自定义拦截器。

这过程使用的是拦截器，这里工具类如下所示：

```java
public class Cookie2SessionInterceptor implements HandlerInterceptor {
    
    private Logger logger = LoggerFactory.getLogger(getClass());

    //方法在请求处理之前进行调用
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        
        HttpSession session = request.getSession();
		
        //从session中获取当前登录的用户
        User user = (User) session.getAttribute(SessionConstant.CURRENT_LOGIN_USER);
        
        //如果当前用户为空
        if (user == null) {
            try { 
                String userId = null;
                String email = null;
                String loginStatus = "0";
				
                //获取到Cookie里面所有的值,map的key是cookie的name
                Map<String, Cookie> cookies = readCookieMap(request);
				
                //获取当前登录用户的Cookie
                Cookie uidCookie = cookies.get(CookieConstants.LOGIN_USER_ID);

                if (uidCookie != null) {
                    userId = uidCookie.getValue();
                }

                Cookie emailCookie = cookies.get(CookieConstants.LOGIN_EMAIL);
                if (emailCookie != null) {
                    email = emailCookie.getValue();
                }

                Cookie loginStatusCookie =cookies.get(CookieConstants.LOGIN_STATUS);         		if (loginStatusCookie != null) {
                    loginStatus = loginStatusCookie.getValue();
                }
				
                //如果当前的用户在cookie中，但是用户在打开第二页面的时候并不需要登录就能访问
                //所以也是需要把当前的用户设置在session中的
                if (userId != null && email != null && "1".equals(loginStatus)) {
                    WebApplicationContext applicationContext = ContextLoader.getCurrentWebApplicationContext();
                    UserService userService = applicationContext.getBean(UserService.class);
                    user = userService.getByPrimaryKey(Integer.valueOf(userId));
                    if (user != null) {
                        session.setAttribute(SessionConstant.CURRENT_LOGIN_USER, user);
                    }
                }
            } catch (Exception e) {
                logger.error(e.getMessage(), e.fillInStackTrace());
            }

        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        
    }
    
    protected Map<String, Cookie> readCookieMap(HttpServletRequest request) {
        Map<String, Cookie> cookieMap = new HashMap<String, Cookie>();
        Cookie[] cookies = request.getCookies();
        if (null != cookies) {
            for (Cookie cookie : cookies) {
                cookieMap.put(cookie.getName(), cookie);
            }
        }
        return cookieMap;
    }
}

```

自定义的拦截器需要配置，配置的代码如下所示：实际的开发中

```java
@Configuration
public class MyWebAppConfigurer 
        extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 多个拦截器组成一个拦截器链
        // addPathPatterns 用于添加拦截规则
        // excludePathPatterns 用户排除拦截
       registry.addInterceptor(newCookie2SessionInterceptor).addPathPatterns("/**");
       super.addInterceptors(registry);
    }
}
```

## 4. 登录忘记密码邮箱验证

话术上Google使用的是“无法访问您的账户吗?”，腾讯使用的是“找回密码”，其他均为“忘记密码?”。个人认为Google的说法更为准确，QQ的“找回密码”操作其实是“重设密码”的过程，并非找回原来的密码。其他的“忘记密码”也不算准确，毕竟账户被盗的事情屡屡发生。“无法访问”更像是现状的描述

常采用的方式：用户名or邮箱or手机号+验证码。验证码是为了防止机器刷，进行批量操作或破解

这里结合目前做的项目，详细的说明一下密码找回的过程。这里用的是邮箱找回。根据一般的忘记密码的流程来说明。我采用的流程是：

- 登录页面

【登录】 --> 【点击忘记密码】 --> 【输入个人邮箱和验证码】 --> 【系统发送邮箱验证】 --> 【用户在限定时间内重置密码】 --> 【重置密码完毕，点击进入登录界面】。

![mark](http://www.hhaxmm.cn/blog/20190111/k7jPJta7fIxk.png)

一般登录界面都有“忘记密码”选项，这里不多说。

### 4.1. "忘记密码"页面（邮箱找回）

点击"忘记密码"选项后，这时，页面会跳向一个新页面，即“忘记密码”页面

![mark](http://www.hhaxmm.cn/blog/20190111/qewIqUl2O2nm.png)

这里的界面设计是模仿的网页163邮箱的忘记密码，给出链接：http://reg.163.com/getpasswd/RetakePassword.jsp?from=mail163。该界面中，有几个关键点：1、是验证码的生成，2、是邮箱发送的功能，3、就是点击确定后的反馈界面。主要包含这3个方面的内容。验证码的生成和校验在前端进行完成。

- 验证码

 在涉及到前后端交互的系统或者实际项目中，常常会遇到登录信息的校验处理。而出于安全性考虑，除了对所输入的明文密码进行加密外，其中验证码也是很必要的一项。而从验证码的校验上来说，可分为前端校验和后端校验。关于验证码的校验在前端完成还是在后端完成，这里我也是查询了一些资料后得到的总结。

- 前端校验

一是前端校验。向服务器端传送数据时，不需再有验证码这个字段，所有的生成、校验和刷新处理都由前端完成。这样的校验方式，之前做的nais的网站就是采用的这样的校验方式。但是这样安全性不高，并且在前端校验的意义不大，所以之前的网站的校验方式不是特别合理的方式。

- 后端校验

如果是在后端校验，那么验证码的生成、刷新以及校验工作全由后端人员完成，前端只需要将用户所输入的验证码传送到后端即可。至于到底用哪个方式更好，还是要根据实际需求来进行选择。**如果是比较简单且不涉及重要个人信息处理的平台，可以选择在前端完成。**

#### 4.1.1 验证码

这里主要是使用后端生成和校验验证码[springboot整合kaptcha验证码](https://www.jianshu.com/p/1f2f7c47e812)，这儿主要使用配置类的方式来配置

- **添加kaptcha依赖**

```xml
<!-- kaptcha验证码 -->
        <dependency>
        <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
        </dependency>
```

- **新建KaptchaConfig.java，内容如下:**

```java
@Configuration
public class KaptchaConfig {
    @Bean
    public DefaultKaptcha getDefaultKaptcha(){
        DefaultKaptcha captchaProducer = new DefaultKaptcha();
        Properties properties = new Properties();
        properties.setProperty("kaptcha.border", "yes");
        properties.setProperty("kaptcha.border.color", "105,179,90");
        properties.setProperty("kaptcha.textproducer.font.color", "blue");
        properties.setProperty("kaptcha.image.width", "110");
        properties.setProperty("kaptcha.image.height", "40");
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        properties.setProperty("kaptcha.session.key", "code");
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
        Config config = new Config(properties);
        captchaProducer.setConfig(config);
        return captchaProducer;

    }
}
```

**注:**这个类用来配置Kaptcha，就相当于方式一的kaptcha.xml，把kaptcha加入IOC容器，然后return 回一个设置好属性的实例，最后注入到CodeController中，在CodeController中就可以使用它生成验证码。要特别注意`return captchaProducer;`与`private Producer captchaProducer = null;`中`captchaProducer`名字要一样，不然就加载不到这个bean。

- **编写controller用于生成验证码:CodeController.java**

```java
@Controller
public class CodeController {
    @Autowired
    private Producer captchaProducer = null;
    @RequestMapping("/kaptcha")
    public void getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpSession session = request.getSession();
        response.setDateHeader("Expires", 0);
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
        response.setHeader("Pragma", "no-cache");
        response.setContentType("image/jpeg");
        //生成验证码,存放在session中了
        String capText = captchaProducer.createText();
        session.setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);
        //向客户端写出
        BufferedImage bi = captchaProducer.createImage(capText);
        ServletOutputStream out = response.getOutputStream();
        ImageIO.write(bi, "jpg", out);
        try {
            out.flush();
        } finally {
            out.close();
        }
    }
}
```

**注:**在这个controller径注入刚刚kaptcha.xml中配置的那个bean，然后就可以使用它生成验证码，以及向客户端输出验证码;记住这个类的路由，前端页面验证码的src需要指向这个路由。生成的验证码存放在session中（15行）

- **新建验证码比对工具类:CodeUtil.java**

```java
public class CodeUtil {
    /**
     * 将获取到的前端参数转为string类型
     * @param request
     * @param key
     * @return
     */
    public static String getString(HttpServletRequest request, String key) {
        try {
            String result = request.getParameter(key);
            if(result != null) {
                result = result.trim();
            }
            if("".equals(result)) {
                result = null;
            }
            return result;
        }catch(Exception e) {
            return null;
        }
    }
    /**
     * 验证码校验
     * @param request
     * @return
     */
    public static boolean checkVerifyCode(HttpServletRequest request) {
        //获取生成的验证码，从session中获取
        String verifyCodeExpected = (String) request.getSession().getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
        //获取用户输入的验证码
        String verifyCodeActual = CodeUtil.getString(request, "verifyCodeActual");
        if(verifyCodeActual == null ||!verifyCodeActual.equals(verifyCodeExpected)) {
            return false;
        }
        return true;
    }
}
```

**注:**这个类用来比对生成的验证码与用户输入的验证码。生成的验证码会自动加到**session**中，用户输入的通过**getParameter**获得。注意**getParameter**的**key**值要与页面中验证码的**name**值一致。

- **使用验证码:**

  ①Controller     HelloWorld.java，这个是示例代码，在实际中封装在请求参数,使用@RequestParam来获取参数，传给我的就是一个String

```java
@RestController
public class HelloWorld {
    @RequestMapping("/hello")
    public String hello(HttpServletRequest request) {
        if (!CodeUtil.checkVerifyCode(request)) {
            return "验证码有误！";
        } else {
            return "hello,world";
        }
    }
}

```

 ②页面     hello.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        function refresh() {
            document.getElementById('captcha_img').src="/kaptcha?"+Math.random();
        }
    </script>
</head>
<body>
<form action="/hello" method="post">
    验证码:  <input type="text" placeholder="请输入验证码" name="verifyCodeActual">
    <div class="item-input">
        <!--生成验证码，这个路由指向的就是生成验证码CodeController的代码 -->
        <img id="captcha_img" alt="点击更换" title="点击更换"
             onclick="refresh()" src="/kaptcha" />
    </div>
    <input type="submit" value="提交" />
</form>

</body>
</html>
```

总结:

**1、过程梳理:**
把kaptcha加入spring容器，即要有一个kaptcha的bean;**再**新建一个controller，把kaptcha的bean注入到controller中用于生成验证码;**然后**需要有一个比对验证码的工具类，在测试的controller中调用工具类进行验证码比对;**最后**在前端页面只需要用一个`<img src = "/生成验证码的controller的路由">`即可获取验证码，通过给这个img标签加点击事件就可实现“点击切换验证码”。

**2、**与spring整合kaptcha对比
 springboot整合kaptcha先配置bean，然后用controller生成验证码，前端用src指向这个controller，不同之处在于:假如生成验证码的controller路由为`/xxx`，那么spring 整合kaptcha时`src = “xxx.jpg”`，springboot整合kaptcha时`src = "/xxx"`。

#### 4.1.2  邮箱验证

　　在输入完用户的邮箱（或者手机号）和验证码之后进行提交，这个时候前端判断提交的是手机号还是邮箱，如果是手机号则会进入到手机号的controller验证，如果是邮箱则会使用邮箱的验证功能。本次使用的是邮箱的验证，提交给后端验证的Controller，邮箱是否合理，验证码是否正确。这个Controller接受的参数有用户的邮箱和验证码，如果都是正确的则发送邮件到用户的邮箱 ，返回responseMessage的JSON格式，里面有是状态码（按照我自己的习惯此时的状态码应该设置为1）。

**验证用户的邮箱和验证码,并且发送邮件** ：Controller大概 如下所示（不是完整的代码）

```java
   @RequestMapping(value = "/InfoVerify", method = RequestMethod.POST)
   @ResponseBody
   public Object addFriend(@RequestParam(value = "emailOrPhone" String emailOrPhone,
                @RequestParam(value = "verifyCodeActual" String verifyCodeActual) {
      //后台也是可以进行判断的，大部分的时候是在前台进行判断了
      if(emailOrPhone.contains("@")){
          //检查是否符合邮箱的格式
          if (!checkEmail(emailOrPhone)){
            return JsonResult.error("请输入正确的邮箱格式");
        	}
      }else{
          if (!checkPhone(emailOrPhone)){
            return JsonResult.error("请输入正确的手机号格式");
        	}
      }
	  //根据手机号或者是邮件获取用户（SQL使用OR）
      User user = userService.getActiveUserByEmailOrPhone(email);

      if (user == null) {
            return JsonResult.error("请输入注册的邮箱");
        }
                    
      if(verifyCodeActual == null ||!verifyCodeActual.equals(verifyCodeExpected)){
     		 return JsonResult.error("验证码不正确");
     	}
        
        WebConfig host = webConfigService.getByKey(ConfigConstants.EMAIL_SERVER_HOST);

        WebConfig port = webConfigService.getByKey(ConfigConstants.EMAIL_SERVER_PORT);

        WebConfig userName = webConfigService.getByKey(ConfigConstants.EMAIL_USERNAME);

        WebConfig pwd = webConfigService.getByKey(ConfigConstants.EMAIL_PASSWORD);
        
		//邮件的内容：这里的邮件的内容是邮箱中的链接
        // 获取当前系统时间
		Date now = new Date();
		String currentTime = "" + now.getTime();
        //跳转的url的地址
        String urlString = "http://localhost:8080/user/verifySubmit？key=";
        //对用户名和时间进行加密处理
         CodeEncryption cEncryption = new CodeEncryption();
	String encryptedCode = cEncryption.encryptCode(currentTime + "@" + userEmail);
		String link = urlString + encryptedCode;

		//开启一个线程来执行发送邮件的任务
        taskExecutor.execute(new EmailVeryfyTask(email,link,host.getV(),port.getV(),userName.getV(),pwd.getV()));

        logger.info("-------------send email to user: "+email);
                    
        return JsonResult.success();                                
```

当邮件发送成功之后会跳转到如下的界面，这里735597346@qq.com就是从`JsonResult.success(user);`获取的，前天根据状态码进行跳转，这个页面的跳转是立即进行的，邮箱是使用单独的一个线程进行发送。

![mark](http://www.hhaxmm.cn/blog/20190111/PTsVPv7TVHBR.png)

用户在提交了验证码和邮箱之后，并且校验正确，服务器<font color='red'>**才**</font>会向用户发送修改密码的邮件，其内容大概如下所示：

![mark](http://www.hhaxmm.cn/blog/20190111/Xgy3EWKJonM7.png)

对于这个链接，还有一个重要的部分，那就是链接要链向哪里，需要哪些参数，这个链接怎么保证时效性，每个密码找回链接都是有时效性的，这是以慕课网为例子慕课网的连接请求的resetpassage，请求的参数是加密后的参数，点击链接后界面如下所示：

![mark](http://www.hhaxmm.cn/blog/20190111/3UBWowpSCdkI.png)

从以上的内容可以得知以下的信息：

> ①一部分是用户的用户名等可以辨识用户的内容，否则在用户点击链接后无法得知用户是谁；（如上面的图中显示的735597346@qq.com）
>
> ②还一部分是邮件发送时的时间，因为只有参数中含有时间信息，才能使得该链接具有时效性。

当用户点击链接之后会提交到后台Controller，作用是要检查用户点击链接的时间是否过期，以及获取到当前的用户，代码大概如下所示：

```java
@Controller
@RequestMapping("/user")
public class LoginController extends BaseController {
    
    @RequestMapping(value = "/verifySubmit")
    public JsonResult<?> passwordReset(@RequestParam("key") String key,
                                       HttpServletRequest request){
        ForgetPasswordService fService = new ForgetPasswordService();
		List<String> datasList = new ArrayList<String>();
        //解码
        datasList = fService.getExplicitCode(key);
        String time = datasList.get(0);
        //获取到用户名，下图中需要使用用户名
		String userEmail = datasList.get(1);
        //根据用户名获取user
        User user = userService.getUserByEmail("userEmail");
        //把用户信息写到session中
        request.getSession().setAttribute(SessonConstants.CURRENT_LOGIN_USER,user); 
        //获取是否过期，下图中过期和不过期的页面不一样，不过期的页面如下图所示
		Boolean timeFlag = fService.judgeOfTime(time);	//判断用户点击链接的时间是否过期
		String flag = "false";
		if (timeFlag) {
			flag = "true";
		}
        //这里使用的是Map的集合的形式
        HashMap<String,Object> map = new HashMap<>();
        map.put("userEmail",userEmail);
        map.put("flag",flag);
        
        
       //使用ModeAndView进行转发
       ModelAndView mv = new ModelAndView("forward:/user/passwordResetSubmit.jsp");
       mv.addObject("Objectbean",map)
		return mAndView;  
	}
```
前端获取到返回的数据，根据flag进行判断，如果flag为true则表示当前链接还是有效的，跳转到下面的界面，如果flag为false的话则说明这个链接已经过期了，跳转到超时的界面（可以在页面中加上超链接，然后跳转到重置密码的页面）。

![mark](http://www.hhaxmm.cn/blog/20190111/3UBWowpSCdkI.png)



最后就是提交密码，重置密码了；因为这个邮件在别的服务器中也是可以打开的，别人服务器打开也会显示账号的，所以此时的这个账号可能是从sessIon中获取到的，也可能是前端传过来的。如果是session获取的则需要在上面的过期时间校验的controller中把当前的用户加入到session中，然后在重置密码的页面上获取用户的信息。如果是前端页面传的email则直接的在请求的参数中获取。

```java
@RequestMapping(value = "/passwordResetSubmit")
@ResponseBody
public JsonResult<?> passwordReset(@RequestParam String password,
                                   @RequestParam("repwd") String repwd,
                                   @RequestParam("email") String email,
                                   HttpServletRequest request){
        if (!checkEmail(email)){
            return JsonResult.error("请输入正确的邮箱格式");
        }
        
        if (password == null || "".equals(password.trim())) {
            return JsonResult.error("请输入密码");
        }

        if (password.length()<6){
            return JsonResult.error("密码长度不得小于6位");
        }

        if (!isValidate(password)){
            return JsonResult.error("密码必须由数字和字母组成");
        }


        if (repwd == null || "".equals(repwd.trim())) {
            return JsonResult.error("请再次确认密码");
        }
        
        if (!password.equals(repwd.trim())) {
            return JsonResult.error("两次输入密码不一致");
        }
        User user = (User) request.getSession().getAttribute(SessonConstants.CURRENT_LOGIN_USER);      

    if (user!=null){
        user.setPassword(EncryptHelper.encryptMD5(password));
        userService.insertOrUpdate(user);
        logger.info("--------------------------------------------reset password: user: "+user.getEmail());
        return JsonResult.success(null,"密码重置成功");
    }else {
        return JsonResult.error("密码重置失败");
    }
}
```

### 4.1. "忘记密码"页面（手机号找回）







## 附加工具

加密工具

```java
/**
	 * 将明文字符串加密成密文，然后以字符串的形式返回.
	 * 说明：在将字符串明文加密后，得到的是一个byte[]数组，这时候如果将byte[]数组转化为字符串，是乱码。解决方式如下：
	 * 步骤1：对于byte[]数组中的每个元素，因为范围都在-128~127之间，所以为了方便表达，统一加上128，都转化为0或者正数；
	 * 步骤2： 利用Integer.toBinaryString()函数，将每个元素转化为16进制，结果若为单个的，用g在右侧补成2位的；
	 * 步骤3：将所有的结果统一串成一个整个的字符串，作为结果返回。
	 * @param explicitPassword
	 * @return
	 */
	public String encryptCode(String explicitString) {
		Security.addProvider(new com.sun.crypto.provider.SunJCE());
		
		String implicitString = "";
		byte[] encoded = encryptMode(keyBytes, explicitString.getBytes());
		
		List<String> codeArrTemp = new ArrayList<String>();
		for (int i = 0; i < encoded.length; i++) {
			codeArrTemp.add(String.valueOf(Integer.toHexString((int) encoded[i] + 128)));
		}
		List<String> codeArr = new ArrayList<String>();
		for (int i = 0; i < codeArrTemp.size(); i++) {
			if (codeArrTemp.get(i).length() == 1) {
				String temp = codeArrTemp.get(i) + "g";
				codeArr.add(temp);
			} else {
				codeArr.add(codeArrTemp.get(i));
			}
		}
		
		for (int i = 0; i < codeArr.size(); i++) {
			implicitString += codeArr.get(i);
		}
		
		return implicitString;
	}

```

与上面加密配套的解密代码：

```java

/**
	 * 对密文解密，返回明文
	 * @param implicitString
	 * @return
	 */
	public String deEncryptCode(String implicitString) {
		Security.addProvider(new com.sun.crypto.provider.SunJCE());
		
		String explicitString = null;
		byte[] encoded = null;
		//根据implicitString得到encoded
		List<String> codeArrTemp = new ArrayList<String>();
		for (int i = 0; i < implicitString.length(); i += 2) {
			codeArrTemp.add(implicitString.substring(i, i + 2));
		}
		List<String> codeArr = new ArrayList<String>();
		for (int i = 0; i < codeArrTemp.size(); i++) {
			if (codeArrTemp.get(i).contains("g")) {
				codeArr.add(codeArrTemp.get(i).substring(0, 1));
			} else {
				codeArr.add(codeArrTemp.get(i));
			}
		}
		encoded = new byte[codeArr.size()];
		for (int i = 0; i < codeArr.size(); i++) {
			encoded[i] = (byte) (Integer.parseInt(codeArr.get(i), 16) - 128);
		}
		
		byte[] srcBytes = decryptMode(keyBytes, encoded);
		explicitString = new String(srcBytes);
		return explicitString;
	}

```



