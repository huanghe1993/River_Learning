# SpringBoot与数据访问

[TOC]

对于数据访问层，无论是SQL还是NOSQL，Spring Boot默认采用整合Spring Data的方式进行统一处理，添加大量自动配置，屏蔽了很多设置。引入各种xxxTemplate，xxxRepository来简化我们对数据访问层的操作。对我们来说只需要进行简单的设置即可。

## 整合基本JDBC与数据源 

```xml
       <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
```

application.yml

```yml
spring:
  datasource:
    username: root
    password: ab2453282
    url: jdbc:mysql://localhost:3306/springboot_jdbc_06
    driver-class-name: com.mysql.jdbc.Driver
```

默认使用的是`org.apache.tomcat.jdbc.pool.DataSource`作为数据源的，数据源的所有的配置都在DataSourceProperties类里面

## 数据源的自动配置的原理

在autoConfig的包下面查看jdbc的配置

![1540345700409](../../../../AppData/Roaming/Typora/typora-user-images/1540345700409.png)

先看一下DataSourceConfiguration

```java
abstract class DataSourceConfiguration {

	@SuppressWarnings("unchecked")
	protected static <T> T createDataSource(DataSourceProperties properties,
			Class<? extends DataSource> type) {
		return (T) properties.initializeDataSourceBuilder().type(type).build();
	}

	/**
	 * Tomcat Pool DataSource configuration.
	 * @ConditionalOnClass：如果路径中有tomcat jdbc 的数据源,就会先配置tomcat的数据源
	 * @ConditionalOnMissingBean:并且路径中没有其他的数据源
	 * @ConditionalOnProperty:控制这个configuration是否生效；具体操作是通过其两个属性      * name以及havingValue来实现的，其中name用来从application.properties中读取某个属性      * 值，如果该值为空，则返回false;如果值不为空，则将该值与havingValue指定的值进行比较，      * 如果一样则返回true;否则返回false。如果返回值为false，则该configuration不生效；为      * true则生效！matchIfMissing()该属性为true时，配置文件中缺少对应的value或name的对      * 应的属性值，也会注入成功
	 */
	@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource", matchIfMissing = true)
	static class Tomcat {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.tomcat")
		public org.apache.tomcat.jdbc.pool.DataSource dataSource(
				DataSourceProperties properties) {
			org.apache.tomcat.jdbc.pool.DataSource dataSource = createDataSource(
					properties, org.apache.tomcat.jdbc.pool.DataSource.class);
			DatabaseDriver databaseDriver = DatabaseDriver
					.fromJdbcUrl(properties.determineUrl());
			String validationQuery = databaseDriver.getValidationQuery();
			if (validationQuery != null) {
				dataSource.setTestOnBorrow(true);
				dataSource.setValidationQuery(validationQuery);
			}
			return dataSource;
		}

	}

	/**
	 * Hikari DataSource configuration.
	 */
	@ConditionalOnClass(HikariDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource", matchIfMissing = true)
	static class Hikari {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.hikari")
		public HikariDataSource dataSource(DataSourceProperties properties) {
			return createDataSource(properties, HikariDataSource.class);
		}

	}

	/**
	 * DBCP DataSource configuration.
	 *
	 * @deprecated as of 1.5 in favor of DBCP2
	 */
	@ConditionalOnClass(org.apache.commons.dbcp.BasicDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.commons.dbcp.BasicDataSource", matchIfMissing = true)
	@Deprecated
	static class Dbcp {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.dbcp")
		public org.apache.commons.dbcp.BasicDataSource dataSource(
				DataSourceProperties properties) {
			org.apache.commons.dbcp.BasicDataSource dataSource = createDataSource(
					properties, org.apache.commons.dbcp.BasicDataSource.class);
			DatabaseDriver databaseDriver = DatabaseDriver
					.fromJdbcUrl(properties.determineUrl());
			String validationQuery = databaseDriver.getValidationQuery();
			if (validationQuery != null) {
				dataSource.setTestOnBorrow(true);
				dataSource.setValidationQuery(validationQuery);
			}
			return dataSource;
		}

	}

	/**
	 * DBCP DataSource configuration.
	 */
	@ConditionalOnClass(org.apache.commons.dbcp2.BasicDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.commons.dbcp2.BasicDataSource", matchIfMissing = true)
	static class Dbcp2 {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.dbcp2")
		public org.apache.commons.dbcp2.BasicDataSource dataSource(
				DataSourceProperties properties) {
			return createDataSource(properties,
					org.apache.commons.dbcp2.BasicDataSource.class);
		}

	}

	/**
	 * Generic DataSource configuration.
	 
	 */
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type")
	static class Generic {

		@Bean
		public DataSource dataSource(DataSourceProperties properties) {
            //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
			return properties.initializeDataSourceBuilder().build();
		}

	}

}
```

