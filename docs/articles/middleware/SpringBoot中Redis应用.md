# SpringBoot中Redis应用

## Pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.txzhe</groupId>
    <artifactId>base-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>base-redis</name>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
    </dependencies>
</project>
```

使用**@EnableCaching**开启缓存使用，可以基于全注解方式使用缓存 



**@Cacheable:** 使用该注解的方法当缓存存在时，会从缓存中获取数据而不执行方法，当缓存不存在时，会执行方法并把返回结果存入缓存中。`  一般使用在查询方法上`

**@CacheEvict：**使用该注解的方法执行时会清空指定的缓存。`  一般使用在更新或删除方法上`

**@CachePut：**使用该注解的方法每次执行时都会把返回结果存入缓存中。`  一般使用在新增方法上`，

### 数据序列化

```java
@EnableCaching
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisSerializer<Object> serializer = redisSerializer();
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(serializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(serializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    @Bean
    public RedisSerializer<Object> redisSerializer() {
        //创建JSON序列化器
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(objectMapper);
        return serializer;
    }

    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
        //设置Redis缓存有效期为1天
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer())).entryTtl(Duration.ofDays(1));
        return new RedisCacheManager(redisCacheWriter, redisCacheConfiguration);
    }

}
```

也可以直接在调用的地方通过RedisTemplate进行操作redis 

​	Jedis在实现上是直连Redis服务，多线程环境下非线程安全，除非使用连接池，为每个 RedisConnection 实例增加物理连接。 		

Lettuce是一种可伸缩，线程安全，完全非阻塞的Redis客户端，多个线程可以共享一个RedisConnection，它利用Netty NIO框架来高效地管理多个连接，从而提供了异步和同步数据访问方式，用于构建非阻塞的反应性应用程序 		

### 配置信息

```
spring:
  redis:
    lettuce:
      pool:
        max-active: 8 # 连接池最大连接数（推荐值: 大于 CPU * 2）
        max-idle: 8 #	连接池最大空闲连接数（推荐值: CPU * 2）
        min-idle: 0 # 连接池最小空闲连接
        max-wait: 5s # 连接池最大阻塞等待时间，负值表示没有限制
	spring.redis.lettuce.pool.time-between-eviction-runs: 1s # 空闲对象逐出器线程的运行间隔时间.空闲连接线程释放周期时间.
```

使用lettuce连接池，需要添加依赖

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



​	

  

  

  

  

  