# SpringBoot和缓存

【概要】：

缓存比如说可以存放商品信息、新闻内容、商品内容，数据库的查询是操作的是磁盘上进行查找，磁盘上的查找是非常的耗时的，我们需要把我们经常使用的热点数据放在缓存中，缓存是基于内存的，所以来说，速度是非常的快的,手机发送的验证码在缓存中，这个时候不太可能把验证码放在数据库中

***

## JSR107缓存规范

Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry 和 Expiry。

- CachingProvider定义了创建、配置、获取、管理和控制多个**CacheManager**。一个应用可以在运行期访问多个CachingProvider。

- CacheManager（**缓存管理器**）定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。

- Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。

- Entry是一个存储在Cache中的key-value对。

- Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

![mark](http://www.hhaxmm.cn/blog/20190116/Ja4Qguoi5jWX.png)

这里的的CacheManager 和Cache类似于数据库连接池和数据库连接一样，从缓存管理器中获取缓存进行使用。

## Spring缓存抽象

Spring从3.1开始定义了`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`接口来统一不同的缓存技术；并支持使用`JCache（JSR-107）`注解简化我们开发；

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
- Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等;

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| :------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                        |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存。当删除数据的时候把缓存中的数据也删除               |
| @CachePut      | 保证方法被调用，又希望结果被缓存。方法总是会被调用的，而且结果是会被更新的 |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

## 开启基于注解的缓存

```java
@SpringBootApplication
@MapperScan("com.huanghe.springboot.mapper")
//开启基于注解的缓存
@EnableCaching
public class SpringBoot06DataMybatisApplication {

	public static void main(String[] args)
	{
		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
	}
}
```

![mark](http://www.hhaxmm.cn/blog/20190116/Jtbh9RhKbDOR.png)

key的SPEL表达式：

![1547630326233](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1547630326233.png)

```java
 /**
     * 从缓存中获取数据
     *
     * CacheManager管理多个Cache组件，对缓存的CRUD的操作是在Cache组件中，每一个缓存组件有自己的	 *																	唯一的一个名字
     *@Cacheable几个属性：
     *   cacheNames/value:缓存组件的名称，将方法的结果放在哪个缓存中，是数组的形式，可以多个
     *   key:缓存数据的时候使用的key,默认是使用的是方法参数的值；值：方法的返回值
     *   可以使用SPEL表达式 “#id”：取出id的值作为key,等同于”#a0“,"",#p0","#root.args[0]"
     *   keyGenerator:key的生成器；可以自己指定生成器的id
     *          key和keyGenerator二选一的使用
     *   cacheManager/cacheResolver(二选一):缓存管理器，可以自己进行指定缓存管理器
     *   condition：指定符合条件的情况下才会缓存，eg:condition="#id>1"当id大于1才缓存
     *   unless:否定缓存，当unless的条件为true不会进行缓存，比如unless ="#result == null"
     *          当结果不为空的时候才进行缓存：unless ="#result == null"
     *   sync:是否使用异步的形式，异步模式unless不支持
     *
     * @param id
     * @return
     */
    @Cacheable(cacheNames = "emp",key = "#id",condition="#id>1")
    public Employee getEmp(Integer id) {
        Employee employee = employeeMapper.getEmpById(id);
        return employee;
    }
	
	/**
	*  清除缓存，按照key进行删除
	*  beforeInvocation = false:缓存的清除是否在方法之前执行，默认是方法执行之后执行
	*  方法执行之前清除缓存：方法出现错误也是会执行的
	*/
	@CacheEvict(cacheNames="emp",key="#id")
    public void deleteEmp(Integer id){
    	employeeMapper.deleteEmpById(id);
    }

	/**
	*@Caching :可以指定多个缓存规则
	*/
	@Caching(
        cacheable={
            @cacheable(value='emp',key="#lastName")
        },
        put = {
            @CachePut(value="emp",key='#result.id')
            @CachePut(value="emp",key='#result.email')
        }
     )
	 public Employee getEmpByLastName(String lastName) {
         employeeMapper.getEmpByLastName(lastName);
     }

```

## 缓存的工作原理

先从缓存的自动的配置类入手查看缓存的原理，CacheAutoConfiguration 类就是缓存的自动的配置类

```java
@Configuration
//类路径下有CacheManager类，才会生效
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
//类路径下有CacheManage的Bean就不生效，如果没有就会执行初始化Bean
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
//让配置类生效
@EnableConfigurationProperties(CacheProperties.class)
//在指定的配置类初始化前加载
@AutoConfigureBefore(HibernateJpaAutoConfiguration.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
		RedisAutoConfiguration.class })
// 把该类（配置）的bean导入到这个配置类中
// org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
// org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
// org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
// org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
// org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
// org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
// org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
// org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
// org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
// org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public CacheManagerCustomizers cacheManagerCustomizers(
			ObjectProvider<List<CacheManagerCustomizer<?>>> customizers) {
		return new CacheManagerCustomizers(customizers.getIfAvailable());
	}

	@Bean
	public CacheManagerValidator cacheAutoConfigurationValidator(
			CacheProperties cacheProperties, ObjectProvider<CacheManager> cacheManager) {
		return new CacheManagerValidator(cacheProperties, cacheManager);
	}

	@Configuration
	@ConditionalOnClass(LocalContainerEntityManagerFactoryBean.class)
	@ConditionalOnBean(AbstractEntityManagerFactoryBean.class)
	protected static class CacheManagerJpaDependencyConfiguration
			extends EntityManagerFactoryDependsOnPostProcessor {

		public CacheManagerJpaDependencyConfiguration() {
			super("cacheManager");
		}

	}

	/**
	 * Bean used to validate that a CacheManager exists and provide a more meaningful
	 * exception.
	 */
	static class CacheManagerValidator implements InitializingBean {

		private final CacheProperties cacheProperties;

		private final ObjectProvider<CacheManager> cacheManager;

		CacheManagerValidator(CacheProperties cacheProperties,
				ObjectProvider<CacheManager> cacheManager) {
			this.cacheProperties = cacheProperties;
			this.cacheManager = cacheManager;
		}

		@Override
		public void afterPropertiesSet() {
			Assert.notNull(this.cacheManager.getIfAvailable(),
					"No cache manager could "
							+ "be auto-configured, check your configuration (caching "
							+ "type is '" + this.cacheProperties.getType() + "')");
		}

	}

	/**
	 * {@link ImportSelector} to add {@link CacheType} configuration classes.
	 * 缓存的配置类
	 */
	static class CacheConfigurationImportSelector implements ImportSelector {

		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
			CacheType[] types = CacheType.values();
			String[] imports = new String[types.length];
			for (int i = 0; i < types.length; i++) {
				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
			}
			return imports;
		}

	}

}

```

```xml
原理：
* 1、自动配置类：CacheAutoConfiguration.java
* 2、缓存的配置类：
 *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration
 * 3、那个配置类默认是生效的呢
 *     SimpleCacheConfiguration
 *  debug =true  # 打开自动配置报告 
```

  SimpleCacheConfiguration的源码如下：

```java
@Configuration
//当容器中没有指明的类，当容器中有其他的CacheManager就不会创建默认的
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class SimpleCacheConfiguration {

	private final CacheProperties cacheProperties;

	private final CacheManagerCustomizers customizerInvoker;

	SimpleCacheConfiguration(CacheProperties cacheProperties,
			CacheManagerCustomizers customizerInvoker) {
		this.cacheProperties = cacheProperties;
		this.customizerInvoker = customizerInvoker;
	}

	@Bean //1:创建CacheManage
	public ConcurrentMapCacheManager cacheManager() {
		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
        //从缓存的配置文件中获取到缓存的名称：prefix = "spring.cache" cacheNames，可以在配置文件中配置缓存的名称
		List<String> cacheNames = this.cacheProperties.getCacheNames();
        //判断缓存是否为空，如果不为空的情况下就在缓存管理器cacheManager中设置缓存的名称
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}
}
```

给容器中注册了一个CacheManager:

```java
public class ConcurrentMapCacheManager implements CacheManager, BeanClassLoaderAware {
	//使用的是ConcurrentHashMap来完成底层的系列操作的
	private final ConcurrentMap<String, Cache> cacheMap = new ConcurrentHashMap<String, Cache>(16);

	private boolean dynamic = true;

	private boolean allowNullValues = true;

	private boolean storeByValue = false;

	private SerializationDelegate serialization;


	/**
	 * Construct a dynamic ConcurrentMapCacheManager,
	 * lazily creating cache instances as they are being requested.
	 */
	public ConcurrentMapCacheManager() {
	}

	/**
	 * Construct a static ConcurrentMapCacheManager,
	 * managing caches for the specified cache names only.
	 */
	public ConcurrentMapCacheManager(String... cacheNames) {
		setCacheNames(Arrays.asList(cacheNames));
	}
    ....
    //从缓存中获取值
    @Override
	public Cache getCache(String name) {
        //从map中通过key获取到value
		Cache cache = this.cacheMap.get(name);
		if (cache == null && this.dynamic) {
			synchronized (this.cacheMap) {
                //如果获取到的缓存为空的情况下，就会创建缓存组件
				cache = this.cacheMap.get(name);
				if (cache == null) {
					cache = createConcurrentMapCache(name);
                    //调用put方法把缓存放入到一个map中去
					this.cacheMap.put(name, cache);
				}
			}
		}
		return cache;
	}
    
    //创建缓存组件
    protected Cache createConcurrentMapCache(String name) {
		SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
		return new ConcurrentMapCache(name, new ConcurrentHashMap<Object, Object>(256),
				isAllowNullValues(), actualSerialization);

	}
    
}
```

创建的缓存组件为ConcurrentMapCache：(它就是缓存，存放数据的地方)

```java
public class ConcurrentMapCache extends AbstractValueAdaptingCache {

	private final String name;

	private final ConcurrentMap<Object, Object> store;

	private final SerializationDelegate serialization;


	/**
	 * Create a new ConcurrentMapCache with the specified name.
	 * @param name the name of the cache
	 */
	public ConcurrentMapCache(String name) {
		this(name, new ConcurrentHashMap<Object, Object>(256), true);
	}
    
    //构造函数完成创建缓存的任务
    protected ConcurrentMapCache(String name, ConcurrentMap<Object, Object> store,
			boolean allowNullValues, SerializationDelegate serialization) {

		super(allowNullValues);
		Assert.notNull(name, "Name must not be null");
		Assert.notNull(store, "Store must not be null");
		this.name = name;
		this.store = store;
		this.serialization = serialization;
	}
    
    //在缓存中获取key对应的数据value，key在生成的时候调用了Keygenerate生产的，默认使用的是         simpleKeyGenerate
    @Override
	protected Object lookup(Object key) {
		return this.store.get(key);
	}
    
    //存放key对应的value
    @Override
	public void put(Object key, Object value) {
		this.store.put(key, toStoreValue(value));
	}

```

key的生成的策略：

- 如果没有参数使用的是SimpleKey对象
- 有一个参数，key就是参数的值
- 多个参数，就是包装的SimpleKey对象

```java
/**
	 * Generate a key based on the specified parameters.
	 */
	public static Object generateKey(Object... params) {
		if (params.length == 0) {
			return SimpleKey.EMPTY;
		}
		if (params.length == 1) {
			Object param = params[0];
			if (param != null && !param.getClass().isArray()) {
				return param;
			}
		}
		return new SimpleKey(params);
	}
```

运行的流程：

- 方法执行之前会先查询Cache组件（缓存组件）是不是存在的，按照cacheNames指定的名称获取（CacheManager）先获取相应的缓存，第一次获取缓存的时候没有的情况下就会自动的创建缓存
- 去Cache中查找缓存的内容，使用的是一个key,默认就是方法的参数；key在生成的时候调用了Keygenerate生产的，默认使用的是 SimpleKeyGenerator
- 没有查到就调用目标方法
- 将目标方法的返回值放到缓存中执行

SimpleKeyGenerato生成Key的策略是如下所示的：

```java
public static Object generateKey(Object... params) {
        if (params.length == 0) { //参数为0时生成SimpleKey的对象
            return SimpleKey.EMPTY;
        } else {
            if (params.length == 1) {//参数只有一个的时候 key为参数
                Object param = params[0];
                //多个参数 key=SimpleKey包装的多个对象
                if (param != null && !param.getClass().isArray()) {
                    return param;
                }
            }
            return new SimpleKey(params);
        }
```

核心：

1. 使用CacheManager[ConcurrentMapCacheManager]按照名字得到Cache[ConcurrentMapCache]组件
2. key是使用SimpleKeyGenerato生成

## 整和Redis作为缓存

当我们在项目中引入JedisConnection、Jedis、RedisOperations等类的时候，RedisAutoConfiguration就会起作用，这个配置类为我们配置了RedisTemplate,作用是用来操作redis的

```java
@Configuration
@ConditionalOnClass({JedisConnection.class, RedisOperations.class, Jedis.class})
@EnableConfigurationProperties({RedisProperties.class})
public class RedisAutoConfiguration {
       public RedisAutoConfiguration() {}
       // 操作redis的
       public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
            RedisTemplate<Object, Object> template = new RedisTemplate();
            template.setConnectionFactory(redisConnectionFactory);
            return template;
        } 
    	// 操作redis里面字符串类型的
        @Bean
        @ConditionalOnMissingBean({StringRedisTemplate.class})
        public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
            StringRedisTemplate template = new StringRedisTemplate();
            template.setConnectionFactory(redisConnectionFactory);
            return template;
        }
}
```

**Redis作为缓存的原理**：

由我们上面得到的缓存的配置类开启的顺序是如下所示的：

```xml
 *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration
```

当我们引入redis的时候是不会开启SimpleCacheConfiguration,因为SimpleCacheConfiguration的条件注解可知

```java
@Configuration
//当没有CacheManager（缓存管理器），才会配置
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class SimpleCacheConfiguration {
```

当没有缓存管理器才会使用SimpleCacheConfiguration，现在是使用redis作为缓存管理器

```java
@Configuration
@AutoConfigureAfter({RedisAutoConfiguration.class})
@ConditionalOnBean({RedisTemplate.class})
@ConditionalOnMissingBean({CacheManager.class})
@Conditional({CacheCondition.class})
class RedisCacheConfiguration {
    private final CacheProperties cacheProperties;
    private final CacheManagerCustomizers customizerInvoker;

    RedisCacheConfiguration(CacheProperties cacheProperties, CacheManagerCustomizers customizerInvoker) {
        this.cacheProperties = cacheProperties;
        this.customizerInvoker = customizerInvoker;
    }
	//给容器中存放缓存管理器
    @Bean
    public RedisCacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        cacheManager.setUsePrefix(true);
        List<String> cacheNames = this.cacheProperties.getCacheNames();
        if (!cacheNames.isEmpty()) {
            cacheManager.setCacheNames(cacheNames);
        }

        return (RedisCacheManager)this.customizerInvoker.customize(cacheManager);
    }
}
```

- 引入redis的starter，容器中保存的是RedisCacheManager
- RedisCacheManager帮我们创建RedisCache来作为缓存组件，RedisCache通过操作redis缓存数据的
- 默认保存数据k-v都是Object,利用序列化保存；如何保存json
  - 引入redis的starter,cacheManager变为RedisCacheManager
  - RedisCacheManager操作redis的时候是通过redisTemplate进行操作的
  - redisTemplate<Object,Object>默认使用的jdk序列化机制
- 自定义CacheManager

