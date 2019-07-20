# [设计模式]代理模式

## 定义

定义：给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象间接地操控原对象。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180514/hKK3EbgcfE.png?imageslim)

在上图中：

- RealSubject 是原对象（本文把原对象称为"委托对象"），Proxy 是代理对象。
- Subject 是委托对象和代理对象都共同实现的接口。
- `Request()` 是委托对象和代理对象共同拥有的方法。

要理解代理模式很简单，其实生活当中就存在代理模式：

> 我们购买火车票可以去火车站买，但是也可以去火车票代售处买，此处的火车票代售处就是火车站购票的代理，即我们在代售点发出买票请求，代售点会把请求发给火车站，火车站把购买成功响应发给代售点，代售点再告诉你。
> 但是代售点只能买票，不能退票，而火车站能买票也能退票，因此代理对象支持的操作可能和委托对象的操作有所不同。

再举一个写程序会碰到的一个例子：

> 如果现在有一个已有项目（你没有源代码，只能调用它）能够调用 `int compute(String exp1)` 实现对于后缀表达式的计算，你想使用这个项目实现对于中缀表达式的计算，那么你可以写一个代理类，并且其中也定义一个`compute(String exp2)`，这个`exp2`参数是中缀表达式，因此你需要在调用已有项目的 `compute()` 之前将中缀表达式转换成后缀表达式(Preprocess)，再调用已有项目的`compute()`，当然你还可以接收到返回值之后再做些其他操作比如存入文件(Postprocess)，这个过程就是使用了代理模式。

在平时用电脑也会碰到代理模式的应用：

> 远程代理：我们在国内因为GFW，所以不能访问 facebook，我们可以用翻墙（设置代理）的方法访问。访问过程是：
> (1)用户把HTTP请求发给代理
> (2)代理把HTTP请求发给web服务器
> (3)web服务器把HTTP响应发给代理
> (4)代理把HTTP响应发回给用户

Java 实现上面的UML图的代码（即实现静态代理）为：

```java
public class ProxyDemo {
    public static void main(String args[]){
        RealSubject subject = new RealSubject();
        Proxy p = new Proxy(subject);
        p.request();
    }
}

interface Subject{
    void request();
}

class RealSubject implements Subject{
    public void request(){
        System.out.println("request");
    }
}

class Proxy implements Subject{
    private Subject subject;
    public Proxy(Subject subject){
        this.subject = subject;
    }
    public void request(){
        System.out.println("PreProcess");
        subject.request();
        System.out.println("PostProcess");
    }
}

```

代理的实现分为：

- 静态代理：代理类是在编译时就实现好的。也就是说 Java 编译完成后代理类是一个实际的 class 文件。
- 动态代理：代理类是在运行时生成的。也就是说 Java 编译完之后并没有实际的 class 文件，而是在运行时动态生成的类字节码，并加载到JVM中。

静态代理的缺点：

 	1. 因为代理对象需要和原对象（委托对象）实现一样的接口。所以会很多代理类，类太多；
    2. 假如接口增加方法，目标对象和代理对象都需要进行维护

## Java 实现动态代理

在前一节我们已经介绍了静态代理（第一节已经实现）和动态代理，那么在 Java 中是如何实现动态代理，即如何做到在运行时动态的生成代理类呢？

首先先说明几个词：

- 委托类和委托对象：委托类是一个类，委托对象是委托类的实例。
- 代理类和代理对象：代理类是一个类，代理对象是代理类的实例。

Java实现动态代理的大致步骤如下：

1. 定义一个委托类和公共接口。
2. 自己定义一个类（调用处理器类，即实现 `InvocationHandler` 接口），这个类的目的是指定运行时将生成的代理类需要完成的具体任务（包括Preprocess和Postprocess），即代理类调用任何方法都会经过这个调用处理器类（在本文最后一节对此进行解释）。
3. 生成代理对象（当然也会生成代理类），需要为他指定(1)委托对象(2)实现的一系列接口(3)调用处理器类的实例。因此可以看出一个代理对象对应一个委托对象，对应一个调用处理器实例。

Java 实现动态代理主要涉及以下几个类：

