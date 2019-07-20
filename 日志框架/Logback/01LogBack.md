# Logback详解

[TOC]

## 一、logback介绍

记得刚开始学习Java日志框架的时候，大部分的情况下使用的日志框架还是log4j，大约从16年中到现在，不管是我参与的别人已经搭建好的项目还是我自己主导的项目，日志框架基本都换成了logback，并且Logback是由log4j创始人设计的又一个开源日志组件。logback当前分成三个模块：

- logback-core：其它两个模块的基础模块，其它模块基于它构建。
- logback-classic：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j
- logback-access：主要作为一个与Servlet 容器交互的模块，比如说tomcat或者 jetty，提供一些与 HTTP访问相关的功能

## 二、logback取代log4j的理由

简单的总结一下logback的一些优点：

- 内核重写、测试充分、初始化内存加载更小，这一切让logback性能和log4j相比有诸多倍的提升
- logback非常自然地直接实现了slf4j，这个严格来说算不上优点，只是这样，再理解slf4j的前提下会很容易理解logback，也同时很容易用其他日志框架替换logback
- logback当配置文件修改了，支持自动重新加载配置文件，扫描过程快且安全，它并不需要另外创建一个扫描线程
- 支持自动去除旧的日志文件，可以控制已经产生日志文件的最大数量
- 配置文件可以处理不同的情况，开发人员经常需要判断不同的Logback配置文件在不同的环境下（开发，测试，生产）。而这些配置文件仅仅只有一些很小的不同，可以通过,和来实现，这样一个配置文件就可以适应多个环境。
- Filters（过滤器）。有些时候，需要诊断一个问题，需要打出日志。在log4j，只有降低日志级别，不过这样会打出大量的日志，会影响应用性能。在Logback，你可以继续 保持那个日志级别而除掉某种特殊情况，如alice这个用户登录，她的日志将打在DEBUG级别而其他用户可以继续打在WARN级别。要实现这个功能只需加4行XML配置。可以参考MDCFIlter

总而言之，如果大家的项目里面需要选择一个日志框架，那么我个人非常建议使用logback。

## 三、logback的配置介绍

**1、Logger、appender及layout**

- Logger作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。

-  Appender主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。

-  Layout 负责把事件转换成字符串，格式化的日志信息的输出。

**2、loggerContext**

   各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

**3、有效级别及级别的继承**

   Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

 **4、打印方法与基本的选择规则**

   打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
   该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR



## 四、logback的默认配置

如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为`%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n `的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG。

## 五、logback.xml常用配置详解

