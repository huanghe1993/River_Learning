# Shiro 过滤器 ShiroFilterFactoryBean

在Spring中使用Shiro的话，可在web.xml中配置一个DelegatingFilterProxy，DelegatingFilterProxy作用是自动到Spring容器查找名字为shiroFilter（filter-name）的bean并把所有Filter的操作委托给它。

首先是在`web.xml`中配置`DelegatingFilterProxy`

```xml
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

配置好`DelegatingFilterProxy`后，下面只要再把`ShiroFilter`配置到`Spring`容器（此处为`Spring`的配置文件）即可：

```java
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
</bean>
```

可以看到我们使用了ShiroFilterFactoryBean来创建shiroFilter，这里用到了Spring中一种特殊的Bean——FactoryBean。当需要得到名为”shiroFilter“的bean时，会调用其getObject()来获取实例。下面我们通过分析ShiroFilterFactoryBean创建实例的过程来探究Shiro是如何实现安全拦截的：

```java
    /**
     * @return the application's Shiro Filter instance used to filter incoming web requests.
     */
    public Object getObject() throws Exception {
        if (instance == null) {
            instance = createInstance();
        }
        return instance;
    }
```

其中调用了`createInstance()`来创建实例：

```java
protected AbstractShiroFilter createInstance() throws Exception {

        // 这里是通过FactoryBean注入的SecurityManager（必须）
        SecurityManager securityManager = getSecurityManager();
        if (securityManager == null) {
            String msg = "SecurityManager property must be set.";
            throw new BeanInitializationException(msg);
        }
		// 和Spring整和的时候需要使用WebSecurityManager
        if (!(securityManager instanceof WebSecurityManager)) {
            String msg = "The security manager does not implement the WebSecurityManager interface.";
            throw new BeanInitializationException(msg);
        }
		
        FilterChainManager manager = createFilterChainManager();

        PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
        chainResolver.setFilterChainManager(manager);

        return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
    }

```

可以看到创建`SpringShiroFilter`时用到了两个组件：`SecurityManager`和`ChainResolver`。

- SecurityManager：我们知道其在Shiro中的地位，类似于一个“安全大管家”，相当于SpringMVC中的DispatcherServlet，所有具体的交互都通过SecurityManager进行控制，它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理。
- `ChainResolver：Filter`链解析器，用来解析出该次请求需要执行的Filter链
- `PathMatchingFilterChainResolver`：`ChainResolver`的实现类，其中还包含了两个重要组件`FilterChainManager`、`PatternMatcher`
- `FilterChainManager`：管理着Filter和Filter链，配合`PathMatchingFilterChainResolver`解析出Filter链
- `PatternMatcher`：用来进行请求路径匹配，默认为Ant风格的路径匹配

先有一个大体的了解，那么对于源码分析会有不少帮助。下面会对以上两个重要的组件进行分析，包括`PathMatchingFilterChainResolver`和`FilterChainManager`。首先贴一段`ShiroFilter`的在配置文件中的定义：

```xml
<!-- Shiro的Web过滤器 -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager" />
        <property name="loginUrl" value="/login" />
        <property name="unauthorizedUrl" value="/special/unauthorized" />
        <property name="filters">
            <util:map>
                <entry key="authc" value-ref="formAuthenticationFilter" />
                <entry key="logout" value-ref="logoutFilter" />
                <entry key="ssl" value-ref="sslFilter"></entry>
            </util:map>
        </property>
        <property name="filterChainDefinitions">
            <value>
                /resources/** = anon
                /plugin/** = anon
                /download/** = anon
                /special/unauthorized = anon
                /register = anon
                /login = ssl,authc
                /logout = logout
                /admin/** = roles[admin]

                /** = user
            </value>
        </property>
    </bean>
```



​                              