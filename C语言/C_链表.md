## C_链表

什么是链表？

链表是一种常见的重要的数据结构,是动态地进行存储分配的一种结构。



链表的组成：

**头指针**：存放一个地址，该地址指向第一个元素

**结点**：用户需要的实际数据和链接节点的指针

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/GC0DEFlhCa.png?imageslim)

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/3Ib4F2af7k.png?imageslim)

```c
#include <stdio.h>

struct student
{
      long num;  
      float score;
      struct student *next;
};

void main()
{
      struct student a, b, c, *head;

      a.num = 10101; 
      a.score = 89.5;
      b.num = 10103;
      b.score = 90;
      c.num = 10107;
      c.score = 85;

      head = &a;
      a.next = &b;
      b.next = &c;
      c.next = NULL;

      do
      {
            printf("%ld %5.1f\n", head->num, head->score);
            head = head->next;
      }while( head != NULL );
}
```



## 建立动态链表

所谓建立动态链表是指在**程序执行过程中从无到有地建立起一个链表，即一个一个地开辟结点和输入各结点数**据，并建立起前后相链的关系。



作业：根据下面的分析写一函数建立一个含有学生（学号，成绩）数据的单向动态链表。

（约定：我们约定学号不会为零，如果输入的学号为０，则表示建立链表的过程完成，该结点不应连接到链表中。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/e4bCie4mG8.png?imageslim)

```c
如果输入的p1->num不等于０，则输入的是第一个结点数据（n=1），令head＝p1，即把p1的值赋给head，也就是使head也指向新开辟的结点p1所指向的新开辟的结点就成为链表中第一个结点。
```

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/dbLBJFLgij.png?imageslim)

```
再开辟另一个结点并使p1指向它，接着输入该结点的数据。

如果输入的p1->num≠０，则应链入第２个结点（n=2), 将新结点的地址赋给第一个结点的next成员。p2->next=p1

接着使ｐ２＝ｐ１，也就是使ｐ２指向刚才建立的结点。
```







![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/gg940FIEAI.png?imageslim)



```
再开辟一个结点并使p1指向它，并输入该结点的数据。

在第三次循环中，由于ｎ＝３（ｎ≠１），又将ｐ１的值赋给ｐ２->ｎｅｘｔ，也就是将第３个结点连接到第２个结点之后，并使ｐ２＝ｐ１，使ｐ２指向最后一个结点。

```





![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/6jmc0BBKB6.png?imageslim)



```
再开辟一个新结点，并使p1指向它，输入该结点的数据。由于p1->num的值为０，不再执行循环，此新结点不应被连接到链表中

将NULL赋给p2->next

建立链表过程至此结束，p1最后所指的结点未链入链表中，第三个结点的next成员的值为NULL，它不指向任何结点
```



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/g8El9Ji0iK.png?imageslim)



```c
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

#define LEN sizeof(struct student)  // student结构的大小

struct student *creat();            //创建链表
void print(struct student *head);   //打印链表

struct student
{
      int num;
      float score;
      struct student *next;
};

int n; //全局变量，用来记录存放了多少数据。

void main()
{
      struct student *stu;

      stu = creat();
      print( stu );

      printf("\n\n");
      system("pause");
}

struct student *creat()
{
      struct student *head;
      struct student *p1, *p2;
  
      p1 = p2 = (struct student *)malloc(LEN);  // LEN是student结构的大小

      printf("Please enter the num :");
      scanf("%d", &p1->num);
      printf("Please enter the score :");
      scanf("%f", &p1->score);

      head = NULL;     
      n = 0;    
      
      while( 0 != p1->num )
      {
            n++;
            if( 1 == n )
            {
                  head = p1;                 
            }
            else
            {
                  p2->next = p1;
            }

            p2 = p1;

            p1 = (struct student *)malloc(LEN);

            printf("\nPlease enter the num :");
            scanf("%d", &p1->num);
            printf("Please enter the score :");
            scanf("%f", &p1->score);
      }

      p2->next = NULL;

      return head;
}

void print(struct student *head)
{
      struct student *p;
      printf("\nThere are %d records!\n\n", n);

      p = head;
      if( NULL != head )
      {
            do
            {
                  printf("学号为 %d 的成绩是: %f\n", p->num, p->score);
                  p = p->next;
            }while( NULL != p );
      }
}
```

