# Spring之ClassPathResource加载资源文件

先看Demo: 

```java
 @Test
 public void testClassPathResource() throws IOException {
     Resource res = new ClassPathResource("resource/ApplicationContext.xml");
     InputStream input = res.getInputStream();
     Assert.assertNotNull(input);
 }
```

再看内部源码：  

```java
 public ClassPathResource(String path) {
     this(path, (ClassLoader) null);
}
```

```java
public ClassPathResource(String path, ClassLoader classLoader) {
    Assert.notNull(path, "Path must not be null");
    //路径清理的工具类清理路径,具体的就不研究了，返回来之后任然是spring-config.xml
    String pathToUse = StringUtils.cleanPath(path);
    if (pathToUse.startsWith("/")) {
        pathToUse = pathToUse.substring(1);
    }
    this.path = pathToUse;
    //因没有指定classLoader所以使用的就是ClassUtils.getDefaultClassLoader()
    this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
}
```

在上面有个 StringUtils.cleanPath(path);工具类，研究一下具体的实现过程可以看源码研究

```java
/**  这段是代码是获取默认的线程的上下文类加载器的
	 * Return the default ClassLoader to use: typically the thread context
	 * ClassLoader, if available; the ClassLoader that loaded the ClassUtils
	 * class will be used as fallback.
	 * <p>Call this method if you intend to use the thread context ClassLoader
	 * in a scenario where you absolutely need a non-null ClassLoader reference:
	 * for example, for class path resource loading (but not necessarily for
	 * <code>Class.forName</code>, which accepts a <code>null</code> ClassLoader
	 * reference as well).
	 * @return the default ClassLoader (never <code>null</code>)
	 * @see java.lang.Thread#getContextClassLoader()
	 */
	public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		}
		catch (Throwable ex) {
			// Cannot access thread context ClassLoader - falling back to system class loader...
		}
		if (cl == null) {
			// No thread context class loader -> use class loader of this class.
			cl = ClassUtils.class.getClassLoader();
		}
		return cl;
	}
```

获取资源内容：  

```java
/**
 * This implementation opens an InputStream for the given class path resource.
 * @see java.lang.ClassLoader#getResourceAsStream(String)
 * @see java.lang.Class#getResourceAsStream(String)
 */
@Override
public InputStream getInputStream() throws IOException {
    InputStream is;
    if (this.clazz != null) {
        //path 不以’/'开头时默认是从此类所在的包下取资源，以’/'开头则是从ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源。
        is = this.clazz.getResourceAsStream(this.path);
    }
    else if (this.classLoader != null) {
        //默认则是从ClassPath根下获取，path不能以’/'开头，最终是由ClassLoader获取资
        is = this.classLoader.getResourceAsStream(this.path);
    }
    else {
        is = ClassLoader.getSystemResourceAsStream(this.path);
    }
    if (is == null) {
        throw new FileNotFoundException(getDescription() + " cannot be opened because it does not exist");
    }
    return is;
}
```

