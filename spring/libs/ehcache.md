# Cache について

Spring Boot で Cache を使う場合のまとめ

## 対象

公式ドキュメントを参照
サポートしているライブラリは以下

https://docs.spring.io/spring-boot/docs/2.2.9.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider
```
12.1. Supported Cache Providers
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

## 設定方法

### ビルドツールへの設定

cache library をアプリケーションに引き込む設定をビルドツールに記述する  
gradle の場合は以下  
```
dependencies {
// https://mvnrepository.com/artifact/org.ehcache/ehcache
compile group: 'org.ehcache', name: 'ehcache', version: '3.6.1'
}
```

### 設定ファイル  

- ehcache の場合  
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

### Spring-Boot 用の設定  

Spring Boot 側の設定  
使用するライブラリを Spring Boot に教えるための設定  
複数のキャッシュライブラリを使う場合に必要  

- application.yaml    
```yaml
spring:
  cache:
    type: ehcache
```

### 実装  

コードのメソッドに `@Cacheable` をつける
```java
@Component
class SampleCache {

     //このメソッドについてキャッシュを有効にする
    @Cacheable("getCache")
    public String getFromCache(String key){
        try {
            Thread.sleep(3000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "End getFromCache:" + key;
    }
}
```

