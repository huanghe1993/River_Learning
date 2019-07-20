# C_数组和指针

## 指向数组的指针

- 定义一个指向数组元素的指针变量的方法，与以前介绍的指向变量的指针变量相同

```c
int a[10]
(定义a为包含10个整型数据的数组)
int *p
(定义p为指向整型变量的指针)

 p=&a[0]  //把a[0]元素的地址赋给指针变量p，也就是使p指向a数组的第0号元素
```



## 引用数组元素

1. 下标法 a[i]的形式
2. 指针法如  *（a+1）或者是  *   (p+1)

其中a是数组名，p是指向数组元素的指针变量，其初值p=a.

**注意：数组名即“翻译成数组的第一个元素的地址”**

```c
#include <stdio.h>

void main()
{
      int a[10];
      int i;
      int *p;
      
      for( i=0; i < 10; i++ )
      {
            scanf("%d", &a[i]);
      }
      
      printf("\n");
      
  		//p指向数组a的首地址
      for( p=a; p < (a+10); p++ )
      {
            printf("%d", *p);
      }
}
```



## 数组名作为函数的参数

```c
f (int arr［ ］, int n)
但在编译时是将arr按指针变量处理的，相当于将函数f的首部写成  f (int *arr, int n)
以上两种写法是等价的
```

需要说明的是：C语言调用函数时虚实结合的方法都是**采用“值传递”方式**，当用变量名作为函数参数时传递的是变量的值，**当用数组名作为函数参数时，由于数组名代表的是数组首元素地址，因此传递的值是地址，所以要求形参为指针变量。** 



## 将数组a中个数组按照相反顺序存放

```c
#include <stdio.h>

void reverse(int x[],int n);   /*形参x是数组名*/

void main()
{
      int i, a[10] = {3, 7, 9, 11, 0, 6, 7, 5, 4, 2};

      printf("The original array:\n");

      for( i=0; i < 10; i++)
      {
            printf("%d ", a[i]);
      }
      printf("\n");

      reverse(a, 10);  //作用使得数组重新倒序排放

      printf("The array has been inverted:\n");

      for( i=0; i < 10; i++)
      {
            printf("%d ", a[i]);
      }
      printf("\n");

}

void reverse(int x[], int n)   /*形参x是数组名*/
{
      int temp, i, j, m;

      m = (n-1)/2;    
      
      for( i=0; i <= m; i++)
      {
            j = n-1-i;  //j指向对应的元素

            temp = x[i];
            x[i] = x[j];
            x[j] = temp;
      }
   
}


```



```c
#include <stdio.h>

void reserve(int *x, int n);   /*形参x为指针变量*/

void main()
{
      int i, a[10] = {3, 7, 9, 11, 0, 6, 7, 5, 4, 2};

      printf("The original array:\n");

      for( i=0; i < 10; i++)
      {
            printf("%d ", a[i]);
      }
      printf("\n");

      reserve(a, 10);

      printf("The array has benn inverted:\n");

      for( i=0; i < 10; i++)
      {
            printf("%d ", a[i]);
      }
      printf("\n");
}

void reserve(int *x, int n)   /*形参x为指针变量*/
{
      int *p, temp, *i, *j, m;

      m = (n-1)/2;
      i = x;         //i指向数组的第一个元素
      j = x-1+n;     //j指向的是数组的最后一个元素
      p = x+m;       //指向中间，配对……
      
      for( ; i <= p; i++, j--)
      {
            temp = *i;
            *i = *j;
            *j = temp;
      }
}

```



## 小结

如果有一个实参数组，想在函数中改变此数组中的元素的值，实参与形参的对应关系有一下的四种

- 形参和实参都是数组名

```c
void  main（）                  void  ｆ（int ｘ[]，int ｎ）
｛    
	{ int  a[10]；                       
    ｆ（ａ，１０）；         
   ｝ 

```

- 实参数组名，形参用指针变量

```c
void main()
{
	  int a[10];
	  f (a, 10);
}
f (int *a, int n)
{
    ……
}

```

- 实参和形参都是指针

```c
  void  main（）                    
  ｛                         
     int  a[10],  *p = a；
	   f (p , 10);
  ｝ 
  void  f (int *x, int n)
  {
        ……
  }

```

- 实参为指针变量，形参为数组名

```c
void main()                        
｛                                               
     int ａ[10],  *p = a；                             
         ………………
    ｆ（p，１０）；             
｝  
f （int x[], int n）
{
	…………
}

```



## 对数组中10个整数由大到小顺序排序

```c
#include <stdio.h>

void sort(int x[], int n);

void main()
{
      int *p, i, a[10] = {3, 7, 9, 11, 0, 6, 7, 5, 4, 2};
      
      printf("The original array:\n");
      
      for( i=0; i < 10; i++)
      {
            printf("%d ", a[i]);
      }
      
      
      printf("\n");
      
      p = a;
      sort( p, 10 );
   
      printf("The result is:\n");

      for( p=a,i=0; i < 10; i++ )         
      {
            printf("%d ", *p);
            p++;
      }
      printf("\n");
      
}

void sort(int x[], int n)
{
      int i, j, k, t;

      for( i=0; i < n-1; i++)
      {
            k = i;

            for( j=i+1; j < n; j++ )
            {
                  if( x[j] > x[k] )     
                  {
                        t = x[j];
                        x[j] = x[k];
                        x[k] = t;
                  }
            }
      }
}




```