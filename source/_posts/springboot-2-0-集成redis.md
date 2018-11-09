---
title: Springboot 2.0 - 集成redis
date: 2018-10-04 10:59:02
tags: 
- SpringBoot
- Redis
categories: 
- JAVA WEB
- SpringBoot
---



![来源网络](https://i.loli.net/2018/10/04/5bb584bc59cac.jpg)



## 序

最近在入门`SpringBoot`，然后在感慨 `SpringBoot `较于`Spring `真的方便多时，顺便记录下自己在集成`redis `时的一些想法。



<!--more-->



### 1、从springboot官网查看redis的依赖包

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



### 2、操作redis

```java
/*
   操作k-v都是字符串的
 */
@Autowired
StringRedisTemplate stringRedisTemplet;

 /*
     操作k-v都是对象的
 */
@Autowired
RedisTemplate redisTemplate;
```

redis的包中提供了两个可以操作方法，根据不同类型的值相对应选择。

两个操作方法对应的redis操作都是相同的

```java
 stringRedisTemplet.opsForValue() // 字符串
 stringRedisTemplet.opsForList() // 列表
 stringRedisTemplet.opsForSet() // 集合
 stringRedisTemplet.opsForHash() // 哈希
 stringRedisTemplet.opsForZSet() // 有序集合
```



### 3、修改数据的存储方式

在`StringRedisTemplet `中，默认都是存储字符串的形式；在`RedisTemplet`中，值可以是某个对象，而redis默认把对象序列化后存储在redis中（**==所以存放的对象默认情况下需要序列化==**）



如果需要更改数据的存储方式，如采用json来存储在redis中，而不是以序列化后的形式。

 1）自己创建一个`RedisTemplate`实例，在该实例中自己定义json的序列化格式（==**org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer**==）

```java
// 这里传入的是employee对象（employee 要求可以序列化）
Jackson2JsonRedisSerializer<Employee> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
```

2）把定义的格式放进自己定义的`RedisTemplate`实例中

```java
RedisTemplate<Object,Employee> template = new RedisTemplate<>();
template.setConnectionFactory(redisConnectionFactory);
// 定义格式
Jackson2JsonRedisSerializer<Employee> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
// 放入RedisTemplate实例中
template.setDefaultSerializer(jackson2JsonRedisSerializer);
```



参考代码：

```java
@Bean
 public RedisTemplate<Object,Employee> employeeRedisTemplate(RedisConnectionFactory redisConnectionFactory)throws UnknownHostException{
        RedisTemplate<Object,Employee> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory); 
        Jackson2JsonRedisSerializer<Employee> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(jackson2JsonRedisSerializer);
        return template;
    }
```



原理：

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    ) // 在容器当前没有redisTemplate时运行
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean // 在容器当前没有stringRedisTemplate时运行
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```



如果你自己定义了`RedisTemplate`后并添加@Bean注解，==（**要在配置类中定义**）==，那么默认的`RedisTemplate`就不会被添加到容器中，运行的就是自己定义的`ReidsTemplate`实例，而你在实例中自己定义了序列化格式，所以就会以你采用的格式定义存放在redis中的对象。



### 4、更改默认的缓冲

`springboot`默认提供基于注解的缓冲，只要在主程序类（**==xxxApplication==**）标注`@EnableCaching`，缓冲注解有

`@Cachingable`、`@CachingEvict`、`@CachingPut`,并且该缓冲默认使用的是`ConcurrentHashMapCacheManager`



当引入`redis`的starter后,容器中保存的是`RedisCacheManager` ,`RedisCacheManager`创建`RedisCache`作为缓冲组件，`RedisCache`通过操纵`redis`缓冲数据



### 5、修改redis缓冲的序列化机制

在SpringBoot中，如果要修改序列化机制，可以直接建立一个配置类，在配置类中自定义`CacheManager`，在`CacheManager`中可以自定义序列化的规则，默认的序列化规则是采用jdk的序列化



**==注：在SpringBoot 1.5.6 和SpringBoot 2.0.5 的版本中自定义CacheManager存在差异==**

参考代码：

```java
// springboot 1.x的版本
public RedisCacheManager employeeCacheManager(RedisConnectionFactory redisConnectionFactory){
    
    // 1、自定义RedisTemplate
    RedisTemplate<Object,Employee> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    Jackson2JsonRedisSerializer<Employee> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
    template.setDefaultSerializer(jackson2JsonRedisSerializer);
    
    // 2、自定义RedisCacheManager
    RedisCacheManager cacheManager = new RedisCacheManager(template);
    cacheManager.setUsePrefix(true); // 会将CacheName作为key的前缀
    
    return cacheManager;
}
```



```java
// springboot 2.x的版本

/**
 * serializeKeysWith() 修改key的序列化规则，这里采用的是StringRedisSerializer()
 * serializeValuesWith() 修改value的序列化规则，这里采用的是Jackson2JsonRedisSerializer<Employee>(Employee.class)
 * @param factory
 * @return
 */
@Bean
public RedisCacheManager employeeCacheManager(RedisConnectionFactory redisConnectionFactory) {


RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
              .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())) 
             .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer<Employee>(Employee.class)));

        RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(config).build();

        return cacheManager;
    }
```



**tip：**可以通过查看各版本的**==org.springframework.data.redis.cache.RedisCacheConfiguration==**去自定义`CacheManager.`

因为不同版本的`SpringBoot`对应的`Redis`版本也是不同的，所以要重写时可以查看官方是怎么定义`CacheManager`，才知道怎样去自定义`CacheManager`。



## 完



>   努力成为一个『不那么差』的程序员 