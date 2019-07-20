# 默认标签的解析

[TOC]

## 概述

　　Spring的标签中有默认标签和自定义标签，两者的解析有着很大的不同，这次重点说默认标签的解析过程。默认标签的解析是在parseDefaultElement函数中进行的，分别对4种不同的标签（import,alias,bean和beans）做了不同处理。

　　默认标签的解析是在parseDefaultElement函数中进行的，分别对4种不同的标签（import,alias,bean和beans）做了不同处理。 

DefaultBeanDefinitionDocumentReader.java

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
       //对import标签进行处理
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
        //对alias标签进行处理
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
        //对bean标签进行处理
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
        //对beans标签进行处理
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```

## 一、对bean标签进行处理

在4中标签的解析中对bean标签的解析最为复杂也最为重要，所以我们从此标签开始深入解析。首先我们进入函数processBeanDefinition(ele, delegate)。

 DefaultBeanDefinitionDocumentReader.java

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
         // 委托BeanDefinitionDelegate类的parseBeanDefinitionElement方法进行元素解析
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
            // 当返回的bdHolder不为空的情况下若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
                 // 解析完成后需要对解析后的bdHolder进行注册，注册操作委托给了BeanDefinitionReaderUtils的registerBeanDefinition方法
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
             // 最后发出响应事件，通知相关监听器这个bean已经被加载
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```

代码的大致逻辑如下：

（1）首先委托BeanDefinitionParserDelegate类的parseBeanDefinitionElement方法进行元素解析，返回BeanDefinitionHolder类型的实例bdHolder。经过这个方法后，bdHolder实例已经包含我们配置文件的各种属性了，例如class、name、id、alias之类的属性；

（2）当返回的bdHolder不为空的情况下若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析；

（3）解析完成后，需要对解析后的bdHolder进行注册，同样，注册操作委托给了BeanDefinitionReaderUtils类的registerBeanDefinition方法；

（4）最后发出响应事件，通知相关的监听器，这个Bean已经加载完成了。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180721/A9H7ECKdba.png?imageslim)

### 1.1、解析Bean标签

下面针对上面的各种操作进行具体的分。首先我们从元素的解析及信息的提取上获取，也就是`BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);`进入到BeanDefinitionDelegate类的parseBeanDefinitionElement方法

```java
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
	}

   /**
   * 传进来的参数ele是Element
   */
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
        // 获取Bean标签的ID属性
		String id = ele.getAttribute(ID_ATTRIBUTE);
         // 获取Bean标签的Name属性的值，name中是可以指定bean的多个名字的，每个名字中间用逗            号、分号或者空格隔开
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

		List<String> aliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
            //将name属性的值通过,; 进行分割 转为字符串数字(即在配置文件中如配置多个name 在此做处理)
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
            //把namer属性名添加到别名的集合中去
			aliases.addAll(Arrays.asList(nameArr));
		}
        //如果是指定了id，beanNamem默认使用的就是id
		String beanName = id;
        // 如果ID为空 使用配置的第一个name属性作为ID
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
            //默认使用文件的第一个name属性值替换成为Id属性的值，beanName使用的是第一个属性值
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
            // 校验beanName和aliases的唯一性
            // 内部核心为使用usedNames集合保存所有已经使用了的beanName和alisa
			checkNameUniqueness(beanName, aliases, ele);
		}
 
        // 进一步解析其他所有属性到GenericBeanDefinition对象中
		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
            // 如果bean没有指定beanName 那么使用默认规则为此Bean生成beanName
			if (!StringUtils.hasText(beanName)) {
				try {
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
             //将信息封装到BeanDefinitionHolder对象中
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}
```

该方法主要处理了 `id、name、alias` 等相关属性，生成了 `beanName`，并且在重载函数 `parseBeanDefinitionElement(ele, beanName, containingBean)`方法中完成**核心的标签解析**。

接下来重点分析**parseBeanDefinitionElement(Element ele, String beanName, @Nullable BeanDefinition containingBean)也就是第40行** 看下它是如何完成标签解析操作的

### 1.2、简单的谈谈spring中bean的名字

