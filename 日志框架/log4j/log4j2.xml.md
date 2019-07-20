# log4j2.xml

[TOC]



### 一、日志级别

在log4j2中，一共有五种log level，分别为：

```xml
TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```

**FATAL**：用在极端的情形中，即必须马上获得注意的情况。这个程度的错误通常需要触发运维工程师的寻呼机。

**ERROR**：显示一个错误，或一个通用的错误情况，但还不至于会将系统挂起。这种程度的错误一般会触发邮件的发送，将消息发送到alert list中，运维人员可以在文档中记录这个bug并提交。

**WARN**：不一定是一个bug，但是有人可能会想要知道这一情况。如果有人在读log文件，他们通常会希望读到系统出现的任何警告。

**INFO**：用于基本的、高层次的诊断信息。在长时间运行的代码段开始运行及结束运行时应该产生消息，以便知道现在系统在干什么。但是这样的信息不宜太过频繁。

**DEBUG**：用于协助低层次的

:warning:注意：

> 级别之间是包含的关系，意思是如果你设置日志级别是trace，则大于等于这个级别的日志都会输出。

### 二、输出格式全解

%d{HH:mm:ss.SSS} 表示输出到毫秒的时间

%t 输出当前线程名称

%-5level 输出日志级别，-5表示左对齐并且固定输出5个字符，如果不足在右边补0

%logger 输出logger名称，

%msg 日志文本

%n 换行

其他常用的占位符有：

%F 输出所在的类文件名，如Client.java

%L 输出行号

%M 输出所在方法名

%l  输出语句所在的行数, 包括类名、方法名、文件名、行数

最后是Logger的配置，这里只配置了一个Root Logger。

### 三、配置全解

1.关于配置文件的名称以及在项目中的存放位置

> log4j 2.x版本不再支持像1.x中的.properties后缀的文件配置方式，2.x版本配置文件后缀名只能为".xml",".json"或者".jsn".

系统选择配置文件的优先级(从先到后)如下：

   (1).classpath下的名为log4j2-test.json 或者log4j2-test.jsn的文件.

   (2).classpath下的名为log4j2-test.xml的文件.

   (3).classpath下名为log4j2.json 或者log4j2.jsn的文件.

   (4).classpath下名为log4j2.xml的文件.

我们一般默认使用log4j2.xml进行命名。如果本地要测试，可以把log4j2-test.xml放到classpath，而正式环境使用log4j2.xml，则在打包部署的时候不要打包log4j2-test.xml即可。

2.缺省默认配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--status属性，这个属性表示log4j2本身的日志信息打印级别。
如果把status改为TRACE再执行测试代码，可以看到控制台中打印了一些log4j加载插件、组装logger等调试信息-->
<Configuration status="WARN">
    <!--Appender可以理解为日志的输出目的地，这里配置了一个类型为Console的Appender，也就是输出到控制台-->
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!--PatternLayout定义了输出日志时的格式-->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

3.配置文件节点解析

(1).Configuration根节点，有两个属性，两个子节点：Appenders和Loggers(表明可以定义多个Appender和Logger).

- status用来指定log4j本身的打印日志的级别.
- monitorinterval用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s.

(2).Appenders节点，充分考虑了日志事件的输出、包装以及过滤转发的可能，包括最基本的输出到本地文件、输出到远程主机，对文件进行封装、注入， 并且还能按照日志文件的时间点、文件大小等条件迸行白动封存。它常用的三种子节点：Console，RollingFile、File。

- **Console**节点：用来定义输出到控制台的Appender.

  - name：指定Appender的名字.

  - target：SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.

  - PatternLayout：定义了输出日志时的格式

- **File**节点：用来定义输出到指定位置的文件的Appender

  - name:指定Appender的名字.
  - fileName:指定输出日志的目的文件带全路径的文件名.
  - PatternLayout:输出格式，不设置默认为:%m%n.

- 　**RollingFile**节点：用来定义超过指定大小自动删除旧的创建新的的Appender.

  - name:指定Appender的名字
  - fileName:指定输出日志的目的文件带全路径的文件名
  - PatternLayout:输出格式，不设置默认为:%m%n.
  - filePattern:指定新建日志文件的名称格式
  - Policies:指定滚动日志的策略，就是什么时候进行新建日志文件输出日志
  - TimeBasedTriggeringPolicy：Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动1次，默认1hour。modulate=true用来调整时间，比如现在是下午七点，interval是4，name第一次滚动是在下午八点，接着是凌晨十二点，午夜四点，而不是一开始就是十一点
  - SizeBasedTriggeringPolicy：Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小
  - DefaultRolloverStrategy：用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的日志文件，通过max属性。不设置max属性，是默认7个文件

(3).Loggers节点，常见的有两种:Root和Logger.

- **Root**节点：用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出
  - level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
  - AppenderRef：Root的子节点，用来指定该日志输出到哪个Appender.‘
- 　**Logger**节点：用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等
  - level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF。
  - name:用来指定该Logger所适用的类或者类所在的包全路径,继承自Root节点.
  - AppenderRef：**用来指定该日志输出到哪个Appender,如果没有指定，就会默认继承自Root。如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出。**

4.比较完整的log4j2.xml配置模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration status="WARN" monitorInterval="30">
    <!--先定义所有的appender-->
    <appenders>
        <!--这个输出控制台的配置-->
        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式-->
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
        </console>
        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定(false)，这个也挺有用的，适合临时测试用-->
        <File name="log" fileName="logs/data.log" append="true">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>
        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <!--获取用户HOME目录的占位符是“${sys:user.home}”。"${sys:user.home}/logs/info.log"在windows中就是"C:\Users\huanghe\logs\info.log"-->
        <!--filePattern：指定当发生Rolling时，文件的转移和重命名规则：当文件的大小大于10KB的时候就会，新建info-%d{yyyy-MM-dd}-%i.log-->
        <RollingFile name="RollingFileInfo" fileName="${sys:user.home}/logs/info.log"
                     filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="10 KB"/>
            </Policies>
        </RollingFile>
        <RollingFile name="RollingFileWarn" fileName="${sys:user.home}/logs/warn.log"
                     filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <!--DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20-->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
        <RollingFile name="RollingFileError" fileName="${sys:user.home}/logs/error.log"
                     filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
        <logger name="org.springframework" level="INFO"></logger>
        <logger name="org.mybatis" level="INFO"></logger>
        <Logger name="com.selenium.log4j2.demo_log4j2" level="trace"> <!--  additivity="false" -->
            <appender-ref ref="log" />
        </Logger>
        <root level="all">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>
    </loggers>
</configuration>
```









