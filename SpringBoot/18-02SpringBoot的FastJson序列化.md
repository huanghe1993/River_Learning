# springboot使用FastJson替换Redis的默认序列方式实现对象的序列化，及autoType is not support的解决办法

自定义Redis的序列化方式需要实现 RedisSerializer<T> 这个接口

```java

public interface RedisSerializer<T> {
 
	@Nullable
	byte[] serialize(@Nullable T t) throws SerializationException;
 
	@Nullable
	T deserialize(@Nullable byte[] bytes) throws SerializationException;
}

```

FastJson序列化器实现：

```java

public class FastJson2JsonRedisSerializer <T> implements RedisSerializer<T> {
 
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
 
    private Class<T> clazz;
 
    public FastJson2JsonRedisSerializer(Class<T> clazz) {
        super();
        this.clazz = clazz;
    }
 
    @Override
    public byte[] serialize(T t) throws SerializationException {
        if (t == null) {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }
 
    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length <= 0) {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);
        return (T) JSON.parseObject(str, clazz);
    }
}
```

设置RedisTemplate及序列化方式

```java
@Bean
public RedisSerializer<Object> fastJson2JsonRedisSerializer() {
    return new FastJson2JsonRedisSerializer<Object>(Object.class);
}
 
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisCF) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
    redisTemplate.setConnectionFactory(redisCF);
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(fastJson2JsonRedisSerializer());
    redisTemplate.afterPropertiesSet();
    return redisTemplate;
}
```

注意事项：

2017年3月15日，fastjson官方发布安全升级公告，该公告介绍fastjson在1.2.24及之前的版本存在代码执行漏洞，当恶意攻击者提交一个精心构造的序列化数据到服务端时，由于fastjson在反序列化时存在漏洞，可导致远程任意代码执行。自1.2.25及之后的版本，禁用了部分autotype的功能，也就是@type这种指定类型的功能会被限制在一定范围内使用。而由于反序列化对象时，需要检查是否开启了autotype。所以如果反序列化检查时，autotype没有开启，就会报错。所以采用上述序列化方式时需要在项目配置fastjson时配置autotype相关内容本文选择在代码中配置其他方式请参考：enable_autotype

```java
@Bean
public HttpMessageConverters fastJsonHttpMessageConverters() {
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
    fastConverter.setFastJsonConfig(fastJsonConfig);
    //必须加否则会报com.alibaba.fastjson.JSONException: autoType is not sup这个错        
    ParserConfig.getGlobalInstance().addAccept("需要序列化对象的包");
    HttpMessageConverter<?> converter = fastConverter;
    return new HttpMessageConverters(converter);
}
```

到此完成配置，项目中应用如下：

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;
	
@RequestMapping(value = "set")
public void set() {
    UserInfo user = new UserInfo(1, "张三", "18758974838", new Date());
    redisTemplate.opsForValue().set("user", user);
    System.out.println(redisTemplate.opsForValue().get("user"));
}
```