　　提到bean的名字，就要从声明bean的地方说起。在应用spring时，有两个地方可以声明一个bean，一个是在spring的配置文件中，一个是在代码中通过Component等标注声明。 

　　代码中可以通过标注的方式来表示这个类是属于spring管理的类，这类标注有Component、Repository、Service以及Controller。它们默认没有直接指定bean的名字，所以bean的名字是spring默认生成的，这些bean的名字生成规则就是bean的类型首字母小写。**这个会则会引起一个问题，若不同的包下有两个名字相同的类，而这两个类都声明成spring的bean**，这时候就会产成冲突。因为bean的名字就是bean的唯一标示，是不允许重复的。当然我们可以在标注中指定bean的名字就解决了这个问题，比如Controller("bean9527")，这样当前Controller的名字就是"bean9527"了。 

　　前面在讲依赖注入中涉及到获取依赖bean的时候用到的方法都是getBean(String beanName)，可见知道bean的名字就可以找到相应的bean。那么bean只能有一个名字吗，当然不是，我们可以给bean取多个名字，也就是别名alias。另外，我们在配置文件中声明bean的时候，不仅可以指定bean的名字还可以指定bean的id，那么bean的名字和id之间是什么关系呢？还有一种情况就是即不指定bean的名字，也不指定bean的id，只是单单指定bean的类型，这时候bean的名字又是怎样的呢？下面通过一段配置来看看spring是怎么管理bean的名字的 

```java
    <bean class="com.test.service.Service1"/>
	<bean class="com.test.service.Service1"/>
	<bean id="service3" class="com.test.service.Service3" init-method="initMethod">
		<property name="service4" ref="service4"/>
	</bean>
	<bean name="service4" id="service42" class="com.test.service.Service4" >
		<property name="service3" ref="service3"/>
		<property name="service6" ref="service64"/>
	</bean>
	<bean id="service6" name="service61"  class="com.test.service.Service6"              	 autowire="byName"/>
	<alias name="service6" alias="service62"/>
	<alias name="service61" alias="service63"/>
	<alias name="service63" alias="service64"/>
```

　　假设在spring的配置文件中声明如上所示的bean，那么spring会怎么生成和保存这些bean的名字呢？透过现象看本质，先来看看现象是怎样的

 ![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180721/JlKI8gKFK7.png?imageslim)

　　上图的截图来自beanFactory中的singletonObjects，这个singletonObjects在AbstractBeanFactory的基类DefaultSingletonBeanRegistry中。它是一个map结构，key就是bean的名字，value就是bean本身，它保存了当前beanFactory中注册的所有的单例bean。利用图中的结果我们来一一分析一下bean的名字的由来。 

1，只指定bean的类型

​     若只指定了bean的类型，spring为bean生成名字是bean类型的全限定名加编号组成。可以看到，我们在配置文件中声明两个com.test.service.Service1类型的bean，而他们对应的名字就是com.test.service.Service1#0和com.test.service.Service1#1。居然有两个类型相同的单例bean，可见spring中单例的概念和传统的单例并不相同(**Spring的单例Bean是与其容器（ApplicationContext）密切相关的，所以在一个JVM进程中，如果有多个Spring容器，即使是单例bean，也一定会创建多个实例** )。spring中单例不是相对类型而言，而是相对于我们定义的bean。也就是说如果我们定义了一个bean，他叫“张三”，那么在spring中就只有一个张三的实例，不论何时你看到的都是他。但是我们还可以定义一个和张三一模一样的bean，只要他不叫张三就好；

2，只指定bean的id

​    在声明bean的时候，可以不指定bean的名字而是指定bean的id，这时它的id就是他的名字。如上图service3；

如果在spring环境下，如果出现多个配置ID属性值一样的bean

- 如果spring是默认设定，即可以覆盖bean定义，则根据spring配置文件加载的顺序，后面同名的bean会覆盖掉前面定义的bean配置，spring不会报错
- 如果设置不可以覆盖bean定义，则出现多个同ID的bean，则会抛出异常，停止运行。

3，同时指定bean的id，名字和类型

