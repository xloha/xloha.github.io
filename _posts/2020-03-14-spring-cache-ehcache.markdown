---
layout: post
title:  "스프링 부트 캐시 - Ehcache(2 / 3)"
date:   2020-03-14 17:30:00 +0900
categories: springboot cache ehcache k8s 
---

쿠버네티스로 파드를 여러 개 띄울 시 로컬 캐시는 생각한 것 보다 캐시 히트율이 낮았기 때문에 Ehcache를 사용해보기로 했다.

Ehcache는 Spring에서 많이 사용되는 캐시인데, **나는 Ehcache를 분산 캐시 때문에 사용**하였다.

RMI방식에 멀티캐스트로 노드를 찾는 방식을 사용하였다. 


> Spring에서는 캐시 추상화를 제공해주기 때문에 cache manager나 캐시마다의 필요한 부분만 설정해주면 된다

<br/>

------

<br/>

# Ehcache

* 자바 캐시 구현체 중 하나
* **로컬 캐시**
* 경량이고 빠르다
* **⭐️분산 지원⭐️**
  * 로컬 메모리의 캐시를 다른 노드와 동기화 가능
  * 피어 자동 발견 및 RMI를 이용한 클러스터간 데이터 전송 가능
* [Ehcache 소개](https://www.ehcache.org/)

<br/>

# Spring boot에서 Ehcache캐시 설정

**[ build.gradle ]**

```
// ehcache cache
implementation 'org.springframework.boot:spring-boot-starter-cache'
implementation 'net.sf.ehcache:ehcache:2.10.3'
```

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

<br>

**[ EhcacheConfig.java ]**

```java
@Configuration
public class EhcacheConfig {
    @Bean
    public EhCacheCacheManager ehCacheCacheManager() {
        EhCacheCacheManager ehCacheCacheManager = new EhCacheCacheManager();
        ehCacheCacheManager.setCacheManager(cacheManager());
        return ehCacheCacheManager;
    }

    @Bean
    public CacheManager cacheManager() {
        return net.sf.ehcache.CacheManager.create(this.getClass().getResource("/ehcache.xml"));
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

* `Serializable` : **직렬화를 위해 인터페이스를 상속받아야함**⭐️
  * 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부 자바 시스템에서도 사용할 수 있도록 Byte형태로 데이터를 변환하는 기술(직렬화) AND Byte로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)

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

<br>

**[ ehcache.xml ]**  *( /src/main/resources/ehcache.xml )*

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <diskStore path="java.io.tmpdir" />

    <cache name="previewUrl"
           maxEntriesLocalHeap="100"
           timeToIdleSeconds="1800" timeToLiveSeconds="1800"
           memoryStoreEvictionPolicy="LFU">
        <cacheEventListenerFactory
                class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"
                properties="replicateUpdatesViaCopy=true,replicateUpdates=true" />
        <bootstrapCacheLoaderFactory
                class="net.sf.ehcache.distribution.RMIBootstrapCacheLoaderFactory"
                properties="bootstrapAsynchronously=true,
                       maximumChunkSizeBytes=50000" />
        <persistence strategy="localTempSwap" />
    </cache>

    <cache name="highlight"
           maxEntriesLocalHeap="100"
           timeToIdleSeconds="1800" timeToLiveSeconds="1800"
           memoryStoreEvictionPolicy="LFU">
        <cacheEventListenerFactory
                class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"
                properties="replicateUpdatesViaCopy=true,replicateUpdates=true" />
        <bootstrapCacheLoaderFactory
                class="net.sf.ehcache.distribution.RMIBootstrapCacheLoaderFactory"
                properties="bootstrapAsynchronously=true,
                       maximumChunkSizeBytes=50000" />
        <persistence strategy="localTempSwap" />
    </cache>

    <cacheManagerPeerProviderFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerProviderFactory"
            properties="peerDiscovery=automatic, multicastGroupAddress=230.0.0.1, multicastGroupPort=4446"/>
    <cacheManagerPeerListenerFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerListenerFactory"/>

</ehcache>
```

* `<cacheManagerPeerProviderFactory>`, `<cacheManagerPeerListenerFactory>` 를 사용해서 분산 캐시 설정
* `multicastGroupAddress=230.0.0.1, multicastGroupPort=4446` : 멀티캐스트 방식을 이용하여 한 노드의 캐시에 변화가 생기면 나머지 노드에 그 변경 내용을 전달하는 방식을 사용함
  * **노드의 캐시 간 데이터 전송은 RMI를 통해 이루어짐**
* `<cacheEventListenerFactory>` : 캐시의 변경 내역을 어떻게 통지할지 여부를 지정
* ` <bootstrapCacheLoaderFactory` : 어플리케이션 구동 시 캐시 데이터 로딩할 수 있도록 해줌

<br>

# 결론

로컬에서 도커를 이용해서 똑같은 애플리케이션을 두개 띄워놓고 테스트 했을 시에 **여러 번 호출해도 캐싱 된 데이터를 가져오기 때문에 불필요한 호출이 되지 않았음!!!**

**또한 캐싱 된 데이터에 대해 CRUD 액션을 취해도 노드 간에 캐시가 동기화 되어 같은 값을 확인 할 수 있음!!**

#### BUT

쿠버네티스로 띄운 pod 3개에서는 multicast로 노드를 못 찾는 것인지 잘 모르겠지만, 배포 환경에서 캐시가 동기화 되지 않는 문제점이 있었음🤔

또한 만약 애플리케이션을 확장한다면 파드의 개수가 더 많아질텐데 그러면 더 **많은 양의 네트워크 트래픽이 증가할 수 있음**

(Ehcache에서 multicast이외에 노드의 ip를 지정하는 방식도 사용할 수 있지만, 파드가 늘어나면 관리해야하는 ip가 많아짐)