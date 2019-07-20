# C—字符串与指针

用字符数组存放一个字符串，然后输出该字符串。

```c
#include <stdio.h>

void main()
{
      char string[] = "I love Fishc.com!";
      
      printf("%s\n", string);
}

```

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180329/JFj7hGEJbj.png?imageslim)

```c
#include <stdio.h>

void main()
{
      char *String = "I love Fishc.com!";
      
      printf("%s\n", string);
}

```

## 字符串中字符的存取方法

对字符串中字符的存取，可以用下标方法，也可以用指针方法！

```c
#include <stdio.h>

void main()
{
      char a[] = "Fishc.com is a good web site!", b[40];
      int i;
      
      for(i=0; *(a+i) != '\0'; i++)
      {
            *(b+i) = *(a+i);
      }
      *(b+i) = '\0';

      printf("String a is: %s\n", a);
      printf("String b is: ");
      for(i=0; b[i] != '\0'; i++)
      {
            printf("%c", b[i]);
      }
      
      printf("\n\n");
}

```

```c
#include <stdio.h>

void main()
{
      char a[] = "Fishc.com is a good web site!", b[40], *p1, *p2;
      int i;
      
      p1 = a;
      p2 = b;
      
      for( ; *p1 != '\0'; p1++, p2++)
      {
            *p2 = *p1;
      }
      *p2 = '\0';

      printf("String a is: %s\n", a);
      printf("String b is: ");
      for(i=0; b[i] != '\0'; i++)
      {
            printf("%c", b[i]);
      }

      printf("\n");
}

```



## 字符赋值的函数

```c
#include <stdio.h>

void  main()
{
      void  copy_string(char from[], char to[]);

      char a[] = "I am a teacher.";
      char b[] = "You are a student.";
      
      printf("string a = %s\nstring b = %s\n", a, b);
      printf("copy string a to string b:\n ");

      copy_string(a, b);
      
      printf("\nstring a = %s\nstring b = %s\n", a, b);
}

void  copy_string(char from[], char to[])
{
      int i = 0;

      while( from[i] != '\0' )
      {
            to[i] = from[i];
            i++;
      }
      to[i] = '\0';
}
```

指针的方法

```c
#include <stdio.h>

void  main()
{
      void copy_string( char *from, char *to );

      char *a = "I am a teacher.";
      char *b = "You are a student.";
      
      printf("String a = %s\nString b = %s\n", a, b);
      printf("copy string a to string b:\n");

      copy_string(a, b);
 
      printf("\nString a = %s\nString b = %s\n", a, b);
}

void  copy_string( char *from, char *to )
{
      for( ; *from != '\0'; from++,to++)
      {
            *to = *from;
      }

      *to = '\0';
}

//丫的，出错了，为什么? WHY??
//因为这里的是字符串是保存在常量存储区的，而指针指向的是常量存储区的，这里面的数值是
//不能改变的
```

## 字符指针作为函数的参数

用指针变量作为字符串的情形下，就是把**字符串的偏移地址**赋值给指针变量。

如果是用**字符数组存储字符串**，是把每一个值赋值给字母。

```c
#include <stdio.h>

void  main()
{
      void copy_string( char *from, char *to );

      char *a = "I am a teacher.";
      char b[] = "You are a student.";

      printf("String a = %s\nString b = %s\n", a, b);
      printf("copy string a to string b:\n");

      copy_string(a, b);
 
      printf("\nString a = %s\nString b = %s\n", a, b);
}

void  copy_string( char *from, char *to )
{
      while( (*to = *from) != '\0' )
      {
            to++;
            from++;
      }
}


```

## 字符指针变量和字符数组

1. 字符数组由若干个元素组成，每个元素中 放一个字符，而字符指针变量中存放的是地址（字符串第1个字符的地址），决不是将字符串放到字符指针变量中



  2.赋值方式。对字符数组只能对各个元素赋值，不能用以下办法对字符数组赋值。

​     char str［20］;

​     str＝″I love Fishc.com！″;

  而对字符指针变量，可以采用下面方法赋值：

​     char   *ａ；

​     a＝″I love Fishc.com！″;

  **但注意赋给ａ的不是字符，而是字符串第一个 元素的地址**。



3.对字符指针变量赋初值：

​    char *ａ＝″I love Fishc.com！″；

​       等价于

   char *ａ；

​    ａ＝″I love Fishc.com!″；

​    而对数组的初始化：

​    char str[20]＝｛″I love Fishc.com!″｝;

​    不能等价于

   char str[20];

​    str[ ]＝″I love Fishc.com!″;



4. 如果定义了一个**字符数组，在编译时为它分配内存单元，它有确定的地址**。而定义一个字符指针变量时，给指针变量分配内存单元，在其中可以放一个字符变量的地址也就是说，该指针变量可以指向一个字符型数据，但如果未对它赋予一个地址值，则它并未具体指向一个确定的字符数据

```c
如:   char str［１０］；
      scanf（″％ｓ″，str）；
           以上是完全可以的！

而常有人用下面的方法,目的是想输入一个字符串，虽然一般也能运行，但这种方法是危险的 ：
             char *ａ；
      scanf（″％ｓ″，ａ）；

```

5.指针变量的值是可以改变的，如

​    

```c
#include <stdio.h>

void main()
{
      char *a = "I love Fishc.com!";
      printf("%s\n", a);

      a += 7;
      printf("%s\n", a);
}
```

​    另外需要说明的是，若定义了一个指针变量，并使它指向一个字符串**，就可以用下标形式引用指针变量所指的字符串中的字符。**

```c
#include <stdio.h>
void main()
{
      char *a = "I love Fishc.com!";
      int i;

      printf("The sixth character is %c\n\n", a[5]);

      for( i=0; a[i] != '\0'; i++ )
      {
            printf("%c", a[i]);
      }

      printf("\n");
}

```