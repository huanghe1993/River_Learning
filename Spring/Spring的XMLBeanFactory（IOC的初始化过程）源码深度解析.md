[TOC]

## 一、Spring的结构组成

先写一个最简单的例子：

```java

```

### 1、核心类的介绍

####1.1、DefaultListableBeanFactory 

XmlBeanFactory继承自DefaultListableBeanFactory，而DefaultListableBeanFactory是整个bean加载的核心部分，是Spring注册及加载bean的默认实现 ，而对于XmlBeanFactory与DefaultListableBeanFactory**不同的地方其实是在XmlBeanFactory中使用了自定义的XML读取器XmlBeanDefinitionReader，实现了个性化的BeanDefinitionReader读取**，DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry接口。下面是DefaultListableBeanFactory的相关类图 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/62Jc9jc6J6.png?imageslim)

XmlBeanFactory对DefaultListableBeanFactory进行了扩展，**主要用于从XML文档中读取BeanDefinition,**对于注册及获取Bean都是使用从父类DefaultListableBeanFactory继承的方法去实现，而唯独与父类不同的个性化实现就是增加了XmlBeanDefinitionReader类型的reader属性。在XmlBeanFactory中主要使用reader属性对资源文件进行读取和注册。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/2LHLIk43i3.png?imageslim)

我们进入这个类发现它被标记为了Deprecated,因为自从spring3.1后这个类就被废弃了，因为现在推荐使用的是XmlApplicationContext，这些后面我们再说，这个类还是有必要研究一下的。 

#### 1.2、XmlBeanDefinitionReader 

XML配置文件的读取时spring的重要功能，因为Spring的大部分功能都是以配置作为切入点的，那么我们可以从XmlBeanDefinitionReader中梳理一下资源文件读取，解析，及注册的大致脉络。首先我们看看各个类的功能。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/l097G56gm0.png?imageslim)

通过以上分析，我们可以梳理出整个XML配置文件读取的大致流程，在XmlBeanDifinitonReader中主要包含以下几个步骤： 

1. 通过继承自AbstractBeanDefinitionReader中的方法，来使用ResourceLoader将资源文件路径转换为对应的Resource文件。(出现在本文的第①处）
2. 通过DocumentLoader对Resource文件进行转换，将Resource文件转换为Document文件。
3. 通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析。

##二、容器基础BeanFactory

现在我们已经对Srping的容器有了一个大概的了解，尽管很多地方还很迷糊，但是不要紧，下面我们开始探讨每个步骤的详细实现。接下来我们要深入分析以下代码的实现：

```java
BeanFactory bf = new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));
```

通过XmlBeanFactory初始化时序图，我们看下上面代码的执行逻辑：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/AA5B4BmfEL.png?imageslim)

　时序图解析：

　　◆ 首先，时序图从一个测试类开始，这个测试类就是上面创建 XmlBeanFactory 那里。

　　◆ 创建 XmlBeanFactory 需要一个 Resource 对象，由于 Resource 是接口，所以我们使用其实现类 ClassPathResource 来构造 Resource 资源文件的实例对象。

　　◆ 有了 Resource 对象就可以进行 XmlBeanFactory 的初始化了，最后得到一个 BeanFactory。

那么，问题来了，什么又是 Resource呢？它是怎么对资源进行封装处理的呢？ 

###2.1、配置文件的封装

　　Spring的配置文件的读取时通过ClassPathResource进行封装的，如new ClassPathResource("beanFactoryTest.xml"),那么ClassPathResource完成了什么功能呢？ 

　　在java中，将不同来源的资源抽象成URL,通过注册不同的handler(URLStreamHander)来处理不同来源的资源的读取逻辑，一般handler的类型使用不同的前缀（协议，Protocol）来识别，如“file:”,"http:","jar："等，然而URL没有默认定义相对Classpath或ServletContext等资源的handler,虽然可以注册自己的URLStreamHandler来解析特定的URL前缀（协议），比如“classpath:”，然而这需要了解URL的实现机制，而且URL也没有提供一些基本的方法，如检查当前资源是否存在，检查当前资源是否可读等方法。因而Spring对其内部使用到的资源实现了自己的抽象结构：Resource接口来封装底层 

