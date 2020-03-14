---
layout: post
title:  "스프링 캐시 - Caffeine(1 / 3)"
date:   2020-03-14 17:11:00 +0900
categories: spring cache caffeine k8s 
---

프로젝트 진행 중 **불필요한 호출을 줄이고, 성능을 향상시키고자** 캐시를 사용하기로 하였다. 

### 캐시

메모리나 디스크에 사용되었던 내용을 특별히 빠르게 접근할 수 있는 곳에 보관하고 관리함으로써 이 내용을 다시 필요로 할 때 보다 빠르게 참조할 수 있도록 하는 것

**참조되었던 data는 다시 사용되어질 가능성이 높다**는 개념을 이용하여 **다시 사용될 확률이 높은 아이들을 좀 더 빠르게 접근 가능한 저장소를 사용한다는 개념** 



# Caffeine

* 자바 캐시 구현체 중 하나로 properties, yml파일로 빠르게 설정 가능
* 인메모리(in-memory) 캐시
  * **메인 메모리의 성능을 활용하여 애플리케이션 데이터를 신속하게 처리하고 관리할 수 있음**
  * 방대한 양의 데이터를 하드디스크가 아닌 메모리에서 관리하고 실시간으로 분석할 수 있게 함으로써 데이터 처리 시간을 단축하고 빠른 의사 결정을 지원
* **로컬 캐시**
  * 느린 네트워크 요청 대신에 로컬 메모리에 저장 된 것을 바로 돌려주기 때문에 성능 향상에 도움
* **concurrentHashMap** 사용
  * Synchronized Map으로 Multi-thread환경에서 동기화 처리로 인한 문제점 보완
  * Hash Table처럼 Map전체에 Lock을 걸지 않고 **Map을 여러 조각으로 쪼개서 일부만 Lock을 거는 형태** (경쟁상황이 심할 경우 더 효율적인 성능)
* Ehcache보다 **히트율이 높음**
  * [Ehcache와 비교한 Hit Rate](https://github.com/ben-manes/caffeine/wiki/Ehcache)
* [카페인 캐시 소개](https://github.com/ben-manes/caffeine)



# Spring boot에서 Caffeine캐시 설정

**[ build.gradle ]**

```build.gradle
// caffeine cache
compile group: 'com.github.ben-manes.caffeine', name: 'caffeine', version: '2.6.2'
compile group: 'org.springframework.boot', name: 'spring-boot-starter-cache'
```

<br>

**[ application.yml ]**

```yaml
# 캐쉬 설정
spring:
  cache:
    cache-names: highlight, previewUrl#캐시이름들....
```

*  사용하고자 하는 캐시 이름을 나열해주면 됨

<br>

**[ Application.java ]**

```java
@EnableCaching
@SpringBootApplication
public class BobApplication {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

* `@EnableCaching` 을 통해 Spring의 annotation기반 Cache management를 사용할 수 있도록 해줌

<br>

**[ CaffeineCacheConfig.java ]**

```java
@Configuration
public class CaffeineCacheConfig {

    @Getter
    public enum CacheType {
        PREVIEWURL("previewUrl", 1800, 1000),
        HIGHLIGHT("highlight", 1800, 1000),
        LOWURL("lowUrl", 1800, 1000);

        CacheType(String cacheName, int expiredAfterWrite, int maximumSize) {
            this.cacheName = cacheName;
            this.expiredAfterWrite = expiredAfterWrite;
            this.maximumSize = maximumSize;
        }

        private String cacheName;
        private int expiredAfterWrite;
        private int maximumSize;
    }

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        List<CaffeineCache> caches = Arrays.stream(CacheType.values())
                .map(cache -> new CaffeineCache(cache.getCacheName(), Caffeine.newBuilder().recordStats()
                                .expireAfterWrite(cache.getExpiredAfterWrite(), TimeUnit.SECONDS)
                                .maximumSize(cache.getMaximumSize())
                                .build()
                        )
                )
                .collect(Collectors.toList());
        cacheManager.setCaches(caches);
        return cacheManager;
    }
}

© 2020 GitHub, Inc.
```

* 캐시 매니저에 카페인 캐시로 설정하여 카페인 캐시 설정을 할 수 있음
* enum 타입을 이용하여 캐시목록 관리
* **캐시마다 ttl 설정 및 관리 가능**

<br>

**[ HighlightService.java ]**

```java
@Service
public class HighlightService {
    @Autowired
    HighlightDao highlightDao;

    @Cacheable(cacheNames = "highlight")
    public Highlight getHighlight(Long clipLinkId) {
        Highlight highlight = highlightDao.getHighlight(Long clipLinkId);
        return highlight;
    }
```

* `@Cacheable(cacheNames = "highlight")` : 메소드를 호출한 결과를 캐시하겠다는 annotation
  * **cacheNames**에 지정한 캐시 이름을 적어주면 됨



# 결론

**캐시를 설정한 메소드는 여러번 호출해도 정해진 시간 내에는 캐싱 된 값을 반환하게 되었음 ^________________________^**

**그런데 생각보다 캐시의 히트율이 낮았음**🤔

### **WHY?** 

### 로컬 캐시를 썼는데 쿠버네티스로 파드를 3개를 띄웠기 때문에 로드밸런싱이 되었기 때문!!

즉, 이미 캐싱 된 데이터를 호출을 해도 로드밸런싱(ex. 라운드 로빈) 때문에 이전 파드로 요청이 가지 않기 때문에 히트율이 낮았다. 