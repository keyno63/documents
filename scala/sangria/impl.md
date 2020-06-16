# impl
sangria の実装方法をまとめる

play で動かす場合

## build.sbt
おなじみの設定から play の引き込み

project/plugins.sbt
```scala
resolvers += "Typesafe repository" at "https://repo.typesafe.com/typesafe/releases/"

addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.8.1")
```

必要なライブラリを build.sbt に追加します.  
必要、というのは、以下のライブラリ
- play関係 (play plugin, guice)  
- graphql 関係 (org.sangria-graphql~)
- json parser (circe を使用)

build.sbt
```scala
lazy val `scala_play` = (project in file(".")).enablePlugins(PlayScala)

libraryDependencies ++= Seq(specs2 % Test , guice ) ++
  Seq(
    // sangria
    "org.sangria-graphql" %% "sangria" % "2.0.0",
    "org.sangria-graphql" %% "sangria-slowlog" % "0.1.8",
    "org.sangria-graphql" %% "sangria-circe" % "1.2.1",
    "org.sangria-graphql" %% "sangria-spray-json" % "1.0.2",
    // circe(json libs)
    "io.circe" %% "circe-core" % "0.12.1",
    "io.circe" %% "circe-parser" % "0.12.1",
    "io.circe" %% "circe-generic" % "0.12.1",
    "io.circe" %% "circe-optics" % "0.9.3"
  ) :+ ("com.dripower" %% "play-circe" % "2812.0")
```

バージョンは 2020/06 のものを使用

## Controller 部分
雛形を作る  
```scala
@Singleton
class GraphQlController @Inject()(cc: ControllerComponents)(implicit ec: ExecutionContext)
  extends AbstractController(cc)
  with Circe {
  def graphql(): Action[Json] = Action(circe.json).async { request => ???
  }
}
```
- Controller は AbstractController を継承  
慣例
- Controller は Circe を継承  
リクエストボディの JSON parse のため
- Controller の第二引数に ExecutionContext の implicit パラメーター  
非同期処理（Future）の処理を行うために実装が必要
- エントリポイントになるメソッドの型は `Action[Json]`  
レスポンスは application/json を返すため
- メソッド右辺の最初は `Action(circe.json).async`  
リクエストは application/json で body の値を受けたいため、Action の引数は circe.json を渡す  
非同期処理を行いたいため、 async を使う  


もうちょっと書く.   
JSON の parse, GraphQL 用の処理のメソッドを呼び出す.
```scala
@Singleton
class GraphQlController @Inject()(cc: ControllerComponents)(implicit ec: ExecutionContext)
  extends AbstractController(cc)
  with Circe {
  def graphql(): Action[Json] = Action(circe.json).async { request =>
    parser.parse(request.body.toString) match {
      case Right(json) => execgraphQL(json)
      case _ => Future(BadRequest(s"Parse Error JSON. body=[${request.body.toString}]"))
    }
  }
}
```

- parser.parse(request.body.toString)  
body の parse  
circe の parser を使って、request body の json を解析  
- execgraphQL(json) は graphql に沿った処理. あとから実装するメソッド.  
- ワイルドカード部分は parse 失敗など. そういう時は BadRequest.  