## 链表的输出

```
首先要知道链表第一个结点的地址，也就是要知道head的值。

然后设一个指针变量p，先指向第一个结点，输出ｐ所指的结点，然后使ｐ后移一个结点，再输出，直到链表的尾结点。

如此我们得到这样的流程图：

```



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/dGgkL898a8.png?imageslim)

## 对链表的删除操作

从一个动态链表中删去一个结点，并不是真正从内存中把它抹掉，而是把它从链表中分离开来，只要撤销原来的链接关系即可。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/8IDk6mFG4l.png?imageslim)

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/1dK98fFHgH.png?imageslim)

```c
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

#define LEN sizeof(struct student)  // student结构的大小

struct student *creat();   //创建链表
struct student *del( struct student *head, int num);  //del函数用于删除结点, *head即链表
                                                      //的头指针, num是要删除的结点num。

void print(struct student *head);   //打印链表

struct student
{
      int num;
      float score;
      struct student *next;
};

int n; //全局变量，用来记录存放了多少数据。

void main()
{
      struct student *stu, *p;
      int n;

      stu = creat();
      p = stu;
      print( p );

      printf("Please enter the num to delete: ");
      scanf("%d", &n);
      print( del(p, n) );
      
      printf("\n\n");
      system("pause");
}

struct student *creat()
{
      struct student *head;
      struct student *p1, *p2;
  
      p1 = p2 = (struct student *)malloc(LEN);  // LEN是student结构的大小

      printf("Please enter the num :");
      scanf("%d", &p1->num);
      printf("Please enter the score :");
      scanf("%f", &p1->score);

      head = NULL;     
      n = 0;    
      
      while( p1->num )
      {
            n++;
            if( 1 == n )
            {
                  head = p1;                 
            }
            else
            {
                  p2->next = p1;
            }

            p2 = p1;
            p1 = (struct student *)malloc(LEN);

            printf("\nPlease enter the num :");
            scanf("%d", &p1->num);
            printf("Please enter the score :");
            scanf("%f", &p1->score);
      }

      p2->next = NULL;

      return head;
}

void print(struct student *head)
{
      struct student *p;
      printf("\nThere are %d records!\n\n", n);

      p = head;
      if( head )
      {
            do
            {
                  printf("学号为 %d 的成绩是: %f\n", p->num, p->score);
                  p = p->next;
            }while( p );
      }
}

struct student *del( struct student *head, int num)
{
      struct student *p1, *p2;
      
      if( NULL == head )
      {
            printf("\nThis list is null!\n");
            goto end;
      }

      p1 = head;
      while( p1->num != num && p1->next != NULL)
      {
            p2 = p1;
            p1 = p1->next;
      }
      if( num == p1->num )
      {
            if( p1 == head )
            {
                  head = p1->next;
            }
            else
            {
                  p2->next = p1->next;
            }

            printf("Delete No: %d succeed!\n", num);
            n = n-1;//全局变量，记录的是链表的个数 
      }
      else
      {
            printf("%d not been found!\n", num);
      }

end:
      return head;
}
```

##对链表的插入操作

对链表的插入是指将一个结点插入到一个已有的链表中。

为了能做到正确插入，必须解决两个问题：

  ①怎样找到插入的位置；

 ②怎样实现插入。



我们可以先用指针变量p0指向待插入的结点，p1指向第一个结点。将p0->num与p1->num相比较，如果p0->num＞p1->num ，此时将p1后移，并使p2指向刚才p1所指的结点。