​    由上面的service4和service6的例子可以看出，只要是指定了bean的id，存储在singletonObjects中的名字就是这个id。如果一个bean只配置了name属性，但是没有配置ID属性，默认会`ID属性=name属性`  ；那么通过配置中的name以及类的全名就找不到了吗？bean的别名在哪里，怎么利用别名查找bean？接下来通过源码进行分析，看看事情的本质。

### 1.3、bean 节点与属性解析

```java
@Nullable
public AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, @Nullable BeanDefinition containingBean) {

    this.parseState.push(new BeanEntry(beanName));

    // 获取Bean标签的class属性
    String className = null;
    if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
        className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
    }

    // 获取Bean标签的parent属性
    String parent = null;
    if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
        parent = ele.getAttribute(PARENT_ATTRIBUTE);
    }

    try {

        // 创建用于承载属性的AbstractBeanDefinition（下面需要进行分析的代码）
        AbstractBeanDefinition bd = createBeanDefinition(className, parent);
        
        // 获取bean标签各种属性
        parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
        // 解析description标签
        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
        // 解析meta标签
        parseMetaElements(ele, bd);
        // 解析lookup-method标签
        parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        // 解析replaced-method标签
        parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
        // 解析constructor-arg标签
        parseConstructorArgElements(ele, bd);
        // 解析property标签
        parsePropertyElements(ele, bd);
        // 解析qualifier标签
        parseQualifierElements(ele, bd);

        bd.setResource(this.readerContext.getResource());
        bd.setSource(extractSource(ele));

        return bd;
    }
    catch (ClassNotFoundException ex) {
        error("Bean class [" + className + "] not found", ele, ex);
    }
    catch (NoClassDefFoundError err) {
        error("Class that bean class [" + className + "] depends on not found", ele, err);
    }
    catch (Throwable ex) {
        error("Unexpected failure during bean definition parsing", ele, ex);
    }
    finally {
        this.parseState.pop();
    }

    return null;
}
```

进一步解析其他属性和元素（元素和属性很多，所以这是一个庞大的工作量）并统一封装至 `GenericBeanDefinition` 中， 解析完成这些属性和元素之后，如果检测到 `bean` 没有指定的 `beanName`，那么便使用默认的规则为 `bean` 生成一个 `beanName`。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180721/jIee9B70cH.png?imageslim)

### 1.4、创建承载属性的实例

要解析属性首先必须要创建承载属性的实例，也就是创建GenericBeanDefinition类型的实例。`

```java
// BeanDefinitionParserDelegate.java
protected AbstractBeanDefinition createBeanDefinition(@Nullable String className, @Nullable String parentName)
        throws ClassNotFoundException {

    return BeanDefinitionReaderUtils.createBeanDefinition(
            parentName, className, this.readerContext.getBeanClassLoader());
}

public class BeanDefinitionReaderUtils {

    public static AbstractBeanDefinition createBeanDefinition(
            @Nullable String parentName, @Nullable String className, @Nullable ClassLoader classLoader) throws ClassNotFoundException {

        GenericBeanDefinition bd = new GenericBeanDefinition();
        // parentName可能为空
        bd.setParentName(parentName);

        // 如果classLoader不为空 
        // 则使用传入的classLoader同一虚拟机加载类对象 否则只记录classLoader

        if (className != null) {
            if (classLoader != null) {
                bd.setBeanClass(ClassUtils.forName(className, classLoader));
            }
            else {
                bd.setBeanClassName(className);
            }
        }
        return bd;
    }
}
```

`BeanDefinition` 是 `<bean>` 在容器中的内部表示形式，`BeanDefinition` 和 `<bean>` 是一一对应的。同时 `BeanDefinition`会被注册到 `BeanDefinitionRegistry` 中，`BeanDefinitionRegistry` 就像 `Spring` 配置信息的内存数据库。

至此 `createBeanDefinition(className, parent);` 已经说完了，而且我们也获得了 `用于承载属性的AbstractBeanDefinition`，接下来看看 `parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);` 是如何解析 `bean` 中的各种标签属性的

```java
public class BeanDefinitionParserDelegate {

