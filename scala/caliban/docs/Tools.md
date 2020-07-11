# Tools
ツール

Calibanには、いくつかの便利な機能を公開する `caliban-tools` というモジュールが付属されています。  
- `caliban-codegen-sbt` による全てのコードの生成機能  
これは sbt を使わずに実行できます。  
詳しくは `caliban.tools.Codegen` を参照してください。
- GraphQL 内部クライアント  
詳しくは `caliban.tools.IntrospectionClient` を参照してください。
- GraphQL スキーマを比較する方法  
スキーマが Caliban なのか、リモートサーバーなのかは、以下を参照してください。

## 依存

```
libraryDependencies += "com.github.ghostdogpr" %% "caliban-tools" % "0.9.0"
```

## スキーマの比較

オブジェクト `caliban.tools.SchemaComparison` は、異なる起源の2つのスキーマを比較する `compare` 関数を公開しています。  
2つの `SchemaLoader` を引数として受け取り、次のコンストラクターのいずれかを使用して作成できます。
- `fromCaliban`：Caliban から` GraphQL`オブジェクトを渡します
- `fromFile`：GraphQL IDL でスキーマを含むファイルへのパスを渡します
- `fromString`：スキーマを含む文字列を GraphQL IDL に渡します
- `fromIntrospection`：内部処理をサポートする GraphQL サーバーの URL を渡します

`compare`の出力は `Task [List [SchemaComparisonChange]]`であり、  
`SchemaComparisonChange`はさまざまな種類の変更を表す sealed trait です。  
`SchemaComparisonChange＃breaking` は、フィールド、および型の削除など、破壊的な変更かを表します。  
`SchemaComparisonChange＃toString` は変更のわかりやすい説明を返します。  

次の例では、Calibanによって取得されたスキーマと文字列で定義されたスキーマを比較し、その違いを出力します。

```scala
import caliban.GraphQL.graphQL
import caliban.RootResolver
import caliban.tools._
import zio.UIO

// schema from String
val schema: String =
  """
  type Hero {
    name(pad: Int!): String!
    nick: String!
    bday: Int
  }
  
  type Query {
    hero: Hero!
  }"""

// schema from Caliban
case class NameArgs(pad: Int)
case class Hero(name: NameArgs => String, nick: String, bday: Option[Int])
case class Query(hero: Hero)

val api = graphQL(RootResolver(Query(Hero(_ => "name", "nick", None))))

for {
  diff <- SchemaComparison.compare(SchemaLoader.fromString(schema), SchemaLoader.fromCaliban(api))
  _    <- UIO(println(diff.mkString("\n")))
} yield ()
```