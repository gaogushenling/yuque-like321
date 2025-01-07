# 八、Redis与SpringBoot整合

## 1、 pom.xml


在pom.xml文件中引入redis相关依赖



```xml
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- spring2.X集成redis所需common-pool2-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.6.0</version>
</dependency>


<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.9</version>
</dependency>
```



## 2、application.properties


配置redis配置



```properties
#Redis服务器地址
spring.redis.host=192.168.199.155
#Redis服务器连接端口
spring.redis.port=6379
#Redis服务器连接密码
spring.redis.password=******
#Redis数据库索引（默认为0）
spring.redis.database=0
#连接超时时间（毫秒）
spring.redis.timeout=1800000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#最大阻塞等待时间(负数表示没限制)
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=5
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```



## 3、添加redis配置类


```java
package com.atguigu.redis_springboot.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    //RedisTemplate<K,V>类的配置
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建RedisTemplate<String, Object>对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(factory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会报异常
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        //redis value 序列化方式
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //edis hash value 序列化方式
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        //使用StringRedisSerializer来序列化和反序列化redis的key值
        StringRedisSerializer redisSerializer = new StringRedisSerializer();
        //redis key 序列化方式
        template.setKeySerializer(redisSerializer);
        // redis hash key 序列化方式
        template.setHashKeySerializer(redisSerializer);

        template.afterPropertiesSet();
        return template;
    }

    //配置Redis作为Spring的缓存管理
    @Bean
    public CacheManager cacheManager(RedisTemplate<String, Object> template) {

        RedisCacheConfiguration config = RedisCacheConfiguration
                .defaultCacheConfig()
                // 设置key为String
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(template.getStringSerializer()))
                // 设置value 为自动转Json的Object
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(template.getValueSerializer()))
                //缓存数据保存1小时
                .entryTtl(Duration.ofHours(1))
                // 不缓存null
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.RedisCacheManagerBuilder
                // Redis 连接工厂
                .fromConnectionFactory(template.getConnectionFactory())
                // 缓存配置
                .cacheDefaults(config)
                // 配置同步修改或删除 put/evict
                .transactionAware()
                .build();
        return cacheManager;
    }

}
```



## 4、测试


```java
package com.atguigu.redis_springboot.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/redisTest")
public class RedisTestController {

    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping
    public String testRedis() {
        //设置值到redis
        redisTemplate.opsForValue().set("name", "jack");
        //从redis获取值
        String name = (String) redisTemplate.opsForValue().get("name");
        return name;
    }

}
```



> 更新: 2022-08-11 16:25:16  
> 原文: <https://www.yuque.com/like321/qgn2qc/ng6gf6>