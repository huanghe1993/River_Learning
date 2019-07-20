# Spring-Data-jpa

　　1、随着Hibernate的盛行，Hibernate主导了EJB3.0的JPA规范，JPA即使Java Persistence API(java的持久化API接口)，JPA是一个基于O/R映射的标准规范。所谓的规范即只定义标准规则（如注解和接口），不是提供实现的，软件提供商可以按照标准规范来实现，而使用者值需要按照规范中定义的方式来使用，而不是和软件提供商的实现进行打交道。JPA的主要的实现有Hibernate、EclipseLink和OpenJDK,这也意味着我们只是需要使用JPA来开发，无论使用那一个开发方式都是一样的。Hibernate是和JPA整合的比较良好，我们可以认为**JPA是标准，事实上也是，JPA几乎都是接口，实现都是Hibernate在做**，宏观上面看，在JPA的统一之下Hibernate很良好的运行。

　　上面阐述了JPA和Hibernate的关系，那么Spring-data-jpa又是个什么东西呢？这地方需要稍微解释一下，我们做Java开发的都知道Spring的强大，到目前为止，企业级应用Spring几乎是无所不能，无所不在，已经是事实上的标准了，企业级应用不使用Spring的几乎没有，这样说没错吧。而Spring整合第三方框架的能力又很强，他要做的不仅仅是个最早的IOC容器这么简单一回事，现在Spring涉及的方面太广，主要是体现在和第三方工具的整合上。而在与第三方整合这方面，Spring做了持久化这一块的工作，我个人的感觉是Spring希望把持久化这块内容也拿下。于是就有了Spring-data-**这一系列包。包括，Spring-data-jpa,Spring-data-template,Spring-data-mongodb,Spring-data-redis，还有个民间产品，mybatis-spring，和前面类似，这是和mybatis整合的第三方包，这些都是干的持久化工具干的事儿。

　　这里介绍Spring-data-jpa，表示与jpa的整合。

　　２、我们都知道，在使用持久化工具的时候，一般都有一个对象来操作数据库，**在原生的Hibernate中叫做Session，在JPA中叫做EntityManager，在MyBatis中叫做SqlSession**，通过这个对象来操作数据库。我们一般按照三层结构来看的话，Service层做业务逻辑处理，Dao层和数据库打交道，在Dao中，就存在着上面的对象。那么ORM框架本身提供的功能有什么呢？答案是基本的CRUD，所有的基础CRUD框架都提供，我们使用起来感觉很方便，很给力，业务逻辑层面的处理ORM是没有提供的，如果使用原生的框架，业务逻辑代码我们一般会自定义，会自己去写SQL语句，然后执行。在这个时候，Spring-data-jpa的威力就体现出来了，ORM提供的能力他都提供，ORM框架没有提供的业务逻辑功能Spring-data-jpa也提供，全方位的解决用户的需求。使用Spring-data-jpa进行开发的过程中，常用的功能，我们几乎不需要写一条sql语句，至少在我看来，企业级应用基本上可以不用写任何一条sql，当然spring-data-jpa也提供自己写sql的方式，这个就看个人怎么选择，都可以。我觉得都行。

![1540173437226](../../../../AppData/Roaming/Typora/typora-user-images/1540173437226.png)

## 2、整合SpringDataJpa

JPA:ORM（Object Relational Mapping）；

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
package com.huanghe.entity;

import javax.persistence.*;

/**
 * @Author: River
 * @Date:Created in  10:14 2018/10/22
 * @Description: 配置关系使用JPA注解
 *
 * @Entity :告诉JPA这是一个实体类（和数据表映射的类）
 * @Table :指定和哪个数据表相关联，还可以指定规则比如说索引，如果省略默认是类名小写
 */
@Entity
@Table(name = "tbl_user")
public class User {
    /**
     * @Id :表明这是一个主键
     * @GeneratedValue(strategy = GenerationType.IDENTITY) ：是主键策略，这里默认的是自增的策略
     * @Column :标识和数据表对应的一个列,name可以用来指定数据表列的名称，length标识数据的长度；默认的列名就是属性的名称
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "last_name",length = 50)
    private String lastName;

    @Column
    private String email;

}
```

(2):编写Dao接口来操作实体类对应的数据表（Repository）

```java
package com.huanghe.repository;

import com.huanghe.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @Author: River
 * @Date:Created in  10:24 2018/10/22
 * @Description: 继承JpaRepository完成对数据库的操作
 *
 * JpaRepository<User,>:继承JpaRepository，JpaRepository有CRUD的功能，也有分页和排序的功能；
 * 它有两个泛型，第一个需要传入的是需要操作哪个实体类，第二个是实体类的主键的类型
 */
public interface UserRepository extends JpaRepository<User, Integer> {



}

```

(3）：基本的配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1/jpa
    username: root
    password: ab2453282
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
#     更新或者创建数据表结构
      ddl-auto: update
#    控制台显示SQL
    show-sql: true
```



















