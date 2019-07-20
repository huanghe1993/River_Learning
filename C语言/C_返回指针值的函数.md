# C_返回指针值的函数

一个函数可以带回一个整型值、字符值、实型值等，**也可以带回指针型的数据，即地址**。其概念与以前类似，只是带回的值的类型是指针类型而已。

**这种带回指针值的函数，一般定义形式为:**

​           类型名  *函数名（参数表列）;

例如：int　*ａ（intｘ，int ｙ）;

```c
#include <stdio.h>

void main()
{
      double score[][4] = {{60.0, 70.0, 80.5, 90.5}, {56.0, 89.0, 67.0, 88.0}, {34.2, 78.5, 90.5, 66.0}};
      double *search(double(*pointer)[4], int n);
      double *p;
      int i, m;

      printf("Please enter the number of student: ");
      scanf("%d", &m);

      printf("The scores of No.%d are: \n", m);

      p = search(score, m);

      for( i=0; i < 4; i++)
      {
            printf("%5.2f\t", *(p + i));
      }

      printf("\n\n\n");
}

double *search(double (*pointer)[4], int n)
{
      double *pt;

      pt = *(pointer + n);

      return pt;
}
```

