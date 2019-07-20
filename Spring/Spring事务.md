# Spring的事务管理

[TOC]

## 概要

在谈事务之前我们先要了解一下什么是事务？对于目前我自己的知识水平层次来说，只能作出如下的回答：事务是逻辑上的一组操作要么都执行，要么都不执行；在计算机术语中是指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)。在计算机术语中，事务通常就是指数据库事务 ;事务是最小的执行单位，是不允许分割的；

##一、事务的回顾

### 1.1、事务的四大特性ACID

1、**原子性（Atomicity）**：是指事务所包含的操作要么全部成功，那么全部失败回滚;

2、**一致性（Consistency）**：事务执行前后，数据保持一致；

3、**隔离性（Isolation）**:并发访问数据库的时候，用户的事务不被其他的事务干扰，各并发事务之间的数据库是独立的；

4：**持久性（Durability）**:一个事物被提交之后，对数据库中数据的修改是持久的，即使数据库发生故障也不应该对其有任何的影响；

以上介绍完事务的四大特性(简称ACID)，现在重点来说明下事务的隔离性，当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性，在介绍数据库提供的各种隔离级别之前，我们先看看如果不考虑事务的隔离性，会发生的几种问题： 

###1.2、隔离问题

1、**脏读**：一个事务读到另一个事务没有提交的数据

　　当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。 

2、**不可重复读（Unrepeatableread）**：一个事务读到另一个事务已提交的数据（update);

　　在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据，并修改了数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读 ;

3、**幻读：一个事务读到另一个事务已提交的数据（insert）** 

　　幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。 

**不可重复度和幻读区别：**

　　不可重复读的重点是修改，幻读的重点在于新增或者删除。

　　例1（同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 ）：事务1中的A先生读取自己的工资为     1000的操作还没完成，事务2中的B先生就修改了A的工资为2000，导        致A再读自己的工资时工资变为  2000；这就是不可重复读。

　　例2（同样的条件, 第1次和第2次读出来的记录数不一样 ）：假某工资单表中工资大于3000的有4人，事务1读取了所有工资大于3000的人，共查到4条记录，这时事务2 又插入了一条工资大于3000的记录，事务1再次读取时查到的记录就变为了5条，这样就导致了幻读。

### 1.3、隔离级别

TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- **TransactionDefinition.ISOLATION_DEFAULT:**	使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
	 **TransactionDefinition.ISOLATION_READ_COMMITTED:** 	允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
	 **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 	对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
	 **TransactionDefinition.ISOLATION_SERIALIZABLE:** 	最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

 ### 1.4、事物的传播行为

　　**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播**。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量： 

 **支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

　　这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 **PROPAGATION_NESTED** 是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

 

 ## 二、Spring管理事物介绍

### 2.1、Spring事务管理接口：

- **PlatformTransactionManager：** （平台）事务管理器

- **TransactionDefinition：** 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)

- **TransactionStatus：** 事务运行状态

  **所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。** 

### 2.2、PlatformTransactionManager接口介绍

**Spring并不直接管理事务，而是提供了多种事务管理器** ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： **org.springframework.transaction.PlatformTransactionManager** ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

PlatformTransactionManager接口代码如下：

```java
Public interface PlatformTransactionManager()...{  
    // Return a currently active transaction or create a new one, according to the specified propagation behavior（根据指定的传播行为，返回当前活动的事务或创建一个新事务。）
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // Commit the given transaction, with regard to its status（使用事务目前的状态提交事务）
    Void commit(TransactionStatus status) throws TransactionException;  
    // Perform a rollback of the given transaction（对执行的事务进行回滚）
    Void rollback(TransactionStatus status) throws TransactionException;  
    } 
```

我们刚刚也说了Spring中PlatformTransactionManager根据不同持久层框架所对应的接口实现类,几个比较常见的如下图所示 

