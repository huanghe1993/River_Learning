# Shiro授权

授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

![mark](http://www.hhaxmm.cn/blog/20190114/aAC4TBQEkjJm.png)

- Authenticor ：认证器，管理我们的登录，退出登录；如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；
- Authorizer：授权器，授予我们主体有哪些权限

## 授权方式

Shiro 支持三种方式的授权：

编程式：通过写 if/else 授权代码块完成：

```java
Subject subject = SecurityUtils.getSubject();
if(subject.hasRole(“admin”)) {
    //有权限
} else {
    //无权限
};
```

注解式：通过在执行的 Java 方法上放置相应的注解完成：

```java
@RequiresRoles("admin")
public void hello() {
    //有权限
}
```

没有权限将抛出相应的异常；

JSP/GSP 标签：在 JSP/GSP 页面通过相应的标签完成：

```java
<shiro:hasRole name="admin">
<!— 有权限 —>
</shiro:hasRole>
```

## 授权

![1547517215434](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1547517215434.png)

**基于角色的访问控制（隐式角色）**

1、在 ini 配置文件配置用户拥有的角色（shiro-role.ini）

```ini
[users]
zhang=123,role1,role2
wang=123,role1;
```

规则即：“用户名=密码,角色1，角色2”，如果需要在应用中判断用户是否有相应角色，就需要在相应的 Realm 中返回角色信息，也就是说 Shiro 不负责维护用户-角色信息，需要应用提供，Shiro 只是提供相应的接口方便验证，后续会介绍如何动态的获取用户角色。

2、测试用例

```java
@Test
    public void testHasRole() {
        login("classpath:shiro-role.ini", "zhang", "123");
        //判断拥有角色：role1
        Assert.assertTrue(subject().hasRole("role1"));
        //判断拥有角色：role1 and role2
        Assert.assertTrue(subject().hasAllRoles(Arrays.asList("role1", "role2")));
        //判断拥有角色：role1 and role2 and !role3
        boolean[] result = subject().hasRoles(Arrays.asList("role1", "role2", "role3"));
        Assert.assertEquals(true, result[0]);
        Assert.assertEquals(true, result[1]);
        Assert.assertEquals(false, result[2]);
    }
```

Shiro 提供了 hasRole/hasRole 用于判断用户是否拥有某个角色/某些权限；但是没有提供如 hashAnyRole 用于判断是否有某些权限中的某一个

```java
 @Test(expected = UnauthorizedException.class)
    public void testCheckRole() {
        login("classpath:shiro-role.ini", "zhang", "123");
        //断言拥有角色：role1
        subject().checkRole("role1");
        //断言拥有角色：role1 and role3 失败抛出异常
        subject().checkRoles("role1", "role3");
    };
```

Shiro 提供的 checkRole/checkRoles 和 hasRole/hasAllRoles 不同的地方是它在判断为假的情况下会抛出 UnauthorizedException 异常

到此基于角色的访问控制（即**隐式角色**）就完成了，这种方式的缺点就是**如果很多地方进行了角色判断，但是有一天不需要了那么就需要修改相应代码把所有相关的地方进行删除；这就是粗粒度造成的问题。**

**基于资源的访问控制（显示角色）**

1、在 ini 配置文件配置用户拥有的角色及角色-权限关系（shiro-permission.ini）

```ini
[users]
zhang=123,role1,role2
wang=123,role1
[roles]
role1=user:create,user:update
role2=user:create,user:delete;
```

规则：“用户名=密码，角色 1，角色 2”“角色=权限 1，权限 2”，即首先根据用户名找到角色，然后根据角色再找到权限；即角色是权限集合；Shiro 同样不进行权限的维护，需要我们通过 Realm 返回相应的权限信息。只需要维护“用户——角色”之间的关系即可。

2、测试用例

```java
@Test
    public void testIsPermitted() {
        login("classpath:shiro-permission.ini", "zhang", "123");
        //判断拥有权限：user:create
        Assert.assertTrue(subject().isPermitted("user:create"));
        //判断拥有权限：user:update and user:delete
        Assert.assertTrue(subject().isPermittedAll("user:update", "user:delete"));
        //判断没有权限：user:view
        Assert.assertFalse(subject().isPermitted("user:view"));
    }
```

Shiro 提供了 isPermitted 和 isPermittedAll 用于判断用户是否拥有某个权限或所有权限，也没有提供如 isPermittedAny 用于判断拥有某一个权限的接口

```java
@Test(expected = UnauthorizedException.class)
    public void testCheckPermission () {
        login("classpath:shiro-permission.ini", "zhang", "123");
        //断言拥有权限：user:create
        subject().checkPermission("user:create");
        //断言拥有权限：user:delete and user:update
        subject().checkPermissions("user:delete", "user:update");
        //断言拥有权限：user:view 失败抛出异常
        subject().checkPermissions("user:view");
    };
```

到此基于资源的访问控制（显示角色）就完成了，也可以叫基于权限的访问控制，这种方式的一般规则是“资源标识符：操作”，即是资源级别的粒度；这种方式的好处就是如果要修改基本都是一个资源级别的修改，不会对其他模块代码产生影响，粒度小。但是实现起来可能稍微复杂点，需要维护“用户——角色，角色——权限（资源：操作）”之间的关系。

## Permission

### 字符串通配符权限

规则：“资源标识符：操作：对象实例 ID” 即对哪个资源的哪个实例可以进行什么操作。其默认支持通配符权限字符串，“:”表示资源/操作/实例的分割；“,”表示操作的分割；“*”表示任意资源/操作/实例。

**1、单个资源单个权限**

```
subject().checkPermissions("system:user:update");
```

用户拥有资源“system:user”的“update”权限。

**2、单个资源多个权限**

```ini
role41=system:user:update,system:user:delete
```

然后通过如下代码判断

```
subject().checkPermissions("system:user:update", "system:user:delete");
```

用户拥有资源“system:user”的“update”和“delete”权限。如上可以简写成：

ini 配置（表示角色4拥有 system:user 资源的 update 和 delete 权限）

```ini
role42="system:user:update,delete"
```

接着可以通过如下代码判断

```
subject().checkPermissions("system:user:update,delete");
```

通过“system:user:update,delete”验证“system:user:update, system:user:delete”是没问题的，但是反过来是规则不成立。

**3、单个资源全部权限**

ini 配置

```ini
role51="system:user:create,update,delete,view"
```

然后通过如下代码判断

```
subject().checkPermissions("system:user:create,delete,update:view");
```

用户拥有资源“system:user”的“create”、“update”、“delete”和“view”所有权限。如上可以简写成：

ini 配置文件（表示角色 5 拥有 system:user 的所有权限）

```ini
role52=system:user:*
```

也可以简写为（推荐上边的写法）：

```ini
role53=system:user
```

然后通过如下代码判断

```java
subject().checkPermissions("system:user:*");
subject().checkPermissions("system:user");
```

通过“system:user:*”验证“system:user:create,delete,update:view”可以，但是反过来是不成立的。

**4、所有资源全部权限**

ini 配置

```ini
role61=*:view
```

然后通过如下代码判断

```
subject().checkPermissions("user:view");
```

用户拥有所有资源的“view”所有权限。假设判断的权限是“"system:user:view”，那么需要“role5=*:*:view”这样写才行

**5、实例级别的权限**

- 单个实例单个权限

ini 配置

```ini
role71=user:view:1
```

对资源 user 的 1 实例拥有 view 权限。

然后通过如下代码判断

```
subject().checkPermissions("user:view:1");
```

- 单个实例多个权限

ini 配置

```ini
role72="user:update,delete:1"
```

对资源 user 的 1 实例拥有 update、delete 权限。

然后通过如下代码判断

```java
subject().checkPermissions("user:delete,update:1");
subject().checkPermissions("user:update:1", "user:delete:1");
```

