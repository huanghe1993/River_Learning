#Java创建对象的存储的区域

>在JAVA的世界里一切都是对象，但是操作的标识符实际上的对象的一种“引用”。你拥有一个引用并不需要对象相关联

##1、对象创建存储的地方
**寄存器**
>最快的存储区，位于处理器内部，但是寄存器数量有限，so 根据需求进行分配，不能直接控制，在程序中不能感觉到寄存器的存在。

**堆栈**
>位于RAM，通过堆栈指针可以从处理器那里获得直接支持。<font color=red>堆栈指针若向下移动，则分配新的内存；若向上移动，则释放那些内存。</font>创建程序时，java系统必须知道存储在堆栈内所有项的确切生命周期，以便上下移动堆栈指针。Java数据存储于堆栈中，特别是对象的引用，Java对象并不存储于其中

**堆**
>一种通用的内存池（也位于RAM区），用于存放所有的Java对象。堆不同于堆栈的好处是：编译器不需要知道存储的数据在堆里存活多长时间。

**常量存储**
>常量值通常直接存放在程序代码内部。

**非RAM存储**
>数据完全存活于程序之外，不受程序的任何控制，在程序没有运行时也可以存在。其中的两个基本的例子就是**流对象和持久化对象**。在流对象中对象转化为字节流，通常被发送给另外的一台机器。在“持久化对象”中对象被存放于磁盘上，因此即使程序终止，他们依然可以保持自己的状态。这种存储方式的技巧在于：把对象转化为可以存储在其他媒介上的事物，在需要的时候可以恢复成常规的RAM对象