　　在说 Resource 之前，我们要知道一个接口：InputStreamSource，该接口封装任何能返回 InputStream 的类，比如File、Classpath下的资源、Byte、Array等。它只定义了一个方法：InputStream getInputStream() throws IOException; 该方法返回一个 InputStream 对象。

　　Resource 接口抽象了所有 Spring 内部所使用到的底层资源： File、URL、Classpath等。Resource 接口中的方法及大致作用如下图：

```java
public interface InputStreamSource {
	InputStream getInputStream() throws IOException;
}
public interface Resource extends InputStreamSource {

	/**
	 * 是否存在资源
	 */
	boolean exists();

	/**
	 * 是否可读
	 */
	boolean isReadable();

	/**
	 *是否处于打开状态
	 */
	boolean isOpen();

	/**
	 * 用于不同的资源到URL的装换
	 */
	URL getURL() throws IOException;

	/**
	 *  用于不同的资源到URI的装换
	 */
	URI getURI() throws IOException;

	/**
	 * Return a File handle for this resource.
	 * @throws IOException if the resource cannot be resolved as absolute
	 * file path, i.e. if the resource is not available in a file system
	 */
	File getFile() throws IOException;

	/**
	 * 返回资源内容的长度
	 */
	long contentLength() throws IOException;

	/**
	 * 最后一次修改的时间戳
	 */
	long lastModified() throws IOException;

	/**
	 * 用于基于当前资源创建一个相对资源
	 */
	Resource createRelative(String relativePath) throws IOException;

	/**
	 * 获取文件名
	 */
	String getFilename();

	/**
	 *在错误处理中打印信息
	 */
	String getDescription();
}
```

　　Resource接口抽象了所有Spring内部使用到的底层资源：File,URL，Classpath等。首先，它定义了3个判断当前资源状态的方法：存在性（exists）,可读性（isReadable）,是否处于打开状态（isOpen）.另外，Resource接口还提供了不同资源到URL,URI,File类型的转换，以及获取lastModified属性，文件名（不带文件信息的文件名，getFilename()）的方法。为了便于操作，Resource还提供了基于当前资源创建一个相对资源的方法：createRelative().在错误处理中需要详细地打印出错的资源文件，因而Resource还提供了getDescription()方法用于在错误处理中的打印信息。

　　对不同来源的资源文件都有相应的Resource实现：文件（FileSystemResource），Classpath资源（ClasspathResource）,URL资源（URLResource）,InputStream资源（InputStreamResource）,Byte数组（ByteArrayResource）等。相关

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/86jlkiFEBE.png?imageslim)

　　在日常的开发工作中，资源文件的加载也是经常用到的，可以直接使用Spring提供的类，比如在希望加载文件时可以使用以下代码：

```java
Resource resource = new ClassPathResource("beanFactoryTest.xml");

InputStream inputStream = resource.getInputStream();
```

得到inputStream后，我们可以按照以前的开发方式进行实现了，并且我们已经可以利用Resource及其子类为我们提供好的诸多特性。

　　有了Resource接口便可以对所有资源文件进行统一处理。至于实现，其实是非常简单的，以getInputStream为例，ClassPathResource中的实现方式便是通过class或者classLoader提供的底层方法进行调用，而对于FileSystemResource的实现其实更简单，直接使用FileInputStream对文件进行实例化。

