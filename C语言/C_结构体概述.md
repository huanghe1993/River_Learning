# C_结构体概述

有时需要将不同类型的数据组合成一个有机的整体，以便于引用。

一个学生有学号/姓名/性别/年龄/地址等属性   int num; char name[20]; char sex; int age;   int char addr[30];

| num  | name | sex  | age  | addr    |
| ---- | ---- | ---- | ---- | ------- |
| 007  | Jane | M    | 24   | Beijing |

定义一个结构的一般形式为：

```c
 struct 结构名

 {
     成员表列
};
```

成员表列由若干个成员组成，每个成员都是该结构的一个组成部分。对每个成员也必须作类型说明，其形式为：

   类型说明符   成员名;

```C
struct student
  {
      int num;
      char name[20];
      char sex;
      int age;
      float score;
      char addr[30];
  }

```

## 可以采取以下3种方法定义结构体类型变量：

**(1)先声明结构体类型再定义变量名****

例如：struct  student       student1, student2;

​                    |            |                     |               |       

​                  类型名   结构体             变量名      变量名 

定义了student1和student2为struct  student类型的变量，即它们具有struct student类型的结构.

```c
struct student

 {

      int num;

      char name[20];

      char sex;

      int age;

      float score;

      char addr[30];

 } student1, student2
```

在定义了结构体变量后，系统会为之分配内存单元。

例如:  student1和student2在内存中各占 ？ 个字节。（4 + 20 + 1 + 4 + 4 + 30 =  67 ）。

**(2)在声明类型的同时定义变量**

  这种形式的定义的一般形式为:

```c
 struct　结构体名
   ｛
       成员表列
   ｝变量名表列； 

```

```c
struct student
｛   
	 int num；
     char name[20]；
     char sex；
     int age；
     float score；
     char addr[30]；
｝student1,student2;
```

**(3) 直接定义结构体类型变量**

```c
其一般形式为:
      struct
    ｛
　　　   成员表列
　　｝变量名表列；

```

即不出现结构体名。



## 嵌套的结构

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/D543hKfkL8.png?imageslim)

首先定义一个结构date，由month(月)、day(日)、year(年)三个成员组成。

在定义并说明变量boy1 和 boy2 时，其中的成员birthday被说明为data结构类型。成员名可与程序中其它变量同名，互不干扰。

```c
struct date
{
    int month;
    int day;
    int year;
};

```

```c
struct
{
    int num;
    char name[20];
    char sex;
    struct date birthday;
    float score;
}boy1, boy2;
```

## 结构体变量的引用

在定义了结构体变量以后,当然可以引用这个变量。但应遵守以下规则:   

(1) 不能将一个结构体变量作为一个整体进行输入和输出。

例如:打印student1的各个变量的值。

可以这样吗？`printf(″%d,%s,%c,%d,%f,%\n″,student1);`这样是不可以的

正确引用结构体变量中成员的方式为：**结构体变量名.成员名**

(2) 可以引用结构体变量成员的地址，也可以引用结构体变量的地址。

**结构体变量的地址主要用作函数参数，传递结构体变量的地址。**







