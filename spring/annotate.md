# Spring Boot のアノテーションについて

#  はじめに
私の所属しているチームでは Spring Boot を使って Java で開発しているものもあります。  
この春、異動でチームに参加されることになった同僚が  
「Javaの開発経験なし」「（もちろん）Spring Bootも未経験」という方で、  
色々と Spring Boot に関して質問をして頂きました。  

（私はチームに参加してから Java の開発はせずにPHPの保守ばかりしているのですが）  
質問されてみて「意外と理解してなかったな」と思うところがあったので書いてみます。  

今回はアノテーション（の一部）について内容を整理します。    
個人的によく使われているものとおもったもので、「これなに？」とか「どうなってるの？」と聞かれて答えた内容です。

## 前提条件
開発環境／使用するバージョンなど

* IntelliJ idea
* Spring Boot 2.2.5 くらい

#  記載するアノテーション
 コンポーネント要素になるアノテーション群

 * Controller
 * Service
 * Repository
 * Component
 * Bean

コントローラーまわりアノテーション

* RestController
* RequestMapping
* PostMapping
* GetMapping

DIで使用するアノテーション

* Autowired

# 本題
## コンポーネント要素になる群
### Controller
MVCのコントローラー層に付与します。  
このコンポーネントのメソッド戻り値でViewやレスポンスボディとなります。

```java
@Controller
public class SampleController {
  
  SampleService sampleService;
  
  public SampleController(SampleService sampleService) {
    this.sampleService = sampleService;
  }

  @GetMapping("/get")
  public String get() {
      return "Hello World";
  }

}
```


### Service
MVCのサービス層に付与します。

```java
@Service
public class SampleService {
  
  private SampleRepository sampleRepository;

  public SampleService(SampleRepository sampleRepository) {
    this.sampleRepository = sampleRepository;
  }

  public String get() {
      return sampleRepository.get();
  }

}
```

### Repository
MVCのコントローラー層に付与します。  
このアノテーションを付与すると依存するドライバの例外をキャッチしてまとめてくれます

```java
@Repository
public class SampleRepository {
  
  public SampleRepository() {
  }

  public String get() {
      return "Hello World";
  }

}
```

### Component
MVC以外のコンポーネントに付与します。  
コンポーネントスキャンの対象になります。  
このアノテーションを付与したクラスは DI 対象になります。

### Bean
Bean 定義のコンポーネントに付与します。  
設定の Bean に付与します。

Configration アノテーションで関連した Bean が起動します。

## コントローラーまわりアノテーション
### RestController
Controller の拡張です。  
このアノテーションを付与したメソッドの戻り値はレスポンスボディとしてリクエスト元に返されます。

### RequestMapping
リクエストパスを設定します。

### GetMapping
GET リクエストのリクエストマップとして使われます。  
`RequestMapping(method=RequestMethod.GET)`のエイリアスです。

### PostMapping
POST リクエストのリクエストマップとして使われます。  
`RequestMapping(method=RequestMethod.POST)`のエイリアスです。

## DIで使用するアノテーション
浮いてしまった感がいなめませんが、質問があったし気になるよなと思ったのでくっつけました。
### Autowired
DI対象のコンポーネントを自動的に代入してくれます。  
いまはロジックコードにつけるのはおススメではなく、  
コンストラクタインジェクションを使ったほうが良いようです。
