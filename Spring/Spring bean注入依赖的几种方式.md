## Bean注入属性有哪几种方式？    

spring支持构造器注入和setter方法注入 构造器注入，通过` <constructor-arg>` 元素完成注入 setter方法注入， 通过`<property>` 元素完成注入【开发中常用方式】    



### spring配置bean实例化有哪些方式？    

1）使用类构造器实例化(默认无参数) `<bean id="bean1" class="cn.itcast.spring.b_instance.Bean1"></bean>   `

2）使用静态工厂方法实 例化(简单工厂模式)    //下面这段配置的含义：调用Bean2Factory的getBean2方法得到bean2 `<bean id="bean2" class="cn.itcast.spring.b_instance.Bean2Factory" factory-method="getBean2"> </bean> `   

3）使用实例工厂方法实例化(工厂方法模式)    

`<bean id = "bean3" factory-bean="bean3Factory" factory-method="getBean3"></bean>   ` 