    public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
            @Nullable BeanDefinition containingBean, AbstractBeanDefinition bd) {

        // ...省略详细代码，该部分代码主要就是通过 if else 判断是否含有指定的属性，如果有就 bd.set(attribute);

        return bd;
    }

}
```

`bean` 标签的完整解析到这就已经全部结束了，其中 `bean` 标签下的元素解析都大同小异，有兴趣的可以自己跟踪一下源代码看看 `qualifier、lookup-method` 等解析方式（*相对 bean 而言不复杂*）。自定义标签内容较多会在下一章详细介绍。

最后将获取到的信息封装到 `BeanDefinitionHolder` 实例中

```java
// BeanDefinitionParserDelegate.java
@Nullable
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
    // ...
    return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
}
```

------

##二、注册解析的 BeanDefinition

　　解析完成后需要对解析后的bdHolder进行注册，注册操作委托给了BeanDefinitionReaderUtils的registerBeanDefinition方法BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());

　　现在已经完成了XML文档到GenericBeanDefiniton的转化，也就是说，XML中所有的配置都在GenericBeanDefinition的实例类中找到了对应的位置。 

 ```java
//BeanDefinitionReaderUtils.java
public class BeanDefinitionReaderUtils {

    public static void registerBeanDefinition(
            BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
            throws BeanDefinitionStoreException {

        // 使用 beanName 做唯一标识符
        String beanName = definitionHolder.getBeanName();

        // 注册bean的核心代码
        registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

        // 为bean注册所有的别名
        String[] aliases = definitionHolder.getAliases();
        if (aliases != null) {
            for (String alias : aliases) {
                registry.registerAlias(beanName, alias);
            }
        }
    }

 ```

以上代码主要完成两个功能，一是使用 `beanName` 注册 `beanDefinition`，二是完成了对别名的注册 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180721/4LHck3aiak.png?imageslim)

### 2.1、BeanName 注册 BeanDefinition

对于beanDefinition的注册，是将beanDefinition直接放入了map中，以beanName为key,但不仅仅如此，代码如下： 

```java
public class DefaultListableBeanFactory {

    @Override
    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
            throws BeanDefinitionStoreException {

        Assert.hasText(beanName, "Bean name must not be empty");
        Assert.notNull(beanDefinition, "BeanDefinition must not be null");

        if (beanDefinition instanceof AbstractBeanDefinition) {
            try {

                // 注册前的最后一次校验，这里的校验不同于XML文件校验
                // 主要是对于AbstractBeanDefinition属性中的methodOverrides校验
                // 校验methodOverrides是否与工厂方法并存或者methodOverrides对于的方法根本不存在
                ((AbstractBeanDefinition) beanDefinition).validate();
            }
            catch (BeanDefinitionValidationException ex) {
                throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                        "Validation of bean definition failed", ex);
            }
        }

        BeanDefinition oldBeanDefinition;
        // 获取缓存中的 beanDefinition
        oldBeanDefinition = this.beanDefinitionMap.get(beanName);
        if (oldBeanDefinition != null) {
            // 如果缓存中存在 判断是否允许覆盖
            if (!isAllowBeanDefinitionOverriding()) {
                throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                        "Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
                        "': There is already [" + oldBeanDefinition + "] bound.");
            }
            else if (oldBeanDefinition.getRole() < beanDefinition.getRole()) {
                // e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Overriding user-defined bean definition for bean '" + beanName +
                            "' with a framework-generated bean definition: replacing [" +
                            oldBeanDefinition + "] with [" + beanDefinition + "]");
                }
            }
            else if (!beanDefinition.equals(oldBeanDefinition)) {
                if (this.logger.isInfoEnabled()) {
                    this.logger.info("Overriding bean definition for bean '" + beanName +
                            "' with a different definition: replacing [" + oldBeanDefinition +
                            "] with [" + beanDefinition + "]");
                }
            }
            else {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Overriding bean definition for bean '" + beanName +
                            "' with an equivalent definition: replacing [" + oldBeanDefinition +
                            "] with [" + beanDefinition + "]");
                }
            }
            // 如果允许覆盖，保存beanDefinition到beanDefinitionMap中
            this.beanDefinitionMap.put(beanName, beanDefinition);
        }
        else {
            // 判断是否已经开始创建bean
            if (hasBeanCreationStarted()) {
                // Cannot modify startup-time collection elements anymore (for stable iteration)
                synchronized (this.beanDefinitionMap) {

                    // 保存beanDefinition到beanDefinitionMap中
                    this.beanDefinitionMap.put(beanName, beanDefinition);

                    // 更新已经注册的beanName
                    List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                    updatedDefinitions.addAll(this.beanDefinitionNames);
                    updatedDefinitions.add(beanName);
                    this.beanDefinitionNames = updatedDefinitions;
                    if (this.manualSingletonNames.contains(beanName)) {
                        Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
                        updatedSingletons.remove(beanName);
                        this.manualSingletonNames = updatedSingletons;
                    }
                }
            }
            else {
                // 还没开始创建bean
                this.beanDefinitionMap.put(beanName, beanDefinition);
                this.beanDefinitionNames.add(beanName);
                this.manualSingletonNames.remove(beanName);
            }
            this.frozenBeanDefinitionNames = null;
        }

        if (oldBeanDefinition != null || containsSingleton(beanName)) {
            // 重置beanName对应的缓存
            resetBeanDefinition(beanName);
        }
    }

}
```

- 对 `AbstractBeanDefinition` 的校验，主要是针对 `AbstractBeanDefinition` 的 `methodOverrides` 属性的
- 对 `beanName` 已经注册的情况的处理，如果设置了不允许 `bean` 的覆盖，则需要抛出异常，否则直接覆盖
- 使用 `beanName` 作为 key，`beanDefinition` 为 Value 加入 `beanDefinitionMap` 存储
- 如果缓存中已经存在，并且该 `bean` 为单例模式则清楚 `beanName` 对应的缓存

### 2.2、注册别名

注册好了 `beanDefinition`，接下来就是注册 `alias`。注册的 `alias` 和 `beanName` 的对应关系存放在了 `aliasMap` 中，沿着类的继承链会发现 `registerAlias` 的方法是在 `SimpleAliasRegistry` 中实现的 

```java
public class SimpleAliasRegistry {

