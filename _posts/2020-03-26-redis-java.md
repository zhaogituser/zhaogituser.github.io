---
layout:     post
title:      redis java config
subtitle:   redis连接池、连接工厂
date:       2020-03-26
author:     jbro
header-img: img/post-bg-java.jpg
catalog: true
tags:
    - java
    - 2020年
---

## java config redis有关配置如下

> Jedis客户端配置

```java
  private JedisClientConfiguration getJedisClientConfiguration() {
    JedisClientConfiguration.JedisClientConfigurationBuilder builder = applyProperties(
            JedisClientConfiguration.builder());
    RedisProperties.Pool pool = this.redisProperties.getJedis().getPool();
    if (pool != null) {
        applyPooling(pool, builder);
    }
    return builder.build();
  }
  private JedisClientConfiguration.JedisClientConfigurationBuilder applyProperties(
          JedisClientConfiguration.JedisClientConfigurationBuilder builder) {
      if (this.redisProperties.isSsl()) {
          builder.useSsl();
      }
      if (this.redisProperties.getTimeout() != null) {
          Duration timeout = this.redisProperties.getTimeout();
          builder.readTimeout(timeout).connectTimeout(timeout);
      }
      return builder;
  }
  private void applyPooling(RedisProperties.Pool pool,
                            JedisClientConfiguration.JedisClientConfigurationBuilder builder) {
      builder.usePooling().poolConfig(jedisPoolConfig(pool));
  }

  private JedisPoolConfig jedisPoolConfig(RedisProperties.Pool pool) {
      JedisPoolConfig config = new JedisPoolConfig();
      config.setMaxTotal(pool.getMaxActive());
      config.setMaxIdle(pool.getMaxIdle());
      config.setMinIdle(pool.getMinIdle());
      if (pool.getMaxWait() != null) {
          config.setMaxWaitMillis(pool.getMaxWait().toMillis());
      }
      return config;
  }
  
```
> jedis 单机相关配置

```java
  RedisStandaloneConfiguration getStandaloneConfig() {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
        config.setHostName(this.redisProperties.getHost());
        config.setPort(this.redisProperties.getPort());
        config.setPassword(RedisPassword.of(this.redisProperties.getPassword()));
        config.setDatabase(this.redisProperties.getDatabase());
        return config;
    }
```
> **创建RedisConnectionFactory 工厂**  

```java
  public RedisConnectionFactory redisConnectionFactory(){
        JedisClientConfiguration clientConfiguration = getJedisClientConfiguration();
//        if (getSentinelConfig() != null) {
//            return new JedisConnectionFactory(getSentinelConfig(), clientConfiguration);
//        }
//        if (getClusterConfiguration() != null) {
//            return new JedisConnectionFactory(getClusterConfiguration(),
//                    clientConfiguration);
//        }
      return new JedisConnectionFactory(getStandaloneConfig(), clientConfiguration);
  }
```
## **记录 1**
学习spring boot配置时得运用，以上代码来自官网源码。其他配置看參考[spring](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc)