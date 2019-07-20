# Collections.addAll 和collection.addall 的比较

The [Java API docs say](https://link.jianshu.com/?t=http://java.sun.com/javase/6/docs/api/java/util/Collections.html#addAll%28java.util.Collection,%20T...%29) the following about `Collections.addAll`

>The behavior of this convenience method is identical to that of c.addAll(Arrays.asList(elements)), but this method is likely to run significantly faster under most implementations.

速度上来说，`Collections.addAll`要比`collection.addAll`更快,举例来说。

```java
a)
Collection<Integer> col = new ArrayList<Integer>();
col.addAll(Arrays.asList(1, 2, 3, 4, 5));
```

```c
b)
Collection<Integer> col = new ArrayList<Integer>();
// Collections.addAll(col, Arrays.asList(1, 2, 3, 4, 5)); <-- won't compile
Collections.addAll(col, 1, 2, 3, 4, 5);
```

Let's take a closer look at the two of them:

```java
// a)
col.addAll(Arrays.asList(1, 2, 3, 4, 5));
```

Here's what happens:

1. varags + autoboxing creates Integer[]（可变参+自动装箱产生整型数组）
2. `Arrays.asList` creates a List<Integer> backed by the array（Arrays.asList）
3. addAll iterates over a Collection<Integer> using Iterator<Integer>


```java
// b)
Collections.addAll(col, 1, 2, 3, 4, 5);
```

Here's what happens:

1. varargs + autoboxing creates `Integer[]`
2. `addAll` iterates over an array (instead of an `Iterable<Integer>`)



**We can see now that `b)` may be faster because:**

- `Arrays.asList` call is skipped, i.e. no intermediary `List` is created.
- Since the elements are given in an array (thanks to varargs mechanism), iterating over them may be faster than using [`Iterator`](http://java.sun.com/javase/6/docs/api/java/util/Iterator.html).
- 由于元素在数组中给出（感谢可变参机制），迭代它们可能比使用迭代器更快

That said, unless profiling shows otherwise, the difference isn't likely to be "significant". Do not optimize prematurely. While Java Collection Framework classes may be slower than arrays, they perform more than adequately for most applications.

这就是说，除非分析为什么这样的原因，否则差异不大可能是“显着的”。 不要过早优化。 虽然Java Collection Framework类可能比数组慢，但它们对大多数应用程序的执行效果要好得多。