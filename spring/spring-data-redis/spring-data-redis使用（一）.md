1.maven依赖配置
```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.0.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

2.spring配置
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="400" />
        <property name="maxTotal" value="100" />
        <property name="maxWaitMillis" value="1000" />
        <property name="blockWhenExhausted" value="true" />
        <property name="testOnBorrow" value="true" />

    </bean>

    <bean id="jedisConnFactory"
          class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:use-pool="true"
          p:port="6379"
          p:hostName="10.8.0.1"
          p:timeout="10000"
          p:poolConfig-ref="jedisPoolConfig"/>

    <bean id="keySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    <bean id="valueSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
    <!-- redis template definition -->
    <bean id="redisTemplate"
          class="org.springframework.data.redis.core.RedisTemplate"
          p:keySerializer-ref="keySerializer"
          p:valueSerializer-ref="valueSerializer"
          p:hashKeySerializer-ref="keySerializer"
          p:hashValueSerializer-ref="valueSerializer"
          p:enableTransactionSupport="true"
          p:connection-factory-ref="jedisConnFactory"/>

    <bean id="redisClient"
          class="com.rpc.redis.RedisClient">
        <constructor-arg ref="redisTemplate" />
    </bean>
</beans>
```

3.自己写的redis操作类，只是单个节点的增删改查
```
package com.rpc.redis;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.SetOperations;
import org.springframework.data.redis.core.ValueOperations;

import java.io.Serializable;
import java.util.concurrent.TimeUnit;

/**
 * @program: nettystudy
 * @description:
 * @author: liujiawei
 * @create: 2018-08-10 15:04
 **/
public class RedisClient {

    private static final Logger logger = LoggerFactory.getLogger(RedisClient.class);

    private RedisTemplate<String, Object> template;

    public RedisClient(RedisTemplate<String, Object> template) {
        this.template = template;
    }

    public void set(String key, Object value) throws Exception {
        template.opsForValue().set(key, value);
        logger.info("redis: set key={}, value={}", key, value);
    }

    public void set(String key, Object value, long time) throws Exception {
        template.opsForValue().set(key, value, time);
        logger.info("redis: set key={}, value={}, expireTime={}", key, value, time);
    }

    public void set(String key, Object value, long time, TimeUnit timeUnit) throws Exception {
        template.opsForValue().set(key, value, time, timeUnit);
        logger.info("redis: set key={}, value={}, expireTime={}, timeUnit={}", key, value, time, timeUnit);
    }

    public Object get(String key) throws Exception {
        logger.info("redis: get key={}", key);
        return template.opsForValue().get(key);
    }

    public void delete(String key) throws Exception {
        template.delete(key);
        logger.info("redis: delete key={}", key);
    }
}

```