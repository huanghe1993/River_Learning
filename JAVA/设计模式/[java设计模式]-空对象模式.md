# java各种类型如何判断为空

首先是见的最多的就是String的判断是否为空？

```java
方法一: 最多人使用的一个方法, 直观, 方便, 但效率很低:

                     if(s == null || s.equals(""));
					 if(s == null || s.trim().equals(""));//去除空格
方法二: 比较字符串长度, 效率高, 是我知道的最好一个方法:

                     if(s == null || s.trim().length() == 0);
方法三: Java SE 6.0 才开始提供的方法, 效率和方法二几乎相等, 但出于兼容性考虑, 推荐使用方法二.

                     if(s == null || s.isEmpty());

方法四: 这是一种比较直观,简便的方法,而且效率也非常的高,与方法二、三的效率差不多:

                     if (s == null || s == "");

 

注意:s == null 是有必要存在的.

　　如果 String 类型为null, 而去进行 equals(String) 或 length() 等操作会抛出java.lang.NullPointerException.

　　并且s==null 的顺序必须出现在前面，不然同样会抛出java.lang.NullPointerException.
```



其次是集合的判断是否为空？

```java
if(list!=null && !list.isEmpty()){
　　　//不为空的情况
}else{
　　　//为空的情况
}
```

判断数组是否为空？

```java
/**
     * 判断输入的字节数组是否为空
     * @return boolean 空则返回true,非空则flase
     */
    public static boolean isEmpty(byte[] bytes){
        return null==bytes || 0==bytes.length;
    }
    
```