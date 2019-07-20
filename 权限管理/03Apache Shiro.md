# shiro

[TOC]



## 简介

Apache Shiro是Java的一个安全框架。目前，使用Apache Shiro的人越来越多，因为它相当简单，对比Spring Security，可能没有Spring Security做的功能强大，但是在实际工作时可能并不需要那么复杂的东西，所以使用小而简单的Shiro就足够了。对于它俩到底哪个好，这个不必纠结，能更简单的解决项目问题就好了。

Shiro可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用在JavaEE环境。Shiro可以帮助我们完成：认证、授权、加密、会话管理、与Web集成、缓存等。这不就是我们想要的嘛，而且Shiro的API也是非常简单；其基本功能点如下图所示：

## Shiro可以做什么

- 验证用户身份
- 用户访问控制，比如用户是否被赋予了某个角色；是否允许访问某些资源
- 在任何环境都可以使用Session API，即使不是WEB项目或没有EJB容器
- 事件响应(在身份验证，访问控制期间，或是session生命周期中)
- 集成多种用户信息数据源
- SSO-单点登陆
- Remember Me，记住我
- Shiro尝试在任何应用环境下实现这些功能，而不依赖其他框架、容器或应用服务器。

Shiro可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用在JavaEE环境。Shiro可以帮助我们完成：认证、授权、加密、会话管理、与Web集成、缓存等。这不就是我们想要的嘛，而且Shiro的API也是非常简单；其基本功能点如下图所示：