![img](https://user-gold-cdn.xitu.io/2018/5/20/1637b21877cf626d?imageslim) 

 比如我们在使用JDBC或者iBatis（就是Mybatis）进行数据持久化操作时,我们的xml配置通常如下：

 ```xml
<!-- 事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 数据源 -->
		<property name="dataSource" ref="dataSource" />
	</bean>
 ```

###2.3、TransactionDefinition事物详情（事物属性、事物定义）

事务管理器接口 **PlatformTransactionManager** 通过 **getTransaction(TransactionDefinition definition)** 方法来得到一个事务，这个方法里面的参数是 **TransactionDefinition类** ，这个类就定义了一些基本的事务属性。

 **那么什么是事务属性呢？**

事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面。

![1532607632157](../../../../AppData/Local/Temp/1532607632157.png)

### TransactionDefinition接口中的方法如下：

TransactionDefinition接口中定义了5个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等的常量。

我下面只是列出了TransactionDefinition接口中的方法而没有给出接口中定义的常量，该接口中的常量信息会在后面依次介绍到。

![1532607751460](../../../../AppData/Local/Temp/1532607751460.png)

 **事务超时属性**(一个事务允许执行的最长时间)

　　所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

**事务只读属性**（对事物资源是否执行只读操作）

　　事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。

**回滚规则**（定义事务回滚规则）

　这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）。 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

 ## 三、事务的使用

从一个经典的转账案例来说一下的四种事务的使用方式：

**基于 TransactionInterceptor 的声明式事务:** Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。

**基于 TransactionProxyFactoryBean 的声明式事务:** 第一种方式的改进版本，简化的配置文件的书写，这是 Spring 早期推荐的声明式事务管理方式，但是在 Spring 2.0 中已经不推荐了。

**基于<  tx> 和< aop>命名空间的声明式事务管理：** 目前推荐的方式，其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。

**基于 @Transactional 的全注解方式：** 将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。

### 3.1、环境的搭建

**创建表**：

```sql
create database ee19_spring_day03;
use ee19_spring_day03;
create table account(
  id int primary key auto_increment,
  username varchar(50),
  money int
);
insert into account(username,money) values('jack','10000');
insert into account(username,money) values('rose','10000');
```

配置相关的jar包，由于我这里使用的是maven工程，所以使用pom文件如下

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!-- 添加Spring依赖 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.11</version>
    </dependency>
    <dependency>
      <groupId>aopalliance</groupId>
      <artifactId>aopalliance</artifactId>
      <version>1.0</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>3.2.0.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib</artifactId>
      <version>3.1</version>
    </dependency>


    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
      <version>3.1.1.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.0.5</version>
    </dependency>

    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.2</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.3.13.RELEASE</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>3.2.0.RELEASE</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

**dao层**：

```java
public class AccountDaoImpl extends JdbcDaoSupport  implements AccountDao {

    @Override
    public void out(String outer, Integer money) {
        this.jdbcTemplate.update("update account set money = money - ? where username = ?", money,outer);
    }

    @Override
    public void in(String inner, Integer money) {
        this.jdbcTemplate.update("update account set money = money + ? where username = ?", v              money,inner);
    }
}
```

**Service层**：

```java
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void transfer(String outer, String inner, Integer money) {
        accountDao.out(outer, money);
        accountDao.in(inner, money);
    }
}
```

**Spring配置：**

```xml
	<!-- 1 datasource -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="com.mysql.jdbc.Driver"></property>
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/ee19_spring_day03"></property>
		<property name="user" value="root"></property>
		<property name="password" value="1234"></property>
	</bean>
	
	<!-- 2 dao  -->
	<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<!-- 3 service -->
	<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
		<property name="accountDao" ref="accountDao"></property>
	</bean>

```

**测试：**

```java
	@Test
	public void demo01(){
		String xmlPath = "applicationContext.xml";
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
		AccountService accountService =  (AccountService) applicationContext.getBean("accountService");
		accountService.transfer("jack", "rose", 1000);
	}
```

### 3.2、手动的管理事物

 spring底层使用 TransactionTemplate 事务模板进行操作。

 操作如下：

1.service 需要获得 TransactionTemplate 

2.spring 配置模板，并注入给service

3.模板需要注入事务管理器

4.配置事务管理器：DataSourceTransactionManager ，需要注入DataSource

**修改service：**

```java
     //需要spring注入模板
	private TransactionTemplate transactionTemplate;
	public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
		this.transactionTemplate = transactionTemplate;
	}

	@Override
	public void transfer(final String outer,final String inner,final Integer money) {
        //没有返回值的匿名内部类
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus arg0) {
				accountDao.out(outer, money);
				//断电
//				int i = 1/0;
				accountDao.in(inner, money);
			}
		});

	}
```

 **修改spring配置**

```xml
     <!-- 3 service -->
	<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
		<property name="accountDao" ref="accountDao"></property>
		<property name="transactionTemplate" ref="transactionTemplate"></property>
	</bean>
	
	<!-- 4 创建模板 -->
	<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
		<property name="transactionManager" ref="txManager"></property>
	</bean>
	
	<!-- 5 配置事务管理器 ,管理器需要事务，事务从Connection获得，连接从连接池DataSource获得 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>

```

### 3.3、 工厂bean 生成代理：半自动

spring提供 管理事务的代理工厂bean TransactionProxyFactoryBean

1.getBean() 获得代理对象

2.spring 配置一个代理

Spring的配置文件

```xml
<!-- 4 service 代理对象 
		4.1 proxyInterfaces 接口 
		4.2 target 目标类
		4.3 transactionManager 事务管理器
		4.4 transactionAttributes 事务属性（事务详情）
			prop.key ：确定哪些方法使用当前事务配置
			prop.text:用于配置事务详情
				格式：PROPAGATION,ISOLATION,readOnly,-Exception,+Exception
					传播行为		隔离级别	是否只读		异常回滚		异常提交
				例如：
					<prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT</prop> 默认传播行为，和隔离级别
					<prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly</prop> 只读
					<prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT,+java.lang.ArithmeticException</prop>  有异常扔提交
	-->
	<bean id="proxyAccountService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
		<property name="proxyInterfaces" value="com.itheima.service.AccountService"></property>
		<property name="target" ref="accountService"></property>
		<property name="transactionManager" ref="txManager"></property>
		<property name="transactionAttributes">
			<props>
				<prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT</prop>
			</props>
		</property>
	</bean>


	<!-- 5 事务管理器 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>

```

###3.4、AOP 配置基于xml【掌握】

 在spring xml 配置aop 自动生成代理，进行事务的管理

1.配置管理器

2.配置事务详情

3.配置aop

```xml
<!-- 4 事务管理 -->
	<!-- 4.1 事务管理器 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<!-- 4.2 事务详情（事务通知）  ， 在aop筛选基础上，对ABC三个确定使用什么样的事务。例如：AC读写、B只读 等
		<tx:attributes> 用于配置事务详情（属性属性）
			<tx:method name=""/> 详情具体配置
				propagation 传播行为 ， REQUIRED：必须；REQUIRES_NEW:必须是新的
				isolation 隔离级别
	-->
	<tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="transfer" propagation="REQUIRED" isolation="DEFAULT"/>
		</tx:attributes>
	</tx:advice>
	<!-- 4.3 AOP编程，目标类有ABCD（4个连接点），切入点表达式 确定增强的连接器，从而获得切入点：ABC -->
	<aop:config>
		<aop:advisor advice-ref="txAdvice" pointcut="execution(* com.itheima.service..*.*(..))"/>
	</aop:config>

```

### 3.5、AOP配置基于注解的方式

 1.配置事务管理器，将并事务管理器交予spring

 2.在目标类或目标方法添加注解即可 @Transactional

```xml
<!-- 4 事务管理 -->
	<!-- 4.1 事务管理器 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<!-- 4.2 将管理器交予spring 
		* transaction-manager 配置事务管理器
		* proxy-target-class
			true ： 底层强制使用cglib 代理
	-->
	<tx:annotation-driven transaction-manager="txManager"/>

```

**ServiceImpl的改造：**

```java
@Transactional(propagation=Propagation.REQUIRED , isolation = Isolation.DEFAULT)
public class AccountServiceImpl implements AccountService {

```

### 3.6、整个类基于的Java配置的使用

config.java

```java
@Configuration
@ComponentScan(basePackages = {"com.huanghe.*"})
@PropertySource("classpath:jdbc.properties")
public class configuration {

    @Value("${jdbc.url}")
    private String jdbcUrl;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Value("${jdbc.driverClassName}")
    private String driverClassName;


    public void show(){
        System.out.println(driverClassName);
        System.out.println(username);
        System.out.println(password);
    }


    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    /**
     * 配置数据源
     * @return
     * @throws PropertyVetoException
     */
    @Bean
    public ComboPooledDataSource  dataSources() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driverClassName);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        dataSource.setJdbcUrl(jdbcUrl);
        return dataSource;
    }

    /**
     * 配置JDBCTemplate
     * @return
     * @throws PropertyVetoException
     */
    @Bean
    public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSources());
        return jdbcTemplate;
    }

    /**
     * 配置事物管理器
     * @return
     * @throws PropertyVetoException
     */
    @Bean
    public DataSourceTransactionManager dataSourceTransactionManager() throws PropertyVetoException {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSources());
        return dataSourceTransactionManager;
    }

}

```

**Service的配置：**

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }


    @Override
    @Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.DEFAULT)
    public void transfer(String outer, String inner, Integer money) {
        accountDao.out(outer, money);
        accountDao.in(inner, money);
    }
}
```

**测试代码：**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = configuration.class)
public class TestAppAnnonation {

    @Autowired
    private AccountService accountService;

    @Test
    public void test() {
        accountService.transfer("jack", "rose", 1000);
    }
}
```



 

　 

 

 

 

 

 



 

 

 

 

 

 

 

 

 





