---
layout: post
title:  "스프링 부트 캐시 - Redis(3 / 3)"
date:   2020-03-19 23:30:00 +0900
categories: springboot cache redis  
---

배포 시 Ehcache가 원하는대로 동작하지 않기 때문에 **리모트 캐시 Redis**를 사용하여 캐시 히트율을 높이기로 하였다.

<br/>

# Spring boot에서 Redis 캐시 설정

**[ build.gradle ]**

```
// redis
implementation 'org.springframework.boot:spring-boot-starter-cache'
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

<br>

**[ application.yml ]**

```yaml
spring:
  cache:
    type: redis
  redis:
    host: #호스트 이름 작성
    port: 6379
    lettuce:
      pool:
        max-active: 10
        max-idle: 10
        min-idle: 2
    timeout: 5
```

<br>

**[ RedisConfig.java ]**

```java
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    @Value("${spring.redis.host}")
    private String redisHostName;

    @Value("${spring.redis.port}")
    private int redisPort;
    
    @Bean
    @Override
    public CacheManager cacheManager() {
        RedisCacheManager.RedisCacheManagerBuilder builder = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory());
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig()
                                                                        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                                                                        .disableCachingNullValues()
                                                                        .entryTtl(Duration.ofMinutes(20L));
        builder.cacheDefaults(configuration);

        return builder.build();
    }

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(redisHostName);
        redisStandaloneConfiguration.setPort(redisPort);
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }
}
```

<br>

**[ Highlight.java ]**

```java
@Getter
@Setter
@ToString
public class Highlight implements Serializable {
    private Integer startSec;
    private Integer endSec;
}
```

* 마찬가지로 `Serializable` 를 상속받아야 함

>  cacheManager나 캐시에 필요한 특성들 빼고 모두 같은 설정으로 사용 가능

<br>

# 결론

여러 개의 클러스터들이 **같은 리모트 캐시를 바라보기 때문**에 내가 원하던 캐시 히트율을 높일 수 있었다.

애플리케이션이 죽었다가 다시 재시작하는 경우에도 리모트 캐시이기 때문에 **초기 히트율을 높일 수 있음** 