![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/G7mKkF5g1E.png?imageslim)







![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180403/dgA5FIli21.png?imageslim)



```c
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

#define LEN sizeof(struct student)  // student结构的大小

struct student *creat();   //创建链表
struct student *del(struct student *head, int num);  //del函数用于删除结点, *head即链表
                                                     //的头指针, num是要删除的结点num。
struct student *insert(struct student *head, struct student *stu_2);  // 第一个参数需要被插入的链表
                                                                      // 第二个参数待插入的结构的地址

void print(struct student *head);   //打印链表

struct student
{
      int num;
      float score;
      struct student *next;
};

int n; //全局变量，用来记录存放了多少数据。

void main()
{
      struct student *stu, *p, stu_2;
      int n;
      
      stu = creat();
      p = stu;
      print( p );
      
      printf("\nPlease input the num to delete: ");
      scanf("%d", &n);
      print( del(p, n) );

      printf("\nPlease input the num to insert: ");
      scanf("%d", &stu_2.num);
      printf("Please input the score: ");
      scanf("%f", &stu_2.score);

      p = insert(stu, &stu_2);
      print( p );


      

      printf("\n\n");
      system("pause");
}

struct student *creat()
{
      struct student *head;
      struct student *p1, *p2;
      
      p1 = p2 = (struct student *)malloc(LEN);  // LEN是student结构的大小
      
      printf("Please enter the num :");
      scanf("%d", &p1->num);
      printf("Please enter the score :");
      scanf("%f", &p1->score);
      
      head = NULL;     
      n = 0;    
      
      while( p1->num )
      {
            n++;
            if( 1 == n )
            {
                  head = p1;                 
            }
            else
            {
                  p2->next = p1;
            }
            
            p2 = p1;
            p1 = (struct student *)malloc(LEN);
            
            printf("\nPlease enter the num :");
            scanf("%d", &p1->num);
            printf("Please enter the score :");
            scanf("%f", &p1->score);
      }
      
      p2->next = NULL;
      
      return head;
}

void print(struct student *head)
{
      struct student *p;
      printf("\nThere are %d records!\n\n", n);
      
      p = head;
      if( head )
      {
            do
            {
                  printf("学号为 %d 的成绩是: %f\n", p->num, p->score);
                  p = p->next;
            }while( p );
      }
}

struct student *del( struct student *head, int num)
{
      struct student *p1, *p2;
      
      if( NULL == head )
      {
            printf("\nThis list is null!\n");
            goto end;
      }
      
      p1 = head;
      while( p1->num != num && p1->next != NULL)
      {
            p2 = p1;
            p1 = p1->next;
      }
      if( num == p1->num )
      {
            if( p1 == head )
            {
                  head = p1->next;
            }
            else
            {
                  p2->next = p1->next;
            }
            
            printf("Delete No: %d succeed!\n", num);
            n = n-1;
      }
      else
      {
            printf("%d not been found!\n", num);
      }
      
end:
      return head;
}

struct student *insert(struct student *head, struct student *stu_2)
{
      struct student *p0, *p1, *p2;
      
      p1 = head;
      p0 = stu_2;
      if( NULL == head ) 
      {
            head = p0;
            p0->next = NULL;
      }
      else
      {
            while( (p0->num > p1->num) && (p1->next != NULL) ) //两种情况推出while，一：
            {
                  p2 = p1;
                  p1 = p1->next;
            }
            
            if( p0->num <= p1->num )
            {
                  if( head == p1 )   // p1是头结点，插入头部
                  {
                        head = p0;  
                  }
                  else               // 普通情况，插入中间
                  {
                        p2->next = p0;
                  }

                  p0->next = p1;
            }
            else   // p0的num最大，插入到末尾
            {
                  p1->next = p0;
                  p0->next = NULL;
            }
      }

      n = n+1;   // 由于插入了，所以增加了一位数据成员进入链表中。
      
      return head;
}
```