![mark](http://man.hhaxmm.cn/blog/20190607/uvSjuFG41S7L.png)

1、根节点`<configuration>`，包含下面三个属性：

- scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟
- debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　　　　　<!--其他配置省略--> 
</configuration>
```

2、子节点`<contextName>`：用来设置上下文名称，

每个logger都关联到logger上下文，默认上下文名称为default。但可以使用`<contextName>`设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。

```xml
　<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　　　　　<contextName>myAppName</contextName> 
　　　　　　<!--其他配置省略-->
</configuration>  
```

3、子节点`<property> `：用来定义变量值，它有两个属性name和value，通过`<property>`定义的值会被插入到logger上下文中，可以使`“${}”`来使用变量。

- name: 变量的名称
- value: 变量定义的值

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　　　　　<property name="APP_Name" value="myAppName" /> 
　　　　　　<contextName>${APP_Name}</contextName> 
　　　　　　<!--其他配置省略--> 
</configuration>
```

4、子节点`<timestamp>`：获取时间戳字符串，他有两个属性key和datePattern

- key: 标识此`<timestamp>` 的名字；
-  datePattern: 设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　　　　　<timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/> 
　　　　　　<contextName>${bySecond}</contextName> 
　　　　　　<!-- 其他配置省略--> 
</configuration>
```

5、子节点`<appender>`：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名。

-  5.1、ConsoleAppender 把日志输出到控制台，有以下子节点：

  -  `<encoder>`：对日志进行格式化。（具体参数稍后讲解 ）

  - `<target>`：字符串System.out(默认)或者System.err

表示把>=DEBUG级别的日志都输出到控制台:

```xml
　<configuration> 
　　　　　　<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
　　　　　　<encoder> 
　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern> 
　　　　　　</encoder> 
　　　　　　</appender> 

　　　　　　<root level="DEBUG"> 
　　　　　　　　<appender-ref ref="STDOUT" /> 
　　　　　　</root> 
</configuration>
```

- 5.2、FileAppender：把日志添加到文件，有以下子节点：

  - `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
  - `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
  - `<encoder>`：对记录事件进行格式化。（具体参数稍后讲解 ）
  - `<prudent>`：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

> 表示把>=DEBUG级别的日志都输出到testFile.log

```xml
<configuration> 
　　　　　　<appender name="FILE" class="ch.qos.logback.core.FileAppender"> 
　　　　　　　　<file>testFile.log</file> 
　　　　　　　　<append>true</append> 
　　　　　　　　<encoder> 
　　　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　　　　　</encoder> 
　　　　　　</appender> 

　　　　　　<root level="DEBUG"> 
　　　　　　<appender-ref ref="FILE" /> 
　　　　　　</root> 
　　　　</configuration>
```

- 5.3、RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

  - 　`<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

  - `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

  - `<rollingPolicy>`:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类。class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
- `<fileNamePattern>`：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个java.text.SimpleDateFormat指定的时间格式，如：`%d{yyyy-MM}`。如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
    
- `<maxHistory>`:可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且`<maxHistory>`是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。
> 配置表示每天生成一个日志文件，保存30天的日志文件。

```xml
<configuration> 
　　　　<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
　　　　<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
　　　　　　<fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern> 
　　　　　　<maxHistory>30</maxHistory> 
　　　　</rollingPolicy> 
　　　　<encoder> 
　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　</encoder> 
　　　　</appender> 

　　　　<root level="DEBUG"> 
　　　　　　　<appender-ref ref="FILE" /> 
　　　　</root> 
</configuration>
```

> 上述配置表示按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。

```xml
<configuration> 
　　<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
      
　　　<file>test.log</file> 
      
	<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy"> 
　　　　　　<fileNamePattern>tests.%i.log.zip</fileNamePattern> 
　　　　　　<minIndex>1</minIndex> 
　　　　　　<maxIndex>3</maxIndex> 
　　　</rollingPolicy> 

　　　<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"> 
　　　　　　<maxFileSize>5MB</maxFileSize> 
　　　</triggeringPolicy> 
      
　　　<encoder> 
　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　</encoder> 
　　</appender> 

　　<root level="DEBUG"> 
　　　　　<appender-ref ref="FILE" /> 
　　</root> 
</configuration>
```

　6、子节点`<logger>`：用来设置某一个包或具体的某一个类的日志打印级别、以及指定`<appender>`。`<logger>`仅有一个name属性，一个可选的level和一个可选的addtivity属性。可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

- name: 用来指定受此loger约束的某一个包或者具体的某一个类。
- level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
- addtivity: 是否向上级loger传递打印信息。默认是true

7、子节点`<root>`:它也是`<logger>`元素，但是它是根loger,是所有`<logger>`的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。

- level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，不能设置为INHERITED或者同义词NULL。 默认是DEBUG。

> **常用loger配置**   

```xml
<!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
<logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
<logger name="org.hibernate.SQL" level="DEBUG" />
<logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
<logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

<!--myibatis log configure-->
<logger name="com.apache.ibatis" level="TRACE"/>
<logger name="java.sql.Connection" level="DEBUG"/>
<logger name="java.sql.Statement" level="DEBUG"/>
<logger name="java.sql.PreparedStatement" level="DEBUG"/>
```

## 六、使用

1、pom.xml

```xml
<dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.7</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.7</version>
        </dependency>
    </dependencies>
```

2、配置文件logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 根标签 -->
<!-- 三个属性 
     scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
　　　scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
　　　debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 
-->
<configuration scan="true" scanPeriod="30 seconds">

    <!-- 用来全局定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使“${}”来使用变量 -->
    <!-- 日志文件保存路径 -->
    <property name="logPath" value="d:\\log"/>
    
    
    <!-- 子节点<appender>：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名-->
　　　<!--　ConsoleAppender 把日志输出到控制台，有以下子节点：  -->
    
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
　　　　　　<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
　　　　　　<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。    
　　　　　　<rollingPolicy>:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类-->
     <!-- 记录日志文件 -->
    <appender name="testLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logPath}/test.log</file>
        <append>true</append>
        <!-- class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
　　　　　　　　    <fileNamePattern>：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。
            如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；
            如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。     
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${logPath}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!-- <maxHistory>可选节点，
                控制保留的归档文件的最大数量，超出数量就删除旧文件。
                假设设置每个月滚动，且<maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。
                注意，删除旧文件是，那些为了归档而创建的目录也会被删除。
             -->
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <!-- 
            <encoder>：对记录事件进行格式化。负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
            PatternLayoutEncoder 是唯一有用的且默认的encoder ，
            有一个<pattern>节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。
         -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>
    
    <!-- 
        子节点<loger>：用来设置某一个包或具体的某一个类的日志打印级别、以及指定<appender>。
        <loger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。
        可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger
　　　　    name: 用来指定受此loger约束的某一个包或者具体的某一个类。
　　　　    level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
        addtivity: 是否向上级loger传递打印信息。默认是true。同<loger>一样，可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。
     -->
    <logger name="testLog" additivity="false" level="INFO">
        <appender-ref ref="stdout" />
        <appender-ref ref="testLogAppender" />
    </logger>
    
    <!-- 
        子节点<root>:它也是<loger>元素，但是它是根loger,是所有<loger>的上级。
        只有一个level属性，因为name已经被命名为"root",且已经是最上级了。
　　　　    level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，
        不能设置为INHERITED或者同义词NULL。 默认是DEBUG。  
     -->
    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>

```