    /** Map from alias to canonical name */
    private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);

    public void registerAlias(String name, String alias) {
        Assert.hasText(name, "'name' must not be empty");
        Assert.hasText(alias, "'alias' must not be empty");
        if (alias.equals(name)) {
            // 如果beanName与alias相同的话不记录alias 并删除对应的alias
            this.aliasMap.remove(alias);
        } else {
            String registeredName = this.aliasMap.get(alias);
            if (registeredName != null) {
                if (registeredName.equals(name)) {
                    // 如果别名已经注册过并且指向的name和当前name相同 不做任何处理
                    return;
                }
                // 如果alias不允许被覆盖则抛出异常
                if (!allowAliasOverriding()) {
                    throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" + name + "': It is already registered for name '" + registeredName + "'.");
                }
            }
            // 校验循环指向依赖 如A->B B->C C->A则出错
            checkForAliasCircle(name, alias);
            this.aliasMap.put(alias, name);
        }
    }

}
```



至此，注册别名也完成了，主要完成了以下几个工作

- 如果 `beanName` 与 `alias` 相同的话不记录 `alias` 并删除对应的 `alias`
- 如果别名已经注册过并且指向的name和当前name相同 不做任何处理
- 如果别名已经注册过并且指向的name和当前name不相同 判断是否允许被覆盖
- 校验循环指向依赖 如A->B B->C C->A则出错

## 三、发送通知

通知监听器解析及注册完成 

```java
//DefaultBeanDefinitionDocumentReader.java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // Send registration event.
    getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
}
```

通过 `fireComponentRegistered` 方法进行通知监听器解析及注册完成工作，这里的实现只为扩展，当程序开发人员需要对注册 `BeanDefinition`事件进行监听时，可以通过注册监听器的方式并将处理逻辑写入监听器中，目前 `Spring` 中并没有对此事件做任何处理

其中 `ReaderContext` 是在类 `XmlBeanDefinitionReader` 中调用 `createReaderContext` 生成的，然后调用 `fireComponentRegistered()`