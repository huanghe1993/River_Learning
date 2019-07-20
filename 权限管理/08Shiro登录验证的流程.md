## Shiro登录验证流程

　　很多时候，开发的项目不仅仅是一个基于浏览器的项目，还可能是基于app的项目，基于小程序的项目，而这些项目都是无状态的。而普通web项目中，一个web项目的会话是由session保持的，而session又是由浏览器携带的cookie来验证身份的，可以这么说，一个会话就是依赖于cookie，但是app与小程序是没有cookie维持的。

　　在一些环境中，可能需要把Web应用做成无状态的，即服务器端无状态，就是说服务器端不会存储像会话这种东西，而是每次请求时带上相应的用户名进行登录。如一些REST风格的API，如果不使用OAuth2协议，就可以使用如REST+HMAC认证进行访问。HMAC（Hash-based Message Authentication Code）：基于散列的消息认证码，使用一个密钥和一个消息作为输入，生成它们的消息摘要

> 消息摘要（Message Digest）又称作数字摘要(Digital Digest)。将长度为不固定的消息（message）作为输入参数，运行特定的Hash函数，生成固定长度的输出，这个输出就是Hash，也称为这个消息的消息摘要。**加密的消息摘要HMAC**，如果有两个输入参数，一个是原始的参数，另外一个是密钥（key），将会生成一个加密的消息摘要HMAC。那么地三方即使篡改数据，由于没有密钥，很难生成一个全新的加密摘要而不被终点主机发现而丢弃

注意该密钥只有客户端和服务端知道，其他第三方是不知道的。访问时使用该消息摘要进行传播，服务端然后对该消息摘要进行验证。如果只传递用户名+密码的消息摘要，一旦被别人捕获可能会重复使用该摘要进行认证。解决办法如：

网上也有很多SpringBoot集成shiro的实例 但是都不太完整，且不是stateless无状态的，不适用于现在这种前后端分离格局。这里特此记录下辛酸的集成过程，让大家少走一点弯路。shiro的集成方式为无状态(禁用session)，通过每次请求带上token进行鉴权。

1、每次客户端申请一个Token，然后使用该Token进行加密，而该Token是一次性的，即只能用一次；有点类似于OAuth2的Token机制，但是简单些；

2、客户端每次生成一个唯一的Token，然后使用该Token加密，这样服务器端记录下这些Token，如果之前用过就认为是非法请求。

## 实现登录验证

### 核心类

1. 需要ShiroConfiguration：在这个类中主要是注入shiro的filterFactoryBean和securityManager等对象。
2. 需要StatelessAccessControlFilter：这个类中实现访问控制过滤，当我们访问url的时候，这个类中的两个方法会进行拦截处理。
3. 需要StatelessAuthorizingRealm：这个类中主要是身份认证，验证信息是否合理，是否有角色和权限信息。
4. 需要StatelessAuthenticationToken：在shiro中有一个我们常用的UsernamePasswordToken，因为我们需要这里需要自定义一些属性值，比如：消息摘要，参数Map。
5. 需要StatelessDefaultSubjectFactory：由于我们编写的是无状态的，每人情况是会创建session对象的，那么我们需要修改createSubject关闭session的创建。
6. 需要HmacSHA256Utils：Java 加密解密之消息摘要算法，对我们的参数信息进行处理。

　　shiro中的权限操作是委托给securityManager的，而securityManager管理session又是委托sessionManager的，在开发web项目中，我们一般会使用`DefaultWebSecurityManager`来创建securityManager，我们看一下这个DefaultWebSecurityManager默认是使用的哪个session管理器，它的构造方法如下:

```java
public DefaultWebSecurityManager() {
        super();
        ((DefaultSubjectDAO) this.subjectDAO).setSessionStorageEvaluator(new DefaultWebSessionStorageEvaluator());
        this.sessionMode = HTTP_SESSION_MODE;
        setSubjectFactory(new DefaultWebSubjectFactory());
        setRememberMeManager(new CookieRememberMeManager());
        setSessionManager(new ServletContainerSessionManager());
    }
```

可以看到，如果构造一个DefaultWebSecurityManager，它使用的是`ServletContainerSessionManager`它是依赖于浏览器的cookie来维护session的，那肯定不能实现无状态的会话。

### 配置文件ShiroConfig

其中ShiroFilterFactoryBean和DefaultWebSecurityManager这两项是在Spring中引入Shiro的基本配置，前者用于配置过滤器的过滤规则，后者用于生成全局的securitymanager。该类将核心类都注册到IoC容器中