```java
/**
*构造函数初始化
*/
public ClassPathResource(String path, ClassLoader classLoader) {
    	//判断是不是空
		Assert.notNull(path, "Path must not be null");
       　//利用StringUtils的工具类进行转换
		String pathToUse = StringUtils.cleanPath(path);
		if (pathToUse.startsWith("/")) {
			pathToUse = pathToUse.substring(1);
		}
		this.path = pathToUse;
		this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
	}

public InputStream getInputStream() throws IOException {
		InputStream is;
		if (this.clazz != null) {
            // path 不以’/'开头时默认是从此类所在的包下取资源，以’/'开头则是从ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源
			is = this.clazz.getResourceAsStream(this.path);
		}
		else {
            //所以使用getClass().getClassLoader().getResourceAsStream()来获取文件的时候不能在路径前面加上/,默认使用的是classes的根目录
			is = this.classLoader.getResourceAsStream(this.path);
		}
		if (is == null) {
			throw new FileNotFoundException(
					getDescription() + " cannot be opened because it does not exist");
		}
		return is;
	}
```

当通过Resource相关类完成了对配置文件进行封装后配置文件的读取工作就全权交给XmlBeanDefinitionReader来处理了。 

　　了解了Spring中将配置文件封装为Resource类型的实例方法后，我们就可以继续探讨XmlBeanFactory的初始化方法了，XmlBeanFactory初始化有许多方法，Spring中提供了很多的构造函数，在这里分析的是使用Resource实例作为构造函数参数的方法，代码如下： 

```java
public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
        //上一步的resource只是对path和classLoader进行了封装，这一步才是资源加载真正的实现
		this.reader.loadBeanDefinitions(resource);
	}
```

　　（**第①处**）上面函数的代码中，this.reader.loadBeanDefinitions(resource)才是资源加载的真正实现，也是我们分析的重点之一。 我们可以看到时序图中提到的XmlBeanDefinitionReader加载数据就是这里完成的，但是在XmlBeanDefinitionReader加载数据前还有一个调用父类构造函数初始化的过程：super(parentBeanFactory)，跟踪代码到父类AbstractAutowireCapableBeanfactory的构造函数中： 

AbstractAutowireCapableBeanFactory.java: 

```java
public AbstractAutowireCapableBeanFactory(BeanFactory parentBeanFactory) {
		this();
		setParentBeanFactory(parentBeanFactory);
	}

public AbstractAutowireCapableBeanFactory() {
		super();
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
	}
```

　　这里有必要提及下ignoreDependencyInterface方法。**ignoreDependencyInterface的主要功能，是忽略给定接口的自动装配功能**，那么，这样做的目的是什么呢？会产生什么样的效果呢？ 

　　举例来说，如果A中有B属性，Spring在获取A的时候会自动初始化B,但某些情况下B不需要被初始化，有种情况就是B实现了BeanNameAware接口。Spring中是这样介绍的，自动装配的时候，忽略给定的依赖接口，典型的应用是通过其他方式解析Application上下文注册依赖，类似于BeanFactory通过BeanFactoryAware进行注入或者ApplicationContext通过ApplicationContextAware进行注入。 

　　最后调用AbstractBeanFactory的 setParentBeanFactory 方法设置 BeanFactory对象。 

```java
public void setParentBeanFactory(BeanFactory parentBeanFactory) {
		if (this.parentBeanFactory != null && this.parentBeanFactory != parentBeanFactory) {
			throw new IllegalStateException("Already associated with parent BeanFactory: " + this.parentBeanFactory);
		}
		this.parentBeanFactory = parentBeanFactory;
	}
```

　　在 setParentBeanFactory 方法中有一个 if 判断，用于判断是否已经关联了 BeanFactory，如果已经关联就抛出异常。 

### 2.2、加载Bean

　　之前提到的在XmlBeanFactory构造函数中调用了XmlBeanDefinitionReader类型的reader属性提供的方法`this.reader.loadBeanDefinitions(resource)`,而这句代码则是**整个资源加载的切入点**，我们先来看看这个方法的时序图，如下图所示： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180720/glD5jjcEhJ.png?imageslim)



　　从上图可以看到，这个方法的调用引起了很大一串的工作。然而这些工作也只是在做准备工作，下面说说这里究竟在准备什么工作： 

