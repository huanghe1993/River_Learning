# MyBatis的查询

[TOC]

## 1、mybatis的查询属性说明

```xml
<select 
        id="selectUser"
        parameterType="int"
        resultType="hashmap"
        resultMap="userResultMap"
        flushCache="true" //如果为true，则任何的本地缓存和二级缓存都是会被清空的
        timeout="10000" //驱动程序等待数据库返回请求结果的的秒数
        statementType:"PREPARED" //选择使用JDBC的Satement,PreparedStatement或CallableStatement></select>
```

## 2、最简单的mybatis的查询

使用的是resultType作为返回类型，parameterType是接收参数的类型

```xml
<!-- 根据用户ID，查询用户信息 -->
    <!--
        select：表示的是MappedStatement对象
        [id]：statement的id，要求在命名空间内唯一
        [parameterType]：入参的java类型
        [resultType]：查询出的单条结果集对应的java类型
        [#{}]： 表示一个占位符?
        [#{id}]：表示该占位符待接收参数的名称为id。注意：如果参数为简单类型时，#{}里面的参数名称可以是任意定义
     -->
    <select id="findUserById" parameterType="int" resultType="com.itheima.mybatis.po.User">
        SELECT * FROM USER WHERE id = #{id}
    </select>
```

第二种使用的是模糊查询：

```xml
<!--根据用户名称模糊的查询用户列表-->
    <!--使用${}可以吧需要的调进来，其余的补上
    ${}：表示的是一个sql的连接符
    ${value}：里面的value表示输入参数的参数名称，如果该参数是简单类型，那么${}里面的参数名称必须是value
    ${}：这种写法存在sql注入的风险，所以要慎用！！但是在一些场景下，必须使用${}，比如排序时，动态传入排序的列名，${}会原样输出，不加解释,如果是用#{}里面放的是String类型的话，会自动的加上‘’，sql会出错-->
    <select id="findUsersByName" parameterType="java.lang.String" resultType="com.itheima.mybatis.po.User">
        SELECT * from user where username like '%${value}%'
    </select>
```

## 3、关于parameterType="map" 的使用

请求参数就是你serviceImpl参数里传入的map

先看看select的sql语句：

```xml
<select id="findUserByHashmap" parameterType="hashmap" resultType="user">
        select * from user where username LIKE "%${user.username}%" AND sex=#{user.sex}
 </select>

```
这里我们可以看到的是parameterType="hashmap"，所以下面传进来的也是map；

下面的是测试代码：

```java
 @Test
    public void testFindUserByHashmap()throws Exception{
        //获取session
        SqlSession session = sqlSessionFactory.openSession();
        //获限mapper接口实例
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //构造查询条件Hashmap对象
        HashMap<String, Object> map = new HashMap<String, Object>();
        map.put("id", 1);
        map.put("username", "管理员");
        //传递Hashmap对象查询用户列表
        List<User>list = userMapper.findUserByHashmap(map);
        //关闭session
        session.close();
    }
```



## 4、关于ResultType="map"的使用

### 4.1、在使用mybatis的esultType="map"之前

我们先看看map类型在pringMVC中的作用：

springmvc形参`(@RequestBody HashMap hashMap)`,即请求过来的json就封装到了hashMap里,这样可以省去QueryVo,直接调用service接口传入`hashMap.get("id"),hashMap.get("name"))`

但通常还是将请求过来的json通过@RequestBody注解转为实体类(或者用`hashmap<String,String> hashmap`来接收),然后调用service层(此时传入对象或hashmap)

### 4.2、使用resultType="map"最简单的情形

 **==返回值map的键就是表中字段的别名,值就是是你查到的值,如果是多个,mapper接口就是list包map,即 `List<Map<String, Object>>`,如此可省去new返回值对象,很简单==**



先看看返回多条数据的情形：

- xml文件如下：

```xml
<select id="selectUser" resultType="map">
	SELECT * FRMO TB_USER
</select>	
```

这里返回的是多个user对象，resultType="map"表示返回的是一个Map集合（使用的是列名作为key，列值作为value），这种情况下在写mapper接口的时候可以使用

- dao层的mapper接口

```java
List<Map<String,Object>> selectUser(); //mapper接口
```

其中String表示的是key的类型，而用Object可以接受任意的返回的类型，

- service层进行封装

然后在service层中使用UserMsg,这样做的目的主要是:如果需要返回的结果中包含着User类中没有的属性就可以使用自定义的类型进行封装。

