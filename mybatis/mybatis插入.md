# mybatis插入

## insert和insertSelective的区别

- insert的代码是：

```xml
<insert id="insert" parameterType="com.taotao.pojo.TbContentCategory" >
  	<selectKey keyProperty="id" resultType="long" order="AFTER">
  		SELECT LAST_INSERT_ID()
  	</selectKey>
    insert into tb_content_category (id, parent_id, name, 
      status, sort_order, is_parent, 
      created, updated)
    values (#{id,jdbcType=BIGINT}, #{parentId,jdbcType=BIGINT}, #{name,jdbcType=VARCHAR}, 
      #{status,jdbcType=INTEGER}, #{sortOrder,jdbcType=INTEGER}, #{isParent,jdbcType=BIT}, 
      #{created,jdbcType=TIMESTAMP}, #{updated,jdbcType=TIMESTAMP})
  </insert>
```

注意：

```xml
	<selectKey keyProperty="id" resultType="long" order="AFTER">
  		SELECT LAST_INSERT_ID()
  	</selectKey>
```

这个代码是mybatis自增主键返回代码



下面的是insertSelective

```xml
<insert id="insertSelective" parameterType="com.taotao.pojo.TbContentCategory" >
    <selectKey keyProperty="id" resultType="long" order="AFTER">
  		SELECT LAST_INSERT_ID()
  	</selectKey>
    insert into tb_content_category
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        id,
      </if>
      <if test="parentId != null" >
        parent_id,
      </if>
      <if test="name != null" >
        name,
      </if>
      <if test="status != null" >
        status,
      </if>
      <if test="sortOrder != null" >
        sort_order,
      </if>
      <if test="isParent != null" >
        is_parent,
      </if>
      <if test="created != null" >
        created,
      </if>
      <if test="updated != null" >
        updated,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=BIGINT},
      </if>
      <if test="parentId != null" >
        #{parentId,jdbcType=BIGINT},
      </if>
      <if test="name != null" >
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="status != null" >
        #{status,jdbcType=INTEGER},
      </if>
      <if test="sortOrder != null" >
        #{sortOrder,jdbcType=INTEGER},
      </if>
      <if test="isParent != null" >
        #{isParent,jdbcType=BIT},
      </if>
      <if test="created != null" >
        #{created,jdbcType=TIMESTAMP},
      </if>
      <if test="updated != null" >
        #{updated,jdbcType=TIMESTAMP},
      </if>
    </trim>
  </insert>
```

其中:

`<trim prefix="" suffix="" suffixOverrides="" prefixOverrides=""></trim>`

prefix:在trim标签内sql语句加上前缀。

suffix:在trim标签内sql语句加上后缀。

suffixOverrides:指定去除多余的后缀内容，如：suffixOverrides=","，去除trim标签内sql语句多余的后缀","。

prefixOverrides:指定去除多余的前缀内容



**insertSelective和insert的区别？**

1、selective的意思是：选择性

2、insertSelective--选择性保存数据；

比如User里面有三个字段:id，name，age，password

但是我只设置了一个字段；

User u=new user();

u.setName（"张三"）；

insertSelective（u）；

3、insertSelective执行对应的sql语句的时候，只插入对应的name字段；（主键是自动添加的，默认插入为空）

insert into tb_user （id，name） value （null，"张三"）；

4、而insert则是不论你设置多少个字段，统一都要添加一遍，不论你设置几个字段，即使是一个。

User u=new user();

u.setName（"张三"）；

insertSelective（u）；

insert into tb_user （id，name，age，password） value （null，"张三"，null，null）；