（1）封装资源文件。当调用 loadBeanDefinitions 方法时，就会跳转到该方法中，该方法就调用了XmlBeanDefinitionReader 本类的一个重载方法，同时根据 Resource 对象创建一个EncodedResource 对象作为参数传递，使用 EncodedResource 的作用就是把 Resource 使用 EncodedResource 类进行封装。 （EncodedResource仅仅多加了一个encoding属性），具体的代码如下：

（2）获取输入流构建 inputSource。从 Resource 中获取对应的 InputStream 并创建 InputSource。 

（3）通过刚刚创建的 InputSource 对象和 Resource继续调用 doLoadBeanDefinitions()方法。 

```java
//第一步的准备工作，封装资源文件
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
}
```

　　那么EncodeResource的作用是什么呢？通过名称，我们可以大致推断这个类主要是用于对资源文件的编码进行处理。其中的主要逻辑体现在getReader()方法中，当设置了编码的时候Spring会使用相应的编码作为输入流的编码 。EncodeResource.java

```java
/**
	 * Open a <code>java.io.Reader</code> for the specified resource,
	 * using the specified encoding (if any).
	 * @throws IOException if opening the Reader failed
	 */
	public Reader getReader() throws IOException {
		if (this.encoding != null) {
			return new InputStreamReader(this.resource.getInputStream(), this.encoding);
		}
		else {
			return new InputStreamReader(this.resource.getInputStream());
		}
	}
```

当构造好encodeResource对象后，再次转入了可复用方法loadBeanDefinitions(new EncodedResource(resource)). 这个方法内部才是真正的数据准备阶段，也就是时序图描述的逻辑： 

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
		}
         //通过属性来记录以及加载过的资源文件，避免重复的加载
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
        //hashSet存储的数据是唯一的，所以添加相同的resource是会抛出异常的
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
            //第二步准备工作：获取到输入流
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
                //第二步准备工作：构建InputSource，不属于Spring,全路径org.xml.sax.InputS
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
                //第三步准备工作(真正进入到核心的部分)：调用doLoadBeanDefinitions
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
```

我们再次准备一下数据准备阶段的逻辑，首先对传入的resource参数做封装，目的是考虑到Resource可能存在编码要求的问题，其次，通过SAX读取XML文件的方式来准备InputSource对象，最后将准备的数据通过参数传入真正的核心处理部分`doLoadBeanDefinitions（inputSource,encodedResource.getResource()）`。XmlBeanDefinationReader.java: 

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
            //1、获取对XML文件的验证模式
			int validationMode = getValidationModeForResource(resource);
            //2、加载XML文件，并得到对应的Document.
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
            //3、根据返回的Document注册Bean信息
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```

上面的代码只做了三件事，每一件都是必不可少的。

1. 获取对XML文件的验证模式。
2. 加载XML文件，并得到对应的Document.
3. 根据返回的Document注册Bean信息。

这个三个步骤支撑着整个Spring容器部分的实现基础，尤其是第三部对配置文件的解析，逻辑非常复杂，下一节里面先从XML文件的验证模式讲起。

## 三、获取XML的验证模式

### 3.1、DTD与XSD验证模式的区别 

　　DTD（Document Type Definition）即文档类型定义，是一种XML约束模式语言，是XML文件的验证机制，属于XML文件组成的一部分。DTD是一种保证XML文档格式正确的有效方法，可以通过比较XML文档和DTD文件来看文档是否符合规范，元素和标签使用是否正确。一个DTD文档包含：元素的定义规则，元素间关系的对应规则，元素可使用的属性，可使用的实体或符号规则。

　　要使用DTD验证模式的时候需要在XML文件的头部声明，以下是在Spring.xml中声明的代码：

```xml-dtd
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN"
"http://www.springframework.org/dtd/spring-beans-2.0.dtd">
```



　　XML Schema语言就是XSD(XML Schemas Definition)。XML Schema描述了XML文档的结构。可以用一个指定的XML Schema 来验证某个XML文档，以检查该XML文档是否符合其要求。文档设计者可以通过XML Schema指定一个XML文档所允许的结构和内容，并可据此检查一个XML文档是否是有效的。XML Schema本身是一个XML文档，它符合XML语法结构。可以用通用的XML解析器解析它。 