```java
List<Map<String,Object>> userList = userMapper.selectUser();
for (Map<String,Object> map:courseChoiceList){
            UserMsg userMsg=new  UserMsg();
            userMsg.setId((Long)map.get("id"));
            userMsg.setName(map.get("course_id").toString());
            userMsg.setBirthday(new Date());
            courseChoiceMsgList.add(courseChoiceMsg);
        }
```

### 4.3、返回`Map<String, Map<String, Object>>`这种类型

需求场景：

批量从数据库查出若干条数据，包括id和name两个字段。希望可以把结果直接用Map接收，然后通过map.get(id)方便地获取name的值。

```java
{1={id=1,name="zhangsan"},2={id=2,name="lisi"},3={id=3,name="wangwu"}}
```

解决的方法是在外面再用一个Map：

```java
Map<Integer, Map<String, Object>> m = abcDao.getNamesByIds(idList);  
```

然后，在这个dao的方法上面加一个注解：

```java
@MapKey("id")  
public Map<Integer, Map<String, Object>> getNamesByIds(List<Map<String, Objec>)
```

Mapper.xml中的配置如下：

```xml
<select id="getNamesByIds" resultType="java.util.Map">  
    SELECT id, name FROM tb_abc WHERE id IN  
    <foreach item="item" collection="list" open="(" separator="," close=")">  
            #{item.id}  
    </foreach>  
</select>  
```

## 5、动态sql进行查询

### 5.1、foreach查询

需求场景：

​	需要通过传过来的一个id数组，查询出对应的用户

- foreach元素的属性主要有 item，index，collection，open，separator，close。
  - item表示集合中每一个元素进行迭代时的别名，
  - index指定一个名字，用于表示在迭代过程中，每次迭代到的位置，
  - open表示该语句以什么开始，
  - separator表示在每次进行迭代之间以什么符号作为分隔符，
  - close表示以什么结束。
- 在使用foreach的时候最关键的也是最容易出错的就是collection属性，该属性是必须指定的，但是在不同情况下，该属性的值是不一样的，主要有一下3种情况：
  - 1**.如果传入的是单参数且参数类型是一个List的时候，collection属性值为list**
  - 2.**如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array**
  - 3.**如果传入的参数是多个的时候，我们就需要把它们封装成一个Map(Object)，当然单参数也可以封装成map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key.**

```java
public class Employees {
    private Integer employeeId;
    private String firstName;
    private String lastName;
    private String email;
    private String phoneNumber;
    private Date hireDate;
    private String jobId;
    private BigDecimal salary;
    private BigDecimal commissionPct;
    private Integer managerId;
    private Short departmentId;
}  
```

#### 5.1.1、传入的是单参数且参数类型是一个List的时候

```xml
<!--List:forech中的collection属性类型是List,collection的值必须是:list,item的值可以随意,Dao接口中参数名字随意 -->
    <select id="getEmployeesListParams" resultType="Employees">
        select *
        from EMPLOYEES e
        where e.EMPLOYEE_ID in
        <foreach collection="list" item="employeeId" index="index"
            open="(" close=")" separator=",">
            #{employeeId}
        </foreach>
    </select>
```

#### 5.1.2、传入的是单参数且参数类型是一个array数组的时候

```xml
 <!--Array:forech中的collection属性类型是array,collection的值必须是:list,item的值可以随意,Dao接口中参数名字随意 -->
    <select id="getEmployeesArrayParams" resultType="Employees">
        select *
        from EMPLOYEES e
        where e.EMPLOYEE_ID in
        <foreach collection="array" item="employeeId" index="index"
            open="(" close=")" separator=",">
            #{employeeId}
        </foreach>
    </select>
```

#### 5.1.3、传入的参数是多个的时候

注意：这里传入的是Object类型的也是可以的

**第一种：map**

```xml
<!--Map:不单单forech中的collection属性是map.key,其它所有属性都是map.key,比如下面的departmentId -->
    <select id="getEmployeesMapParams" resultType="Employees">
        select *
        from EMPLOYEES e
        <where>
            <if test="departmentId!=null and departmentId!=''">
                e.DEPARTMENT_ID=#{departmentId}
            </if>
            <if test="employeeIdsArray!=null and employeeIdsArray.length!=0">
                AND e.EMPLOYEE_ID in
                <foreach collection="employeeIdsArray" item="employeeId"
                    index="index" open="(" close=")" separator=",">
                    #{employeeId}
                </foreach>
            </if>
        </where>
    </select>
```

