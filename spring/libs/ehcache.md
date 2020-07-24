

https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-caching.html
```
33.1 Supported Cache Providers
The cache abstraction does not provide an actual store and relies on abstraction materialized by the org.springframework.cache.Cache and org.springframework.cache.CacheManager interfaces.

If you have not defined a bean of type CacheManager or a CacheResolver named cacheResolver (see CachingConfigurer), Spring Boot tries to detect the following providers (in the indicated order):

Generic
JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, and others)
EhCache 2.x
Hazelcast
Infinispan
Couchbase
Redis
Caffeine
Simple
```

cache library をアプリケーションに引き込む設定をビルドツールに記述する  
gradle の場合は以下  
```
dependencies {
// https://mvnrepository.com/artifact/org.ehcache/ehcache
compile group: 'org.ehcache', name: 'ehcache', version: '3.6.1'
}
```

設定ファイル  
ehcache.xml に設定を記載する  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <diskStore path="java.io.tmpdir"/>
    <cache
            name="getCache"
            timeToLiveSeconds="5"
            maxEntriesLocalHeap="0">
    </cache>
</ehcache>
```

application.yaml  
複数のキャッシュライブラリを使う場合に必要  
```yaml
spring:
  cache:
    type: ehcache
```

コードのメソッドに `@Cacheable` をつける
```java
@Cacheable("getCache") //このメソッドについてキャッシュを有効にする
    public String getFromCache(String key){
        try {
            Thread.sleep(3000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "End getFromCache:" + key;
    }
```