- `java.lang.reflect.Proxy`: 这是生成代理类的主类，通过 Proxy 类生成的代理类都继承了 Proxy 类，即 `DynamicProxyClass extends Proxy`。
- `java.lang.reflect.InvocationHandler`: 这里称他为"调用处理器"，他是一个接口，我们动态生成的代理类需要完成的具体内容需要自己定义一个类，而这个类必须实现 InvocationHandler 接口。

Proxy 类主要方法为：

```java
//创建代理对象  
static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
```

​	这个静态函数的第一个参数是类加载器对象（即哪个类加载器来加载这个代理类到 JVM 的方法区），第二个参数是接口（表明你这个代理类需要实现哪些接口），第三个参数是调用处理器类实例（指定代理类中具体要干什么）。这个函数是 JDK 为了程序员方便创建代理对象而封装的一个函数，因此你调用`newProxyInstance()`时直接创建了代理对象（略去了创建代理类的代码）。其实他主要完成了以下几个工作:

```java
static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler handler)
{
    //1. 根据类加载器和接口创建代理类
    Class clazz = Proxy.getProxyClass(loader, interfaces); 
    //2. 获得代理类的带参数的构造函数
    Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class });
    //3. 创建代理对象，并制定调用处理器实例为参数传入
    Interface Proxy = (Interface)constructor.newInstance(new Object[] {handler});
}

```

Proxy 类还有一些静态方法，比如：

- `InvocationHandler getInvocationHandler(Object proxy)`: 获得代理对象对应的调用处理器对象。
- `Class getProxyClass(ClassLoader loader, Class[] interfaces)`: 根据类加载器和实现的接口获得代理类。

Proxy 类中有一个映射表，映射关系为：(<ClassLoader>,(<Interfaces>,<ProxyClass>) )，可以看出一级key为类加载器，根据这个一级key获得二级映射表，二级key为接口数组，因此可以看出：一个类加载器对象和一个接口数组确定了一个代理类。

我们写一个简单的例子来阐述 Java 实现动态代理的整个过程：

```java
public class DynamicProxyDemo01 {
    public static void main(String[] args) {
        RealSubject realSubject = new RealSubject();    //1.创建委托对象
        ProxyHandler handler = new ProxyHandler(realSubject);   //2.创建调用处理器对象
        Subject proxySubject = (Subject)Proxy.newProxyInstance(RealSubject.class.getClassLoader(),
                                                        RealSubject.class.getInterfaces(), handler);    //3.动态生成代理对象
        proxySubject.request(); //4.通过代理对象调用方法
    }
}

/**
 * 接口
 */
interface Subject{
    void request();
}

/**
 * 委托类
 */
class RealSubject implements Subject{
    public void request(){
        System.out.println("====RealSubject Request====");
    }
}
/**
 * 代理类的调用处理器
 */
class ProxyHandler implements InvocationHandler{
    private Subject subject;
    public ProxyHandler(Subject subject){
        this.subject = subject;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        System.out.println("====before====");//定义预处理的工作，当然你也可以根据 method 的不同进行不同的预处理工作
        Object result = method.invoke(subject, args);
        System.out.println("====after====");
        return result;
    }
}
```

第二种方式：使用匿名内部类的形式

```java
public class ProxyFactory{
  
 	//动态代理对象不需要实现和委托对象一样的接口，但是需要知道使用了哪些接口
  	private Object target;
  	//维护一个委托对象
  	public ProxyFactory(Object target){
      	this.target=target;
  	}
  	//给委托对象生成一个代理对象
  	public static Object getProxyInstance(){
      	return Proxy.newProxyInstance(target.class.getClassLoader(), 															   target.class.getInterfaces(),
                           new InvocationHandler(){
                             @Override
    						public Object invoke(Object proxy, Method method, Object[] args)
            													throws Throwable {
                               System.out.println("====before====");
                             	//执行委托对象的方法
                                Object result = method.invoke(target, args);
                                System.out.println("====after====");
                                return result;
                          	}
                           });
                        }
  }
/**
 * 接口
 */
interface Subject{
    void request();
}

/**
 * 委托类：一定要实现接口，否则是不能使用动态代理的
 */
class RealSubject implements Subject{
    public void request(){
        System.out.println("====RealSubject Request====");
    }
}
```

InvocationHandler 接口中有方法：