　　在使用XML Schema文档对XML实例文档进行检验，除了要声明名称空间外（xmlns="http://www.springframework.org/schema/beans"）,还必须指定该名称空间做对应的XML Schema文档的存储位置。通过schemaLocation属性来指定名称空间所对应的XML Schema文档的存储位置，它包含两个部分，一部分是名称空间的URI,另一部分是名称空间所标识的XML Schema文件位置或URL地址（xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"）。 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd>
```

### 3.2、验证模式的读取

　　了解DTD与XSD后，更容易理解Spring中验证模式的提取。之前的分析锁定了，Spring通过getValidationModeForResource方法获取对应资源的验证模式`int validationMode = getValidationModeForResource(resource);`XMLBeanDefinationReader.java

```java
private int validationMode = VALIDATION_AUTO;

protected int getValidationModeForResource(Resource resource) {
        //获取验证模式
		int validationModeToUse = getValidationMode();
         //如果是手动指定了验证模式则使用指定的验证模式
		if (validationModeToUse != VALIDATION_AUTO) {
			return validationModeToUse;
		}
         //如果未使用则进行自动的检测
		int detectedMode = detectValidationMode(resource);
		if (detectedMode != VALIDATION_AUTO) {
			return detectedMode;
		}
		// Hmm, we didn't get a clear indication... Let's assume XSD,
		// since apparently no DTD declaration has been found up until
		// detection stopped (before finding the document's root tag).
		return VALIDATION_XSD;
	}
```

在 上面方法中，使用了几个常量，下图是几个常量所表示的值 

```java
public static final int VALIDATION_NONE = 0;
public static final int VALIDATION_AUTO = 1;
public static final int VALIDATION_DTD = 2;
public static final int VALIDATION_XSD = 3;
```

在往上第二张图中得知 getValidationModeForResource(resource) 方法首先判断是否手动指定(通过 setValidationMode() 方法设置验证模式)了验证模式，判断方式就是 获取 validationMode 属性进行判断。否则使用自动检测的方式，自动检测调用了 org.springframework.beans.factory.xml.XmlBeanDefinitionReader.detectValidationMode(resource) 方法（本类中的方法）实现。 

```java
//这个方法是XMLBeanDefinationReader.java的
protected int detectValidationMode(Resource resource) {
        //如果rsource处于打开就是异常的状态
		if (resource.isOpen()) {
			throw new BeanDefinitionStoreException(
					"Passed-in Resource [" + resource + "] contains an open stream: " +
					"cannot determine validation mode automatically. Either pass in a Resource " +
					"that is able to create fresh streams, or explicitly specify the validationMode " +
					"on your XmlBeanDefinitionReader instance.");
		}

		InputStream inputStream;
		try {
			inputStream = resource.getInputStream();
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"Unable to determine validation mode for [" + resource + "]: cannot open InputStream. " +
					"Did you attempt to load directly from a SAX InputSource without specifying the " +
					"validationMode on your XmlBeanDefinitionReader instance?", ex);
		}

		try {
            //这一句才是真正的检测代码，调用的是validationModeDetector
			return this.validationModeDetector.detectValidationMode(inputStream);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("Unable to determine validation mode for [" +
					resource + "]: an error occurred whilst reading from the InputStream.", ex);
		}
	}
