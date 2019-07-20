# [JVM01]初体验

- 了解历史
- 内存结构
- 垃圾回收机制
- 性能监控工具
- 性能调优案例实战
- 认识类的文件结构
- 类加载机制
- 字节码执行引擎
- 虚拟机编译及运行时优化
- java线程高级

## JVM内存监控体验

打开jdk自己的内存监控工具jconsole,这个程序是在c盘的bin目录下，或者是在命令行中输入jconsole就可以打开

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180409/Cij1KH9fFC.png?imageslim)

下面我自己写一段程序来体验一下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180409/gd0FeF3l2c.png?imageslim)

当有对象new出来就会放到Eden中，当进行垃圾收集的过程中，如果Eden中的对象标记为垃圾对象，垃圾回收就会直接的清除，如果存活下来了就会放到存活区域中去（Survivor）,如果是多次存活的话可能会放到持久带中或者是元数据区域等等；

下面不停的创建对象，Eden区域就会不停的增加，每一次垃圾收集的过程中把Eden区域清除掉，放到Survivor区域中，如果长期存活的就可能会放到所谓的老年代区域中；

```java
package com.huanghe.test2;

import java.util.ArrayList;
import java.util.List;


public class JConsoleTest {
	public byte[] byte1=new byte[128 * 1024];
	
	public static void main(String[] args) {
		
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		System.out.println("start......");
		
		filler(10000);
		
		
	}

	private static void filler(int n)  {
		
		List<JConsoleTest> jList=new ArrayList<JConsoleTest>();
		
		for (int i = 0; i < n; i++) {
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			jList.add(new JConsoleTest());
		}
		
	}

}

```

会发现堆内存是不断的上升

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180409/Hbm4lhDhml.png?imageslim)

Eden的曲线如下面所示：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180409/Fkagf9iK3b.png?imageslim)

