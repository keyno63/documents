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
```groovy
dependencies {
  compile "org.springframework.boot:spring-boot-starter-cache"
  // https://mvnrepository.com/artifact/org.ehcache/ehcache
  compile "org.ehcache:ehcache:3.6.1"
  
  compile "com.github.ben-manes.caffeine:caffeine"
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

- caffeine の場合  
application.yaml に設定を記述する
```yaml
spring:
  cache:
    cache-names: HistoryCache
    caffeine:
      spec: maximumSize=100, expireAfterWrite=60s
```
詳しい設定辺りは以下
https://www.javadoc.io/doc/com.github.ben-manes.caffeine/caffeine/2.2.0/com/github/benmanes/caffeine/cache/Caffeine.html

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

- メソッドの実行結果をキャッシュする

コードのメソッドに `@Cacheable` をつける
```java
@Component
class SampleCache {

     //このメソッドについてキャッシュを有効にする
    @Cacheable("getCache") // 引数は cache 名
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

- データをキャッシュする

CacheManager を使い、Key:Value の形式でオブジェクトを保存する  

事前に設定ファイル（ehcache.xml）を作成する  
格納場所は <project root>/main/resources/ehcache.xml とする
```xml
  <cache>
    <key-type>my.class.Key</key-type>
    <value-type>my.class.Value</value-type>
  </cache>
```
Key, Value はキャッシュの管理をおこなうのに使う Class を指定する。  
今回は my.class.* にある独自 Class で管理している場合を想定している。  

キャッシュの登録を管理する CacheManager の生成
```java
class CacheManager {
    public CacheManager() {init();}

    public void init() throws IOException {
        Configuration xmlConfiguration =
                new XmlConfiguration(new ClassPathResource("ehcache.xml").getURL());

        CacheManager cacheManager = CacheManagerBuilder.newCacheManager(xmlConfig);
        cacheManager.init();

        Cache<Key, Value> cache = cacheManager.getCache("alias", Key.class, Value);
    }
}
```

実際にデータの追加や取得する方法は以下
```java
// 停止
cacheManager.close()
// キャッシュ取得
Value get(Key key) {return cache.get(key);}
// キャッシュ追加・更新
void put(Key key, Value value) { cache.put(key, value);}
```