```

调用XmlValidationModeDetector的validationModeDectector方法进行检测：

```java
	public int detectValidationMode(InputStream inputStream) throws IOException {
		// Peek into the file to look for DOCTYPE.查看文件以查找DOCTYPE
		BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
		try {
			boolean isDtdValidated = false;
			String content;
			while ((content = reader.readLine()) != null) {
				content = consumeCommentTokens(content);
                //如果读取的是空行或空格则跳过
				if (this.inComment || !StringUtils.hasText(content)) {
					continue;
				}
                //判断读取的内容中是否包含“DOCTYPE”
				if (hasDoctype(content)) {
					isDtdValidated = true;
					break;
				}
                //读取到<(开始符号)，验证模式一定是在开始符号之前的
				if (hasOpeningTag(content)) {
					// End of meaningful data...
					break;
				}
			}
            //如果包含“DOCTYPE”就返回2，否则返回3
			return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
		}
		catch (CharConversionException ex) {
			// Choked on some character encoding...
			// Leave the decision up to the caller.
			return VALIDATION_AUTO;
		}
		finally {
			reader.close();
		}
	}
```

　　上面获取验证模式部分需要根据 DTD 和 XSD来进行理解，因为获取验证模式就是根据两种验证模式使用方法来的。Spring 检测验证模式的方法就是判断是否包含 DOCTYPE，如果包含就是 DTD，否则就是 XSD（这一点从一行注释就是可以看出：查看文件以查找 DOCTYPE），这一点从上图方法中可以很容易的看出。到这里，获取验证模式就讲解完了。

### 3.3、获取Document

　　上面我们知道了在获取 Document 之前要先获取 XML 验证模式。下面我们就来看看 Spring 中是怎么获取 Document 的。在上面 （doLoadDocument() 方法）图中我们看见 doLoadDocument 方法调用了本类 documentLoader 的 loadDocuemnt() 方法。`Document doc = this.documentLoader.loadDocument(      inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());`documentLoader 定义如下： 

```java
private DocumentLoader documentLoader = new DefaultDocumentLoader();
```

　　DocumentLoader 是一个接口，所以使用其实现类 DefaultDocumentLoader； 先来看看 DefaultDocumentLoader 类中的 loadDocument() 方法吧 ！

```java
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
			ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

		DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
		if (logger.isDebugEnabled()) {
			logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
		}
		DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
		return builder.parse(inputSource);
	}
```

上面这段代码就是基本的 通过 SAX 解析 XML，这里算是基本步骤了。首先创建 DocumentBuilderFacotry 对象，再通过 DocumentBuilderFactory 创建 DocumentBuilder，然后解析 inputSource 来返回 Document 对象。这里涉及到了 XML 解析相关知识 

**EntityResolver用法：** 

在loadDocument方法中涉及一个参数EntityResolver，何为EntitiResolver.

如果SAX应用程序需要实现自定义处理外部实体，则必须实现此接口并使用setEntityResolver方法向SAX驱动器注册一个实例。也就是说，对于解析一个XML,SAX首先读取该XML文档上的声明，根据声明去寻找相应的DTD定义，以便对文档进行一个验证。默认的寻找规则，即通过网络（实现上就是声明的DTD的URL地址）来下载相应的DTD声明，并进行认证。下载的过程漫长，而且当网络中断或不可用的时候，这里会报错，就是因为相应的DTD声明没有被找到的原因。

EntityResolver的作用是  项目本身就可以提供一个如何寻找DTD声明的方法，即由程序来实现寻找DTD声明的过程，比如我们将DTD文件放到项目中某处，在实现时直接将此文档读取并返回给SAX即可。这样就避免了通过网络来寻找相应的声明。

## 四：解析及注册BeanDefinitions

文档转换为Document之后，接下来就是提取及注册Bean就是最重要的内容了。继续上面的分析，当程序已经拥有XML文档文件的Document实例对象时，就会被引入下面的方法

在上面 (doLoadBeanDefinitions() 方法) 图中我们知道 获取到了 Document 后，就执行下面这行代码：

`return registerBeanDefinitions(doc, resource);`也就是 继续调用` registerBeanDefinitions(doc, resource)` 方法。

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    //使用DefaultBeanDefinitionDocumentReader实例化BeanDefinitionDocumentReader
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
        //将环境变量设置在其中
		documentReader.setEnvironment(this.getEnvironment());
        //记录统计前的BeanDefinition的加载的个数
		int countBefore = getRegistry().getBeanDefinitionCount();
        //加载及注册Bean
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    　　　//记录本次加载的BeanDefinition的个数
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

　　第一行代码就是创建 BeanDefinitionDocumentReader，BeanDefinitionDocumentReader 是接口，而实例化是在 createBeanDefinitionDocumentReader() 方法中完成的，而通过执行此方法后，BeanDefinitionDocumentReader 真正的类型就是 DefaultBeanDefinitionDocumentReader (它的实现类)了。第三行就是加载、注册 Bean了，由于 BeanDefinitionDocumentReader 是接口，所以我们来到 DefaultBeanDefinitionDocumentReader 类中的 registerBeanDefinitions() 方法。

DefaultBeanDefinitionDocumentReader.java

```java
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;

		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement();
        //重要的目的之一就是提取root,再将root作为参数继续BeanDefination 
		doRegisterBeanDefinitions(root);
	}
