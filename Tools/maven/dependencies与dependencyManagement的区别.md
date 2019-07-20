# Maven实战（六）--- dependencies与dependencyManagement的区别

[TOC]



 ## 1、DepencyManagement应用场景

考虑这样一个不大不小的项目，它有 10 多个 Maven 模块，这些模块分工明确，各司其职，相互之间耦合度比较小，这样大家就能够专注在自己的模块中进行开发而不用过多考虑他人对自己的影响。（好吧，我承认这是比较理想的情况）那我开始对模块 A 进行编码了，首先就需要引入一些常见的依赖如 JUnit、Log4j 等等：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactid>junit</artifactId>
    <version>4.8.2</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>log4j</groupId>
    <artifactid>log4j</artifactId>
    <version>1.2.16</version>
  </dependency>
```

我的同事在开发模块 B，他也要用 JUnit 和 Log4j（我们开会讨论过了，统一单元测试框架为 JUnit 而不是 TestNG，统一日志实现为 Log4j 而不是 JUL，为什么做这个决定就不解释了，总之就这么定了）。同事就写了如下依赖配置：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactid>junit</artifactId>
    <version>3.8.2</version>
  </dependency>
  <dependency>
    <groupId>log4j</groupId>
    <artifactid>log4j</artifactId>
    <version>1.2.9</version>
  </dependency>
```

　　看出什么问题来没有？对的，他漏了 JUnit 依赖的 scope，那是因为他不熟悉 Maven。还有什么问题？对，版本！虽然他和我一样都依赖了 JUnit 及 Log4j，但版本不一致啊。我们开会讨论没有细化到具体用什么版本，但如果一个项目同时依赖某个类库的多个版本，那是十分危险的！OK，现在只是两个模块的两个依赖，手动修复一下没什么问题，但如果是 10 个模块，每个模块 10 个依赖或者更多呢？看来这真是一个泥潭，一旦陷进去就难以收拾了。 

　　好在 Maven 提供了优雅的解决办法，使用继承机制以及 dependencyManagement 元素就能解决这个问题。注意，是 dependencyMananget 而非 dependencies。也许你已经想到在父模块中配置 dependencies，那样所有子模块都自动继承，不仅达到了依赖一致的目的，还省掉了大段代码，但这么做是有问题的，例如你将模块 C 的依赖 spring-aop 提取到了父模块中，但模块 A 和 B 虽然不需要 spring-aop，但也直接继承了。dependencyManagement 就没有这样的问题，**dependencyManagement 只会影响现有依赖的配置，但不会引入依赖**。例如我们可以在父模块中配置如下：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactid>junit</artifactId>
      <version>4.8.2</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactid>log4j</artifactId>
      <version>1.2.16</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

这段配置不会给任何子模块引入依赖，但如果某个子模块需要使用 JUnit 和 Log4j 的时候，我们就可以简化依赖配置成这样：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactid>junit</artifactId>
  </dependency>
  <dependency>
    <groupId>log4j</groupId>
    <artifactid>log4j</artifactId>
  </dependency>
```

　　现在只需要 groupId 和 artifactId，其它元素如 version 和 scope 都能通过继承父 POM 的 dependencyManagement 得到，如果有依赖配置了 exclusions，那节省的代码就更加可观。但重点不在这，重点在于现在能够保证所有模块使用的 JUnit 和 Log4j 依赖配置是一致的。而且子模块仍然可以按需引入依赖，如果我不配置 dependency，父模块中 dependencyManagement 下的 spring-aop 依赖不会对我产生任何影响。 

　　也许你已经意识到了，**在多模块 Maven 项目中，dependencyManagement 几乎是必不可少的，因为只有它是才能够有效地帮我们维护依赖一致性**。 

　　本来关于 dependencyManagement 我想介绍的也差不多了，但几天前和[Sunng](http://sunng.info/blog/)的一次讨论让我有了更多的内容分享。那就是在使用 dependencyManagement 的时候，我们可以不从父模块继承，而是使用特殊的 import scope 依赖。Sunng 将其列为自己的[Maven Recipe #0](http://sunng.info/blog/2010/05/maven-recipe-0/)，我再简单介绍下。

　　我们知道 Maven 的继承和 Java 的继承一样，是无法实现多重继承的，如果 10 个、20 个甚至更多模块继承自同一个模块，那么按照我们之前的做法，这个父模块的 dependencyManagement 会包含大量的依赖。如果你想把这些依赖分类以更清晰的管理，那就不可能了，import scope 依赖能解决这个问题。你可以把 dependencyManagement 放到单独的专门用来管理依赖的 POM 中，然后在需要使用依赖的模块中通过 import scope 依赖，就可以引入 dependencyManagement。例如可以写这样一个用于依赖管理的 POM：‘

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.juvenxu.sample</groupId>
  <artifactId>sample-dependency-infrastructure</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
  <dependencyManagement>
    <dependencies>
        <dependency>
          <groupId>junit</groupId>
          <artifactid>junit</artifactId>
          <version>4.8.2</version>
          <scope>test</scope>
        </dependency>
        <dependency>
          <groupId>log4j</groupId>
          <artifactid>log4j</artifactId>
          <version>1.2.16</version>
        </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```

然后我就可以通过非继承的方式来引入这段依赖管理配置：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
          <groupId>com.juvenxu.sample</groupId>
          <artifactid>sample-dependency-infrastructure</artifactId>
          <version>1.0-SNAPSHOT</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
    </dependencies>
  </dependencyManagement>

  <dependency>
    <groupId>junit</groupId>
    <artifactid>junit</artifactId>
  </dependency>
  <dependency>
    <groupId>log4j</groupId>
    <artifactid>log4j</artifactId>
  </dependency>
```

这样，父模块的 POM 就会非常干净，由专门的 packaging 为 pom 的 POM 来管理依赖，也契合的面向对象设计中的单一职责原则。此外，我们还能够创建多个这样的依赖管理 POM，以更细化的方式管理依赖。这种做法与面向对象设计中使用组合而非继承也有点相似的味道。

