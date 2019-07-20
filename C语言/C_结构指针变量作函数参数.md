# C_结构指针变量作函数参数

将一个结构变量的值传递给另外一个函数，有三种方法：

（1）用结构体变量的**成员**作参数

（2）用结构体变量作实参

（3）用指向结构体变量（或数组）的指针作实参，将结构体变量（或数组）的地址传给形参

```c
include <stdio.h>
#include <string.h>

struct student
{
      int num;
      char *name;
      float score[3];
};

void print(struct student);

void main()
{
      struct student stu;
      
      stu.num = 8;
      stu.name = "Fishc.com!";
      stu.score[0] = 98.5;
      stu.score[1] = 99.0;
      stu.score[2] = 99.5;

      print( stu );//结构体变量的作实参
}

void print( struct student stu )
{
      printf("\tnum     : %d\n", stu.num);
      printf("\tname    : %s\n", stu.name);
      printf("\tscore_1 : %5.2f\n", stu.score[0]);
      printf("\tscore_2 : %5.2f\n", stu.score[1]);
      printf("\tscore_3 : %5.2f\n", stu.score[2]);
      printf("\n");
}
```

第二种方式：

```c
#include <stdio.h>
#include <string.h>

struct student
{
      int num;
      char name[20];
      float score[3];
};

void print(struct student *);

void main()
{
      struct student stu;
      
      stu.num = 8;
      strcpy(stu.name, "Fishc.com!"); 
      stu.score[0] = 98.5;
      stu.score[1] = 99.0;
      stu.score[2] = 99.5;

      print( &stu );
}

void print( struct student *p )
{
      printf("\tnum     : %d\n", p -> num);
      printf("\tname    : %s\n", p -> name);
      printf("\tscore_1 : %5.2f\n", p -> score[0]);
      printf("\tscore_2 : %5.2f\n", p -> score[1]);
      printf("\tscore_3 : %5.2f\n", p -> score[2]);
      printf("\n");
}


```