```

终于到了最后的核心逻辑底部doRegisterBeanDefinitions(root);如果说以前一直是XML的加载解析的准备阶段，**那么doRegisterBeanDefinitions算是真正的解析，核心的部分正式的开始**：

DefaultBeanDefinitionDocumentReader.java

```java

public static final String PROFILE_ATTRIBUTE = "profile";

protected void doRegisterBeanDefinitions(Element root) {
        //获取根结点下属性为“profile”的值，通过名称获取属性
		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
        //判断是否为空,profile的环境必须不能是空，profile表示的是环境（develop/product）
		if (StringUtils.hasText(profileSpec)) {
			Assert.state(this.environment != null, "environment property must not be null");      //这个函数用于将给定的字符串按照给定的分隔符分隔成字符串数组，这里就是 把nameAttr               按照“,;”分隔开
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
            //看spring.profiles.active环境变量中是否有该属性，如果没有则不加载下面的标签
			if (!this.environment.acceptsProfiles(specifiedProfiles)) {
				return;
			}
		}
        //专门的处理解析
         BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createHelper(readerContext, root, parent);

        //解析前处理、留给子类实现(模板设计模式)
		preProcessXml(root);
        //解析bean definition
		parseBeanDefinitions(root, this.delegate);
        //解析后处理、留给子类实现
		postProcessXml(root);

		this.delegate = parent;
	}
```

　　继续分析代码逻辑：首先程序会获取beans节点是否定义了profile属性，如果定义了则会需要到环境变量（web.xml）中去寻找，所以这里首先断言environment不可能为空；因为profile是可以同时指定多个的，需要程序对其拆分，并解析每个profile是都符合环境变量中所定义的，不定义则不会浪费性能去分析。

　　在DefaultBeanDefinitionDocumentReader处理Document元素时,将Document文档内元素具体解析工作委托给BeanDefinitionParserDelegate类来处理,默认BeanDefinitionParserDelegate会处理”<http://www.springframework.org/schema/beans>“命名空间下元素及其属性, 查看源码可以看到BeanDefinitionParserDelegate下面定义了一堆元素和属性名称,这些元素和属性名称分别可以在类中找到处理方法. 

 　　处理了 profile 就可以开始进行 XML的读取了，下面看看的方法`parseBeanDefinitions(root, this.delegate)`。 

DefaultBeanDefinitionReader.java

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
                         //对默认的标签进行处理
						parseDefaultElement(ele, delegate);
					}
					else {
                        //对自定义标签的解析
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```

在 Spring 的 XML文件中可以使用默认的 Bean声明，也可以自定义。所以 Spring 针对不同的 Bean 声明做了不同的处理。这段代码还比较清晰，在Spirng的配置里面有两大类Bean声明，一个是默认的，如：

`<bean id="test" class="test.TestBean">`

另一类就是自定义的，如：

`<tx:annotation-driven>`

两种方式解析差别还挺大的，如果采用Spring默认配置，Spring自然知道怎么做，而如果使用自定义的方式需要增加一些接口和配置了。 



![1532138656518](../../../../AppData/Local/Temp/1532138656518.png)

