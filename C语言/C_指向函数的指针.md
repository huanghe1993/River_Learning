# C_指向函数的指针

## 用函数指针变量调用函数

可以用指针变量指向整型变量、字符串、数组，**也可以指向一个函数**。一个函数在编译时被分配给一个入口地址。这个函数的入口地址就称为函数的指针

```c
/****************************/
/* 比较a 和 b的大小，求大值 */
/****************************/

#include <stdio.h>

#if(0)
void main()
{
      int max(int, int);
      int a, b, c;

      scanf("%d %d", &a, &b);
      
      c = max(a, b);
      
      printf("a = %d, b = %d, max = %d\n\n", a, b, c);
}
#endif

int max(int x, int y)
{
      int z;
      
      if( x > y )
      {
            z = x;
      }
      else
      {
            z = y;
      }

      return z;
}

#if(1)
//将 main 函数改写为
#include <stdio.h>

void main()
{
      int max(int, int);
      int (*p)();//声明指向函数的指针
      int a, b, c;

      p = max;
      scanf("%d %d", &a, &b);
      
      c = (*p)(a, b);
      
      printf("a = %d, b = %d, max = %d\n\n", a, b, c);
}

#endif
```

## 用指向函数的指针作函数参数

- 函数指针变量常用的用途之一是把指针作为参数传递到其他函数。
- 前面介绍过，函数的参数可以是变量、指向变量的指针变量、数组名、指向数组的指针变量等。
- 现在介绍指向函数的指针也可以作为参数，以实现函数地址的传递，这样就能够在被调用的函数中使用实参函数。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180330/8HHEJieFha.png?imageslim)

它的原理可以简述如下：有一个函数（假设函数名为sub），它有两个形参（x1和x2），定义x1和x2为指向函数的指针变量。在调用函数sub时，实参为两个函数名ｆ１和ｆ２，给形参传递的是函数ｆ１和ｆ２的地址。这样在函数ｓｕｂ中就可以调用ｆ１和ｆ２函数了。



## 输入ａ和ｂ两个数，第一次调用process时找出ａ和ｂ中大者，第二次找出其中小者，第三次求ａ与ｂ之和

```c
/***********************************************************/
/*  设一个函数process，在调用它的时候，每次实现不同的功能。*/
/*  输入ａ和ｂ两个数，第一次调用process时找出ａ和ｂ中大者，*/
/*  第二次找出其中小者，第三次求ａ与ｂ之和。               */
/***********************************************************/

#include <stdio.h>

void main()
{
      int max(int, int);            /* 函数声明 */
      int min(int, int);            /* 函数声明 */
      int add(int, int);            /* 函数声明 */
    
      void process( int, int, int(*fun)() );    /* 函数声明 */
      
      int a, b;

      printf("Endter a and b: ");
      scanf("%d %d", &a, &b);
      
      printf("max = ");
      process(a, b, max);

      printf("min = ");
      process(a, b, min);

      printf("sum = ");
      process(a, b, add);
}

int max(int x, int y)              /* 函数定义 */
{
      int z;
      
      if( x > y )
      {
            z = x;
      }
      else
      {
            z = y;
      }

      return z;
}

int min(int x, int y)              /* 函数定义 */
{
      int z;

      if( x < y )
      {
            z = x;
      }
      else
      {
            z = y;
      }

      return z;
}

int add(int x, int y)
{
      int z;
      
      z = x + y;
      return z;
}

void process( int x, int y, int(*fun)() )    /* 函数定义 */ 
{
      int result;

      result = (*fun)(x, y);
      printf("%d\n", result);
}

```

