**第二种：pojo**  

 传入的是pojo的类型`parameterType="com.itheima.mybatis.po.UserQueryVO"`

```xml
 <!--定义sql片段-->
    <!--sql片段内最好不要将where和select声明在内-->
    <sql id="whereClause">
        <!-- if标签可以对输入的参数进行判断-->
        <!-- test:判断表达式-->
        <!-- if:防止出现空指针异常的出现-->
        <if test="user!=null">
            <if test="user.username!=null and user.username!=''">
                AND username LIKE "%${user.username}%"
            </if>
            <if test="user.sex!=null and user.sex!=''">
                AND sex=#{user.sex}
            </if>
        </if>
        <if test="idList!=null">
            AND id IN
            <foreach collection="idList" item="id" open="(" close=")" separator=",">#{id}</foreach>
        </if>
    </sql>
    <!--综合查询，查询用户列表(根据用户名，性别，用户id；SELECT * FROM USER WHERE id IN (1,2,10))-->
    <select id="findUserList" parameterType="com.itheima.mybatis.po.UserQueryVO" resultType="user">
        SELECT * FROM user
        <!--where标签：默认的去掉后面的第一个AND符号，如果是没有参数，则把自己干掉-->
        <where>
           <include refid="whereClause"/>
        </where>
```

#### 5.1.4、对应的mapper接口

```java
Mapper类:
public interface EmployeesMapper { 

    List<Employees> getEmployeesListParams(List<String> employeeIds);

    List<Employees> getEmployeesArrayParams(String[] employeeIds);

    List<Employees> getEmployeesMapParams(Map<String,Object> params);
}
```

#### 5.1.5、测试方法

```java
@Test 
    public void testGetEmployeesListParams() {
        List<String> employeeIds = Arrays.asList("100", "101", "200");
        List<Employees> result = employeesMapper
                .getEmployeesListParams(employeeIds);
        assertEquals(3, result.size());
    }

    @Test
    public void testGetEmployeesArrayParams() {
        String[] employeeIds = new String[] { "100", "200" };
        List<Employees> result = employeesMapper
                .getEmployeesArrayParams(employeeIds);
        assertEquals(2, result.size());
    }

    @Test
    public void testGetEmployeesMapParams() {
        String departmentId = "60";
        List<String> employeeIdsList = Arrays.asList("103", "104", "105");
        String[] employeeIdsArray = new String[] { "103", "104" };
        // 传入多个参数
        Map<String, Object> params = new HashMap<String, Object>();
        params.put("departmentId", departmentId);
        params.put("employeeIdsList", employeeIdsList);
        params.put("employeeIdsArray", employeeIdsArray);
        List<Employees> result = employeesMapper.getEmployeesMapParams(params);
        assertEquals(3, result.size());
    }
```

### 5.2 if标签的一些问题

1、常见错误：

**There is no getter for property named 'parentId' in 'class java.lang.Long'（或者String）**

org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'parentId' in 'class java.lang.Long'

sql是这样写的：

```xml
<select id="queryByParentId" parameterType="long" resultMap="beanMap">
        SELECT * FROM <include refid="t_pt_category"/>
        <where>
            isdel = 0
            <if test="parentId != null">
                AND parent_id = #{parentId}
            </if>
        </where>
        ORDER BY parent_id
    </select>
```

解决1：

把`<if>`标签去掉传参还是直接传递传递long，如下：sql2

```xml
<select id="queryByParentId" parameterType="long" resultMap="beanMap">
        SELECT * FROM <include refid="t_pt_category"/>
        <where>
            isdel = 0
           AND parent_id = #{parentId}
            <!--<if test="parentId != null">
                AND parent_id = #{parentId}
            </if> -->
        </where>
        ORDER BY parent_id
 </select>  
```

**解决2：**保持sql1不变，在dao层把参数换成map类型，

```java
Map<String, Object> params = new HashMap<>(1);
params.put("parentId", parentId);
```

解决3：改写sql1，如下，全部的参数都换成"_parameter",如下sql3：

```xml
select id="queryByParentId" parameterType="long" resultMap="beanMap">
        SELECT * FROM <include refid="t_pt_category"/>
        <where>
            isdel = 0
            <if test="_parameter != null">
                AND parent_id = #{_parameter}
            </if>
        </where>
        ORDER BY parent_id
    </select>
```