![mark](http://www.hhaxmm.cn/blog/20190114/K6QmlLtkBUUa.png)

**四大基石----身份验证，授权，会话管理，加密**

1.Authentication：身份认证/登录，验证用户是不是拥有相应的身份；

2.Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

3.Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

4.Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

## shiro整体的框架图

![mark](http://www.hhaxmm.cn/blog/20190114/aAC4TBQEkjJm.png)

Security Manager是Shiro的核心，shiro也是通过Security Manager来提供安全服务的，Security Manager管理其他组件的实例

- Subject：主体，可以看到主体可以是任何可以与应用交互的 “用户”；

- Authenticor ：认证器，管理我们的登录，退出登录；如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；
- Authorizer：授权器，授予我们主体有哪些权限
- Session Manager：session管理器，shiro自己实现了一套session的管理机制
- Session DAO：提供session操作的API，增删改查
- Cache Manager :换成管理器，使用Cache Manager来换成角色数据和权限数据
- Realm：shiro和数据库数据源之间的桥梁，shiro获取角色数据，权限数据都是通过Reaml来获取的；可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是 JDBC 实现，也可以是 LDAP 实现，或者内存实现等等；由用户提供；注意：Shiro 不知道你的用户 / 权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的 Realm；
- Cryptograpghy：是用来做加密数据的

大致的流程是：主体提交请求到Security Manager，Security Manager调用Authenticor 做认证，Authenticor 获取我们的认证数据是通过Realm中获取的。从数据库中获取的认证信息和我们主体提交过来的认证数据做比对

## Shiro认证

**身份验证**，即在应用中谁能证明他就是他本人。一般提供如他们的身份 ID 一些标识信息来表明他就是他本人，如提供身份证，用户名 / 密码来证明。

在 shiro 中，用户需要提供 principals （身份）和 credentials（证明）给 shiro，从而应用能验证用户身份：

**principals**：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个 principals，但只有一个 Primary principals，一般是用户名 / 密码 / 手机号。

**credentials**：证明 / 凭证，即只有主体知道的安全值，如密码 / 数字证书等。

最常见的 principals 和 credentials 组合就是用户名 / 密码了。接下来先进行一个基本的身份认证。

![mark](http://www.hhaxmm.cn/blog/20190114/dAr7NFW4z2Ue.png)



![mark](http://www.hhaxmm.cn/blog/20190114/S4lfkzCCfe0P.png)

应用代码直接交互的对象是 Subject，也就是说 Shiro 的对外 API 核心就是 Subject；其每个 API 的含义：

**Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet 前端控制器；

**Realm**：域，Shiro 从从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。

也就是说对于我们而言，最简单的一个 Shiro 应用：

1. 应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager；
2. 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法的用户及其权限进行判断。

**从以上也可以看出，Shiro 不提供维护用户 / 权限，而是通过 Realm 让开发人员自己注入。**

SpringBoot引入Shiro

pom.xml

```xml
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.4.0</version>
</dependency>
```

简单的测试类

```java
package com.huanghe.loginmodule;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.SimpleAccountRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Before;
import org.junit.Test;

/**
 * @Author huanghe
 * @Date 2019/1/14 21:21
 * @Description
 */
public class AuthenticationTest {

    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("huanghe","123456");
    }


    @Test
    public void testAuthentication(){
        //1、创建SecurityManager
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        //2、主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("huanghe", "123456");
        subject.login(token);

        System.out.println("isAuthenticated"+subject.isAuthenticated());

        subject.logout();

    }
}

```

通过代码来简单的分析下认证的流程：

1：创建SecurityManager

2：主体提交认证，这一步在代码中的体现就是` subject.login(token);`

3:   提交到认证管理器进行认证，进入到login的方法，`Subject subject = securityManager.login(this, token);`这一句代码就体现出主体提交的认证是SecurityManager进行进行认证的

```java
public void login(AuthenticationToken token) throws AuthenticationException {
        clearRunAsIdentitiesInternal();
        Subject subject = securityManager.login(this, token);

        PrincipalCollection principals;
```

4：进行进入到login方法，info = authenticate(token);从代码中可以看到是调用authenticator方法进行认证的，authenticator调用authenticate传入用户名和密码

```java
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info;
        try {
            info = authenticate(token);
        } catch (AuthenticationException ae) {
```

```java
public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        return this.authenticator.authenticate(token);
    }
```

5：authenticator通过Collection<Realm> realms = getRealms();获取到realm,通过reaml获取到认证数据

```java
public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {

        if (token == null) {
            throw new IllegalArgumentException("Method argument (authentication token) cannot be null.");
        }

        log.trace("Authentication attempt received for token [{}]", token);

        AuthenticationInfo info;
        try {
            info = doAuthenticate(token);
            if (info == null) {
                String msg = "No account information found for authentication token [" + token + "] by this " +
                        "Authenticator instance.  Please check that it is configured correctly.";
                throw new AuthenticationException(msg);
            }
```

```java
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        assertRealmsConfigured();
        Collection<Realm> realms = getRealms();
        if (realms.size() == 1) {
            return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
        } else {
            return doMultiRealmAuthentication(realms, authenticationToken);
        }
    }
```

**从如上代码可总结出身份验证的步骤**：

1. 收集用户身份 / 凭证，即如用户名 / 密码；
2. 调用 Subject.login 进行登录，如果失败将得到相应的 AuthenticationException 异常，根据异常提示用户错误信息；否则登录成功；
3. 最后调用 Subject.logout 进行退出操作。

如上测试的几个问题：

1. 用户名 / 密码硬编码在 ini 配置文件，以后需要改成如数据库存储，且密码需要加密存储；
2. 用户身份 Token 可能不仅仅是用户名 / 密码，也可能还有其他的，如登录时允许用户名 / 邮箱 / 手机号同时登录。

 **身份认证流程**

![mark](http://www.hhaxmm.cn/blog/20190114/Jkx4QXtsYRa1.png)

流程如下：

1. 首先调用 Subject.login(token) 进行登录，其会自动委托给 Security Manager，调用之前必须通过 SecurityUtils.setSecurityManager() 设置；
2. SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3. Authenticator 才是真正的身份验证者，Shiro API 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认 ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证；
5. Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。

## Realm

Realm：域，Shiro 从从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。如我们之前的 ini 配置方式将使用 org.apache.shiro.realm.text.IniRealm。

org.apache.shiro.realm.Realm 接口如下：

```java
String getName(); //返回一个唯一的Realm名字
boolean supports(AuthenticationToken token); //判断此Realm是否支持此Token
AuthenticationInfo getAuthenticationInfo(AuthenticationToken token)
throws AuthenticationException;  //根据Token获取认证信息
```

**单 Realm 配置**

1、自定义 Realm 实现

```java
public class MyRealm1 implements Realm {
    @Override
    public String getName() {
        return "myrealm1";
    }
    @Override
    public boolean supports(AuthenticationToken token) {
        //仅支持UsernamePasswordToken类型的Token
        return token instanceof UsernamePasswordToken; 
    }
    @Override
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String)token.getPrincipal();  //得到用户名
        String password = new String((char[])token.getCredentials()); //得到密码
        if(!"zhang".equals(username)) {
            throw new UnknownAccountException(); //如果用户名错误
        }
        if(!"123".equals(password)) {
            throw new IncorrectCredentialsException(); //如果密码错误
        }
        //如果身份认证验证成功，返回一个AuthenticationInfo实现；
        return new SimpleAuthenticationInfo(username, password, getName());
    }
}
```

2、ini 配置文件指定自定义 Realm 实现 (shiro-realm.ini)

```ini
#声明一个realm
myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1
#指定securityManager的realms实现
securityManager.realms=$myRealm1;
```

通过 $name 来引入之前的 realm 定义

3、测试用例请参考 com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest 的 testCustomRealm 测试方法，只需要把之前的 shiro.ini 配置文件改成 shiro-realm.ini 即可。

**多 Realm 配置**

1、ini 配置文件（shiro-multi-realm.ini）

```ini
#声明一个realm
myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1
myRealm2=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm2
#指定securityManager的realms实现
securityManager.realms=$myRealm1,$myRealm2;
```

securityManager 会按照 realms 指定的顺序进行身份认证。此处我们使用显示指定顺序的方式指定了 Realm 的顺序，如果删除 “securityManager.realms=$myRealm1,$myRealm2”，那么securityManager 会按照 realm 声明的顺序进行使用（即无需设置 realms 属性，其会自动发现），当我们显示指定 realm 后，其他没有指定 realm 将被忽略，如 “securityManager.realms=$myRealm1”，那么 myRealm2 不会被自动设置进去。

2、测试用例请参考 com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest 的 testCustomMultiRealm 测试方法。

**Shiro 默认提供的 Realm**

![mark](http://www.hhaxmm.cn/blog/20190114/2SWIOike2rUT.png)

以后一般继承 AuthorizingRealm（授权）即可；其继承了 AuthenticatingRealm（即身份验证），而且也间接继承了 CachingRealm（带有缓存实现）。其中主要默认实现如下：

**org.apache.shiro.realm.text.IniRealm**：[users] 部分指定用户名 / 密码及其角色；[roles] 部分指定角色即权限信息；

```ini
[users]
zhang=123
wang=123
```

**org.apache.shiro.realm.text.PropertiesRealm**： user.username=password,role1,role2 指定用户名 / 密码及其角色；role.role1=permission1,permission2 指定角色及权限信息；

**org.apache.shiro.realm.jdbc.JdbcRealm**：通过 sql 查询相应的信息，如 “select password from users where username = ?” 获取用户密码，“select password, password_salt from users where username = ?” 获取用户密码及盐；“select role_name from user_roles where username = ?” 获取用户角色；“select permission from roles_permissions where role_name = ?” 获取角色对应的权限信息；也可以调用相应的 api 进行自定义 sql；

**JDBC Realm 使用**

1、数据库及依赖

2、到数据库 shiro 下建三张表：users（用户名 / 密码）、user_roles（用户 / 角色）、roles_permissions（角色 / 权限），具体请参照 shiro-example-chapter2/sql/shiro.sql；并添加一个用户记录，用户名 / 密码为 zhang/123；

3、ini 配置（shiro-jdbc-realm.ini）

```ini
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiro
dataSource.username=root
dataSource.password=ab2453282
jdbcRealm.dataSource=$dataSource
securityManager.realms=$jdbcRealm;
```

## Authenticator 及 AuthenticationStrategy

Authenticator 的职责是验证用户帐号，是 Shiro API 中身份验证核心的入口点：

```java
public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)
            throws AuthenticationException;&nbsp;
```

如果验证成功，将返回 AuthenticationInfo 验证信息；此信息中包含了身份及凭证；如果验证失败将抛出相应的 AuthenticationException 实现。

SecurityManager 接口继承了 Authenticator，另外还有一个 ModularRealmAuthenticator 实现，其委托给多个 Realm 进行验证，验证规则通过 AuthenticationStrategy 接口指定，默认提供的实现：

**FirstSuccessfulStrategy**：只要有一个 Realm 验证成功即可，只返回第一个 Realm 身份验证成功的认证信息，其他的忽略；

**AtLeastOneSuccessfulStrategy**：只要有一个 Realm 验证成功即可，和 FirstSuccessfulStrategy 不同，返回所有 Realm 身份验证成功的认证信息；

**AllSuccessfulStrategy**：所有 Realm 验证成功才算成功，且返回所有 Realm 身份验证成功的认证信息，如果有一个失败就失败了。

ModularRealmAuthenticator 默认使用 AtLeastOneSuccessfulStrategy 策略。

假设我们有三个 realm：
myRealm1： 用户名 / 密码为 zhang/123 时成功，且返回身份 / 凭据为 zhang/123；
myRealm2： 用户名 / 密码为 wang/123 时成功，且返回身份 / 凭据为 wang/123；
myRealm3： 用户名 / 密码为 zhang/123 时成功，且返回身份 / 凭据为 zhang@163.com/123，和 myRealm1 不同的是返回时的身份变了；

1、ini 配置文件 (shiro-authenticator-all-success.ini)

```ini
#指定securityManager的authenticator实现
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator
securityManager.authenticator=$authenticator
#指定securityManager.authenticator的authenticationStrategy
allSuccessfulStrategy=org.apache.shiro.authc.pam.AllSuccessfulStrategy
securityManager.authenticator.authenticationStrategy=$allSuccessfulStrategy
```

```ini
myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1
myRealm2=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm2
myRealm3=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm3
securityManager.realms=$myRealm1,$myRealm3
```

2、测试代码（com.github.zhangkaitao.shiro.chapter2.AuthenticatorTest）

- 首先通用化登录逻辑

```java
private void login(String configFile) {
        //1、获取SecurityManager工厂，此处使用Ini配置文件初始化SecurityManager
        Factory<org.apache.shiro.mgt.SecurityManager> factory =
                new IniSecurityManagerFactory(configFile);
        //2、得到SecurityManager实例 并绑定给SecurityUtils
        org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //3、得到Subject及创建用户名/密码身份验证Token（即用户身份/凭证）
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");
        subject.login(token);
    }
```

- 测试 AllSuccessfulStrategy 成功

```java
@Test
    public void testAllSuccessfulStrategyWithSuccess() {
        login("classpath:shiro-authenticator-all-success.ini");
        Subject subject = SecurityUtils.getSubject();
        //得到一个身份集合，其包含了Realm验证成功的身份信息
        PrincipalCollection principalCollection = subject.getPrincipals();
        Assert.assertEquals(2, principalCollection.asList().size());
    }
```

即 PrincipalCollection 包含了 zhang 和 zhang@163.com 身份信息。

- 测试 AllSuccessfulStrategy 失败：

```java
 @Test(expected = UnknownAccountException.class)
    public void testAllSuccessfulStrategyWithFail() {
        login("classpath:shiro-authenticator-all-fail.ini");
        Subject subject = SecurityUtils.getSubject();
};
```

**自定义 AuthenticationStrategy 实现**，

首先看其 API：

```java
//在所有Realm验证之前调用
AuthenticationInfo beforeAllAttempts(
Collection<? extends Realm> realms, AuthenticationToken token) 
throws AuthenticationException;
//在每个Realm之前调用
AuthenticationInfo beforeAttempt(
Realm realm, AuthenticationToken token, AuthenticationInfo aggregate) 
throws AuthenticationException;
//在每个Realm之后调用
AuthenticationInfo afterAttempt(
Realm realm, AuthenticationToken token, 
AuthenticationInfo singleRealmInfo, AuthenticationInfo aggregateInfo, Throwable t)
throws AuthenticationException;
//在所有Realm之后调用
AuthenticationInfo afterAllAttempts(
AuthenticationToken token, AuthenticationInfo aggregate) 
throws AuthenticationException;
```

因为每个 AuthenticationStrategy 实例都是无状态的，所有每次都通过接口将相应的认证信息传入下一次流程；通过如上接口可以进行如合并 / 返回第一个验证成功的认证信息。自定义实现时一般继承 org.apache.shiro.authc.pam.AbstractAuthenticationStrategy 即可