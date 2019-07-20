# C_带参数的宏定义

Ｃ语言允许宏带有参数。在宏定义中的参数称为形式参数，在宏调用中的参数称为实际参数。



对带参数的宏，在调用中，**不仅要宏展开，而且要用实参去代换形参**。



带参宏定义的一般形式为` #define 宏名(形参表)  字符串`



带参宏调用的一般形式为：宏名(实参表)；  

例如: 

```c
 #define M(y) y*y+3*y      /*宏定义*/

      ……

   k=M(5);                   /*宏调用*/

          ……
```

在宏调用时，用实参5去代替形参y，经预处理宏展开后的语句为：

​    k=5*5+3*5

```c
#include <stdio.h>

#define MAX(a,b) (a>b)?a:b

void main()
{     
      int x, y, max;
      
      printf("input two numbers: ");
      
      scanf("%d %d", &x, &y);      
      max = MAX(x, y); // max = (x>y) ? x : y;
      
      printf("The max is %d\n\n", max);
      
}
```

## 对于带参的宏定义有以下问题需要说明：

1. 带参宏定义中，宏名和形参表之间不能有空格出现


```c
例如把：
    #define MAX(a,b) (a>b)?a:b
写为：
    #define MAX  (a,b)  (a>b)?a:b//这样是错误的

```

2. 在带参宏定义中，形式参数不分配内存单元，因此不必作类型定义**。而宏调用中的实参有具体的值**。要用它们去代换形参，因此必须作类型说明。（看example01.c）
3. 在宏定义中的形参是标识符，而宏调用中的实参可以是**表达式**。

```java
#include <stdio.h>

#define SAY(y) (y)

void main()
{      
      int i = 0;
      char say[] = {73, 32, 108, 111, 118, 101, 32, 102, 105, 115, 104, 99, 46, 99, 111, 109, 33, 0};

      while( say[i] )
      {
            say[i] = SAY(say[i]*1+1-1);
            i++;
      }

 
      printf("\n\t%s\n\n", say);
      
      
}
```

4. 在宏定义中，字符串内的形参通常要用括号括起来以避免出错。在上例中的宏定义中(y)*(y)表达式的y都用括号括起来，因此结果是正确的。如果去掉括号，把程序改为以下形式


```java
#include <stdio.h>

#define SQ(y) (y)*(y)   // y*y试试

void main()
{
      int a, sq;
      
      printf("input a number: ");
      
      scanf("%d", &a);
      
      sq = SQ(a+1); // sq = a+1 * a+1
      
      printf("sq = %d\n", sq);
      
}



```






