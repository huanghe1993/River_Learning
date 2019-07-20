# SpringBoot的微信点餐系统——项目设计



## 一、功能分析

从角色的角度上讲

 买家是（微信公众账号上看到的服务）                                 卖家（PC端）

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/EJKebIkiJC.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/HacjeHdihe.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/9lEIlB1m7B.png?imageslim)

如果访问的是前端的资源则会请求Nginx服务器，如果请求的是服务端的资源则会访问tomcat





![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/7mCmcE7Bm8.png?imageslim)



## 数据库的设计

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/h1HeiIg66E.png?imageslim)

类目表--->商品表    1：n

订单主表------>订单详情表    1：n     (一个订单主表里有多个订单)      一个人买了两个商品，详情表会有两条记录

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/Kil1Gl0D3C.png?imageslim)

```sql
1 创建表的时候写注释

create table test1

(
    field_name int comment '字段的注释'

)comment='表的注释';


2 修改表的注释

alter table test1 comment '修改后的表的注释';

3 修改字段的注释

alter table test1 modify column field_name int comment '修改后的字段注释';

decimal(8,2)
8指定指定小数点左边和右边可以存储的十进制数字的最大个数，最大精度38。
2指定小数点右边可以存储的十进制数字的最大个数。
```

```sql
#商品表
create table 'product_info'(
	'product_id' varchar(32) not null,
    'product_name' varchar(64) not null comment '商品名称',
  	'product_price' decimal(8,2) not null comment '单价',
  	'product_stock' int not null comment '库存',
  	'product_description' varchar(64) comment '描述',
    'product_icon' varchar(512) comment '小图',
    'category_type' int not null comment '类目',
  	'create_time' timestamp not null default current_timestamp comment '创建时间',
    'update_time' timestamp not null default current_timestamp on update current_timestamp       comment '修改时间',
   primary key('product_id')
) comment '商品表'
```
![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/fLAhlID5k6.png?imageslim)

```
unique 就是唯一，当你需要限定你的某个表字段每个值都唯一,没有重复值时使用。比如说,如果你有一个person 表，并且表中有个身份证的column，那么你就可以指定该字段为unique。 从技术的角度来看，Primary Key和Unique Key有很多相似之处。但还是有以下区别：

一、作为Primary Key的域/域组不能为null，而Unique Key可以。 

二、在一个表中只能有一个Primary Key，而多个Unique Key可以同时存在。 

更大的区别在逻辑设计上。Primary Key一般在逻辑设计中用作记录标识，这也是设置Primary Key的本来用意，而Unique Key只是为了保证域/域组的唯一性。 

oracle的constraint中有两种约束，都是对列的唯一性限制――unique与primary key，但其中是有区别的：

1、unique key要求列唯一，但不包括null字段，也就是约束的列可以为空且仅要求列中的值除null之外不重复即可；

2、primary key也要求列唯一，同时又限制字段的值不能为null，相当于Primary Key=unique + not null。 

创建一个primary key和unique key都会相应的创建一个unique index。

0primary key的语法：alter table table name add constraint key name primary key( columns); 

unique key的语法：alter table table name add constraint key name unique( columns
```

```sql
--类目表
create table 'product_category'(
	'category_id' int not null auto_increment,
	'category_name' varchar(64) not null comment '类目名称',
    'category_type' int not null comment '类目编号',
  	'create_time' timestamp not null default current_timestamp comment '创建时间',
    'update_time' timestamp not null default current_timestamp on update current_timestamp       comment '修改时间',
  	  primary key('category_id'),
  	  unique key 'uqe_category_type'
) comment '类目表'
```






```sql
CREATE TABLE wh_logrecord ( 
logrecord_id int(11) NOT NULL auto_increment, 
user_name varchar(100) default NULL, 
operation_time datetime default NULL, 
logrecord_operation varchar(100) default NULL, 
PRIMARY KEY (logrecord_id), 
KEY wh_logrecord_user_name (user_name) 
) 
解析： 
KEY wh_logrecord_user_name (user_name) 
本表的user_name字段与wh_logrecord_user_name表user_name字段建立外键 
括号外是建立外键的对应表，括号内是对应字段 

类似还有 KEY user(userid) 当然，key未必都是外键 
总结： 
Key是索引约束，对表中字段进行约束索引的，都是通过primary foreign unique等创建的。常见有foreign key，外键关联用的。 
KEY forum (status,type,displayorder)  # 是多列索引（键） 
KEY tid (tid)                         # 是单列索引（键）。 
如建表时： KEY forum (status,type,displayorder) 
select * from table group by status,type,displayorder 是否就自动用上了此索引， 
而当 select * from table group by status 此索引有用吗？ 
key的用途：主要是用来加快查询速度的。
```

```sql
--订单主表
create table 'order_master'(
  	'order_id' varchar(32) not null,
  	'buyer_name' varchar(32) not null comment '买家名字',
  	'buyer_phone' varchar(32) not null comment '买家电话',
  	'buyer_address' varchar(128) not null comment '买家地址',
  	'buyer_openid' varchar(64) not null comment '买家微信id',
  	'order_amount' decimal(8,2) not null comment '订单总金额',
  	'order_status' tinyint(3) not null default '0' comment '订单状态，默认0新下单',
  	'pay_status' tinyint(3) not null default '0' comment '支付状态。默认为0未支付',
  	'create_time' timestamp not null default current_timestamp comment '创建时间',
    'update_time' timestamp not null default current_timestamp on update current_timestamp       comment '修改时间',
  	primary key('order_id'),
  	key 'idx_buyer_openid'('buyer_openid')	
) comment '订单表'
```



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180324/lJFICjICG1.png?imageslim)





```sql
-- 订单详情表
create table `order_detail` (
    `detail_id` varchar(32) not null,
    `order_id` varchar(32) not null,
    `product_id` varchar(32) not null,
    `product_name` varchar(64) not null comment '商品名称',
    `product_price` decimal(8,2) not null comment '当前价格,单位分',
    `product_quantity` int not null comment '数量',
    `product_icon` varchar(512) comment '小图',
    `create_time` timestamp not null default current_timestamp comment '创建时间',
    `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
    primary key (`detail_id`),
    key `idx_order_id` (`order_id`),
    foreign key(`order_id`) REFERENCES order_master(`order_id`)
);
```

```sql
 foreign key(`order_id`) REFERENCES order_master(`order_id`)
 -- 表示的是一个订单主表可能有多个订单详情表，所以在订单表里需要加上外键引用
```





```sql
-- 卖家(登录后台使用, 卖家登录之后可能直接采用微信扫码登录，不使用账号密码)
create table `seller_info` (
    `id` varchar(32) not null,
    `username` varchar(32) not null,
    `password` varchar(32) not null,
    `openid` varchar(64) not null comment '微信openid',
    `create_time` timestamp not null default current_timestamp comment '创建时间',
    `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
    primary key (`id`)
) comment '卖家信息表';
```