```java
@Configuration
public class ShiroConfig 
{ 
    /** 
     * ShiroFilterFactoryBean 处理拦截资源文件问题。 
     * 注意：单独一个ShiroFilterFactoryBean配置是或报错的，以为在 
     * 初始化ShiroFilterFactoryBean的时候需要注入：SecurityManager 
     * 
        Filter Chain定义说明 
       1、一个URL可以配置多个Filter，使用逗号分隔 
       2、当设置多个过滤器时，全部验证通过，才视为通过 
       3、部分过滤器可指定参数，如perms，roles 
     * 
     */  
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager)
    {
       logger.info("ShiroConfiguration.shirFilter()");
       ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //1、配置securityManager
    	factoryBean.setSecurityManager(securityManager);
        
        // 2、如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 3、登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 4、未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        
        //5、自定义过滤器的配置
        Map<String, Filter> filtersMap = new LinkedHashMap<String, Filter>();
        //限制同一帐号同时在线的个数。
        filtersMap.put("kickout", kickoutSessionControlFilter());
        filtersMap.put("statelessAuthc", new StatelessAccessControlFilter())
        shiroFilterFactoryBean.setFilters(filtersMap);
        System.out.println("Shiro拦截器工厂类注入成功");
        
        //6、配置过滤器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        //<!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/druid", "anon");
        filterChainDefinitionMap.put("/services/**", "statelessAuthc");
        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        
      
        
    	return factoryBean;
    }
```

在传统JSP web应用中，如果用shiro处理授权的话，通常通过`FormAuthenticationFilter`来直接截取用户提交的表单，并从中取出`username`、`password`、`rememberMe`三个参数来进行登录验证（截取的参数名可以在配置文件中手动修改）。
但是这并不适用于RESTful风格的应用，因为前端的用户验证信息是通过一个http的请求头请求发送过来的，没办法通过`FormAuthenticationFilter`进行处理，所以我们需要实现自己的filter。

# 无状态Filter的实现

此处为了正常处理登录验证和授权的功能，我们需要继承类`HttpMethodPermissionFilter`并实现两个方法

```java
public class StatelessAccessControlFilter extends AccessControlFilter {
	private static final Logger logger = LoggerFactory.getLogger(StatelessAccessControlFilter.class);

	@Override
	protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
		String requestURL = getPathWithinApplication(request);
        if(requestURL.endsWith("/")){
            return  true;
        }
		return false;
	}

	/**
	 * onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回true表示需要继续处理；
	 * 如果返回false表示该拦截器实例已经处理了，将直接返回即可。
	 */
	@Override
	protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
		// 1、客户端生成的消息摘要
		String clientDigest = request.getParameter("digest");
		String tokenId = request.getParameter("tokenId");
		// 2、客户端传入的用户身份
		String username = request.getParameter("userName");
		String userId = "";
		String password = request.getParameter("password");
		

		String requestURL = getPathWithinApplication(request);// 获取url
		Boolean isLogin = false;
		// 3、客户端请求的参数列表
		Map<String, String[]> params = new HashMap<String, String[]>(request.getParameterMap());
		params.remove("digest");// 为什么要移除呢？签名或者消息摘要算法的时候不能包含digest.

		logger.info("StatelessAuthenticationToken(username= " + username + "  clientDigest= " + clientDigest);
		// 4、生成无状态Token
		// 用户名，参数，客户端需验证的密码
		try {
			StatelessAuthenticationToken token = new StatelessAuthenticationToken(username, params, clientDigest);
			// 4、如当前URL匹配拦截器名字（URL模式）
			if (requestURL.endsWith("login")) {// 如为登陆，就只对密码进行加密
				//加密后进行匹配
		        Md5Hash md5Hash=new Md5Hash(password);
		        password= md5Hash.toString();
				AssertUtil.notEmpty(username, "用户名不能为空.");
				token.setClientDigest(password);
				token.setLogin(true);
			} else {
				AssertUtil.notEmpty(tokenId, "请先登陆获取授权.");
				userId=StringUtils.split(tokenId, "==")[1];
				AssertUtil.notEmpty(userId, "用户ID不能为空.");
				token.setTokenId(tokenId);
				token.setUserName(userId);
				token.setClientDigest(tokenId);
				token.setLogin(false);
				
				String key=Constants.CACHE_TOKNID+userId;
				String value= StringRedisUtil.get(key);
				if(value!=null){
					StringRedisUtil.setHours(key,value, Constants.CACHE_TOKENID_TIME);//设置Token
				}
			}
			Subject subject = getSubject(request, response);
			subject.login(token);
		}catch (MyException ex){
			onLoginFail(response, ex.getMessage(), 5);
           return false;
       } catch (UnauthorizedException e){
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            onLoginFail(response, e.getMessage(), httpResponse.SC_UNAUTHORIZED);
            return false;
        }catch (IncorrectCredentialsException e){
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            onLoginFail(response, "密码错误", httpResponse.SC_UNAUTHORIZED);
        }catch (Exception e) {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            if(e.getCause() != null ){
            	onLoginFail(response,e.getCause().getMessage(), Constants.SHIRO_ERROR);
            }else{
            	onLoginFail(response, e.getMessage(), httpResponse.SC_UNAUTHORIZED);
            }
           e.printStackTrace();
           return false;
       }
		return true;
	}
	
	/**
	 * 登录失败时默认返回401状态码  
	 * @param response
	 * @param message
	 * @param code
	 */
	private void onLoginFail(ServletResponse response, String message, int code){
	       HttpServletResponse httpResponse = (HttpServletResponse) response;
	       BaseResponse<String> responseMessage = new BaseResponse<>();
	       responseMessage.setCode(code);//错误码
	       responseMessage.setError(message);
	       WebUtil.responseWrit(httpResponse, responseMessage);
	    }
	

}
```