1、可以使用`spring.datasource.type`来指定自定义的数据源的类型

2、SprinBoot默认支持的数据源有：`org.apache.tomcat.jdbc.pool.DataSource、HikariDataSource、BasicDataSource、`

3、自定义的数据源类型

4、**DataSourceInitializer：ApplicationListener**；

​	作用：

​		1）、runSchemaScripts();运行建表语句；

​		2）、runDataScripts();运行插入数据的sql语句；

默认只需要将文件命名为：

```properties
schema-*.sql、data-*.sql
默认规则：schema.sql，schema-all.sql；
可以使用   
	schema:
      - classpath:department.sql
      指定位置
```

5、自动的配置JDBCTemplate操作数据库

## SpringBoot配合druid

```yml
spring:
  datasource:
    username: root
    password: ab2453282
    url: jdbc:mysql://localhost:3306/springboot_jdbc_06
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

```

配置文件：

```java
package com.huanghe.springboot.config.druid;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author: River
 * @Date:Created in  11:11 2018/10/24
 * @Description:
 */
@Configuration
public class config {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druid() {
        return new DruidDataSource();
    }

    //配置Druid监控

    /**
     * 1:配置管理后台的servlet
     * 使用嵌入式的servlet容器
     * @return
     */
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>(10);
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        //默认就是允许所有访问
        initParams.put("allow", "");
        initParams.put("deny", "192.168.15.21");
        bean.setInitParameters(initParams);
        return bean;
    }

    /**
     * 2:配置监控的filter
     * @return
     */
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        //不拦截以下的一些请求
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return  bean;
    }
}
```

## SpringBoot和MyBatis

工程idea spring-boot-06-data-mybatis

配置数据源的配置和上面一样

### 注解版本的使用

不需要做任何的配置

```java
package com.huanghe.springboot.mapper;

import com.huanghe.springboot.bean.Department;
import org.apache.ibatis.annotations.*;

/**
 * @Author: River
 * @Date:Created in  11:56 2018/10/24
 * @Description:
 */

/**
 * @Mapper :指定这个是操作数据库的mapper
 */
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);

}

```

开启驼峰命名法

> 可以在配置文件中使用mybatis.configuration.map-underscore-to-camel-case=true就行，下面的代码就可以省略掉

```java
@org.springframework.context.annotation.Configuration
public class MybatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){

            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

```java
使用MapperScan批量扫描所有的Mapper接口；
@MapperScan(value = "com.atguigu.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
	}
}
```

### 配置文件的方式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.huanghe.springboot.mapper.EmployeeMapper">
    <select id="getEmpById" resultType="com.huanghe.springboot.bean.Employee" parameterType="int">
        select * from employee where id = #{id}
    </select>

    <!--public void inserEmp(Employee employee);-->
    <insert id="inserEmp" parameterType="com.huanghe.springboot.bean.Employee">
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            SELECT Last_insert_id()
        </selectKey>
        INSERT INTO employee
        (lastName,gender,email,dId)
        VALUES(#{lastName},#{gender},#{email},#{dId})         
    </insert>

</mapper>
```

mybatis的全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

配置文件的application.yml

```yml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml  指定sql映射文件的位置
```

项目的结构文件图：

![1540369561017](../../../../AppData/Roaming/Typora/typora-user-images/1540369561017.png)\

注解版本的和配置文件版本的是可以同时的进行使用的



























































































































































































