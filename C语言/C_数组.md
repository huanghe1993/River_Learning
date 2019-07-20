# C_一维数组

> 数组按照数据元素类型的不同可以分为数值数组、字符数组、指针数组、结构数组等各类别

## 一维数组的定义和初始化的赋值

VC的编译器整型占据的是4个字节，而浮点型占据的是8个字节

允许在同一个类型说明中，说明多个数组和多个变量。

比如int a,b,c,d,k1[10],k2[20]；

特别注意的是：c语言不允许对数组的大小作动态的定义(这在Java语言中也是一样的,java语言中数组是必须要初始化才可以使用的，但是在c语言中是不需要进行初始化的就可以进行使用的),但是可以进行动态的赋值，注意他们之间的区别；如下的形式是不允许的

```c
int a;
scanf("%d",&n);
int a[n];//在编译的阶段就需要规定数组需要多少空间，不能再允许的阶段再进行定义

int k,a[k]//这种类型也是不正确的，不能用变量说明数组的大小
  
  
```



初始化的赋值：（在编译的阶段进行赋值的）

类型说明符 数组名[常量表达式]={值，值，值，值，.........}；

```c
int a[10]={0,1,2,3,4};//这种只给前面的五个数值赋值，后面的五个数值自动的赋值为0

int b[10]={0};//所有的数值都赋值为0

```



在对数组元素赋初值的时候，由于数据的个数已经确定，因此可以不用指定数组的长度

```c
int a[]={1,2,3,4,5}; //这种方式是正确的，系统会自动的确定数组的长度
```



动态赋值

```c
void main()
{
  int i,max[10];
  printf("input 10 number!")
  for(i=0;i<10;i++){
    scanf("%d",&a[i]);//动态的进行赋值
  }
}
```



## 一维数组在内存中的存放

一维数组int mark[100];

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180323/m76Kl3k92c.png?imageslim)

## 一维数组元素使用的注意事项

在C语言中只能逐个地使用下标变量，而不能一次引用整个数组

```c
printf("%d",a); //这种 引用是错误的
```



## 二分法进行查找数据



```c
#include <stdio.h>
#define M 10
int main()
{
	int low,mid,high,n,found;
	int a[M]={-12,0,6,16,23,56,80,100,110,115};
	low=0;
	high=M-1;
	printf("input a number to be searched!\n");
  #if (0)
	scanf("%d",&n);  //scanf是有返回值的，返回的是成功读入的个数，如果函数遇到错误返回的是EOF
  #endif
    
  	while (scanf("%d",&n)!=1){
      printf("illeagal input\n  please input again");
      getchar();
  	}
	while(low<=high){
		mid=(low+high)/2;
		if(n==a[mid])
		{
			found=1;
			break;
		}
		else if(a[mid]<n){
			low=mid+1;
		}
		else{
			high=mid-1;
		}
	}
	if(1==found){
		printf("the indes of %d is %d \n",n,mid);
	}
	else{
		printf("There is no %d.........\n",n);
	}
	return 0;
}
```



