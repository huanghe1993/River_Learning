# Spring Data  Jpa

## spring data jpa有什么

主要看看Spring Data JPA提供的编程接口

**Respository**:最顶层的接口，是一个**空接口**，目的是为统一所有的Respository的类型，且让组件扫面的时候自动识别

**CrudRespository**:Respository的子接口，提供**CRUD**的功能

**PagingAndSortingRepository**:CrudResporsitory的子接口，添加**分页排序**的功能

**JpaRepository**:PagingAndSortingRepository的子接口，增加**批量操作**等功能

**JpaSpecificationExector**:用来做复杂查询的接口





JpaRepository接口方法：

delete:删除或者是批量删除；

findAll:查找所有激励

findone:查找单个

save:保存单个或者是批量

saveAndFlush:保存并且立即的刷新到数据库中去







![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/1f7aLfjc1A.png?imageslim)



## 建立简单的项目

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/3ei1hLD40g.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/LJ1EmAfi6A.png?imageslim)

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/hLmcD2F8eG.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/9km3dDJBI8.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/7CDmBlaiH8.png?imageslim)



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180325/7KfdDHlHG3.png?imageslim)



测试类：





