# MyBatis-实体类属性和表字段的映射

之前的章节将的实体类属性名和表字段名都是相同的，MyBatis 会自动去映射。那么问题来了，如果实体类属性名和表字段名不相同时，MyBatis 能智能地去映射到吗？答案是：**不能**。这里用两种解决方案： 

1. 在使用 SQL 语句的时候，为每个字段定义别名； 
2. 使用 MyBatis 映射文件的 resultMap 标签。

## 使用别名

使用别名这个很容易理解，因为日常在写 SQL 语句时，通常会用到别名。如：

```sql
SELECT Count(*) AS total
FROM order
```
而这里说的别名和 SQL 语句的别名是一回事，只不过每个字段的别名必须是其对应的实体类属性名。

```sql
<select id="selectWithAlias" parameterType="int" resultType="Order">
    select order_id id, order_no orderNo, order_price price
    from orders
    where order_id = #{id}
</select>
```

可能会有人问，如果我不使用别名会怎么样？如果不使用别名，那么 MyBatis 找不到映射的属性，导致最终的结果返回 **null**。

# 使用 resultMap

另一种 MyBatis 的方式是在映射文件中使用 **resultMap** 标签。这种方式是使用MyBatis提供的解决方式来解决字段名和属性名的映射关系的。在 **resultMap** 中 id 和 result 有几个属性需要了解下：

| 属性          | 说明                                       |
| ----------- | ---------------------------------------- |
| column      | 数据库表的列名                                  |
| property    | 需要映射到JavaBean 的属性名称                      |
| javaType    | 一个完整的类名，或者是一个类型名。如果你匹配的是一个JavaBean，那 MyBatis 通常会自行检测到。如果你是要映射到一个 HashMap，那你需要指定 javaType 要达到的目的。 |
| jdbcType    | 数据库表支持的类型。这个属性只在 insert,update 或delete 的时候针对允许空的列有用。JDBC 需要这项，但 MyBatis 不需要。如果你是直接针对 JDBC 编码，且有允许空的列，而你要指定这项。 |
| typeHandler | 使用这个属性可以覆写类型处理器。这项值可以是一个完整的类名，也可以是一个类型别名。 |

```xml
<select id="selectWithMapping" parameterType="int" resultMap="OrderMapping">
    select order_id, order_no, order_price
    from orders
    where order_id = #{id}
</select>
<!-- 通过<resultMap>映射实体类属性名和表的字段名对应关系 -->
<resultMap id="OrderMapping" type="Order">
    <!-- id属性来映射主键字段 -->
    <id column="order_id" property="id" javaType="int"/>
    <!-- result属性映射非主键字段 -->
    <result column="order_no" property="orderNo" javaType="String"/>
    <result column="order_price" property="price"/>
</resultMap>
```