```java
invoke(Object proxy, Method method, Object[] args)
```

这个函数是在代理对象调用任何一个方法时都会调用的，**方法不同会导致第二个参数method不同**，第一个参数是代理对象（表示哪个代理对象调用了method方法），第二个参数是 Method 对象（表示哪个方法被调用了），第三个参数是指定调用方法的参数。

动态生成的代理类具有几个特点：

- 继承 Proxy 类，并实现了在`Proxy.newProxyInstance()`中提供的接口数组。
- 命名方式为 `$ProxyN`，其中N会慢慢增加，一开始是 `$Proxy1`，接下来是`$Proxy2`...
- 有一个参数为 InvocationHandler 的构造函数。这个从 `Proxy.newProxyInstance()` 函数内部的`clazz.getConstructor(new Class[] { InvocationHandler.class })` 可以看出。

Java 实现动态代理的缺点：因为 Java 的单继承特性（每个代理类都继承了 Proxy 类），只能针对接口创建代理类，不能针对类创建代理类。

## Cglib代理

- JDK的动态代理有一个限制，就是使用动态代理的对象必须实现一个或多个接口。如果想代理没有实现接口的类，就可以使用CGLIB实现。
- CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。**它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop**，**为他们提供方法的interception（拦截）**。
- CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。

Cglib子类代理：

​         1)需要引入cglib – jar文件，但是spring的核心包中已经包括了cglib功能，所以直接引入spring-core-3.2.5.jar即可。

​         2）引入功能包后，就可以在内存中动态构建子类

​         3）代理的类不能为final， 否则报错。

​         4）目标对象的方法如果为final/static, 那么就不会被拦截，即不会执行目标对象额外的业务方法。

```java
package cn.itcast.c_cglib;

import java.lang.reflect.Method;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

/**
 * Cglib子类代理工厂
 * (对UserDao 在内存中动态构建一个子类对象)
 * @author Jie.Yuan
 *
 */
public class ProxyFactory implements MethodInterceptor{
	
	// 维护目标对象
	private Object target;
	public ProxyFactory(Object target){
		this.target = target;
	}
	
	// 给目标对象创建代理对象
	public Object getProxyInstance(){
		//1. 工具类
		Enhancer en = new Enhancer();
		//2. 设置父类
		en.setSuperclass(target.getClass());
		//3. 设置回调函数
		en.setCallback(this);
		//4. 创建子类(代理对象)
		return en.create();
	}
	

	@Override
	public Object intercept(Object obj, Method method, Object[] args,
			MethodProxy proxy) throws Throwable {
		
		System.out.println("开始事务.....");
		
		// 执行目标对象的方法
		Object returnValue = method.invoke(target, args);
		
		System.out.println("提交事务.....");
		
		return returnValue;
	}

}
```

方式二

```java
public class MyBeanFactory {
	
	public static UserServiceImpl createService(){
		//1 目标类
		final UserServiceImpl userService = new UserServiceImpl();
		//2切面类
		final MyAspect myAspect = new MyAspect();
		// 3.代理类 ，采用cglib，底层创建目标类的子类
		//3.1 核心类
		Enhancer enhancer = new Enhancer();
		//3.2 确定父类
		enhancer.setSuperclass(userService.getClass());
		/* 3.3 设置回调函数 , MethodInterceptor接口 等效 jdk InvocationHandler接口
		 * 	intercept() 等效 jdk  invoke()
		 * 		参数1、参数2、参数3：以invoke一样
		 * 		参数4：methodProxy 方法的代理
		 * 		
		 * 
		 */
		enhancer.setCallback(new MethodInterceptor(){

			@Override
			public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
				
				//前
				myAspect.before();
				
				//执行目标类的方法
				Object obj = method.invoke(userService, args);
				// * 执行代理类的父类 ，执行目标类 （目标类和代理类 父子关系）
				methodProxy.invokeSuper(proxy, args);
				
				//后
				myAspect.after();
				
				return obj;
			}
		});
		//3.4 创建代理
		UserServiceImpl proxService = (UserServiceImpl) enhancer.create();
		
		return proxService;
	}

}

```



在Spring的AOP编程中，

​         如果加入容器的目标对象有实现接口，用JDK代理；

​         如果目标对象没有实现接口，用Cglib代理；