# C_结构体数组

```c
#include <stdio.h>

void main()
{
      struct student    /*定义结构*/
      {            
            int num;            
            char *name;            
            char sex;            
            float score;            
      }boy1, boy2 = { 102, "Jane", 'M', 98.5 };

      boy1 = boy2;
      
      printf("Number = %d\nName = %s\nScore = %d\n", boy1.num, boy1.name, boy1.score);
      printf("\n\n");
      printf("Number = %d\nName = %s\nScore = %d\n", boy2.num, boy2.name, boy2.score);
      
      
}


```

## 结构体数组

一个结构体变量中可以存放一组数据（如一个学生的学号、姓名、成绩等数据）

如果有10个学生的数据需要参加运算，显然应该用数组，这就是结构体数组。

结构体数组与以前介绍过的数值型数组不同之处在于每**个数组元素都是一个结构体类型的数据**，它们都分别包括各个成员（分量）项。

## 定义结构体数组

和定义结构体变量的方法相仿，只需说明其为数组即可。例如：

```c
struct student
{
		int num;
		char name[20];
		char sex;
		int age;
		float score;
		char addr[30];
 };
struct student student[3];

```

或者是：

```c
struct student
{
	int num;
	char name[20];
	char sex;
	int age;
	float score;
	char addr[30];
 }student[3];

```

## 结构体数组的初始化

```c
struct student
｛
	int num;
	char name[20]； 
	char sex；     
  	int age； 
	float score; 
	char addr[30]；
｝stu［2］＝ {
		 {101，″LiLin″，′M′，18，87.5，″Beijing″｝，
         {102，″Zhang″，′F′，19，99，″Shanghai″}
	     }；　 

```

当然，数组的初始化也可以用以下形式：

```c
struct student
｛
	int num；
  	…
｝;
  
struct student　str[]{{…},{…},{…}}；

```

即先声明结构体类型，然后定义数组为该结构体类型，在定义数组时初始化