# Getting Started

[原文](https://ghostdogpr.github.io/caliban/docs/)  

Caliban は Scala で GraphQl バックエンドを作成するための純粋な関数型ライブラリーです。  
[Magnolia](https://github.com/propensive/magnolia) を用いてデータ型から GraphQL スキーマを自動的に作成し、  
[Fastparse](https://github.com/lihaoyi/fastparse) を用いてクエリーを解析し、  
[ZIO](https://github.com/zio/zio) を用いて様々な作用を扱います。 


このライブラリの設計原理は次のとおりです。
* 純粋なインターフェイス: エラーや作用は明示的に返却され(例外は投げられることなく)、  
全ての返される型は参照透過です（`Future` でくるまれていません）。
* 「スキーマ定義」と「実装」の明確な分離： スキーマの定義と検証は、  
（リフレクションは使用せず）コンパイル時に Scala 標準の型を用いて行われ、  
リゾルバーは実行時に用いられる単純な値です。
* 最小のボイラープレート：APIで使う全ての型に対して、手動での定義は必要ありません。

Calibanは、フロントエンドのGraphQL構築にも使用できます。詳細については、専用セクションを参照してください。  

## 依存
`caliban` を使うには、`build.sbt` に以下の行を追加してください。  
```sbtshell
libraryDependencies += "com.github.ghostdogpr" %% "caliban" % "0.8.3"
```

以下のモジュールはオプションです。
```sbtshell
libraryDependencies += "com.github.ghostdogpr" %% "caliban-http4s"     % "0.8.3" // routes for http4s
libraryDependencies += "com.github.ghostdogpr" %% "caliban-akka-http"  % "0.8.3" // routes for akka-http
libraryDependencies += "com.github.ghostdogpr" %% "caliban-play"       % "0.8.3" // routes for play
libraryDependencies += "com.github.ghostdogpr" %% "caliban-finch"      % "0.8.3" // routes for finch
libraryDependencies += "com.github.ghostdogpr" %% "caliban-uzhttp"     % "0.8.3" // routes for uzhttp
libraryDependencies += "com.github.ghostdogpr" %% "caliban-cats"       % "0.8.3" // interop with cats effect
libraryDependencies += "com.github.ghostdogpr" %% "caliban-monix"      % "0.8.3" // interop with monix
libraryDependencies += "com.github.ghostdogpr" %% "caliban-federation" % "0.8.3" // interop with apollo federation
```

## 簡単な例

Caliban を使って GraphQL API を作成することは、 case class の作成と同じくらい簡単です。  
実際、 GraphQL スキーマ全体は case class 構造（フィールドや参照する他の型）から得られ、  
Resolver はその case class の単なるインスタンスです。  

例として、`Character` クラスと2つの関数`getCharacters`と`getCharacter`があるとします。  
```scala
case class Character(name: String, age: Int)

def getCharacters: List[Character] = ???
def getCharacter(name: String): Option[Character] = ???
```

API を表す Queries という名前の case class を作ってみましょう。  
2つのフィールドは公開する関数（関数のレコード）に基づいて命名とモデル化がされています。  
実際の関数を呼び出す値を class に作成します。これが Resolver です。  
```scala
// schema
case class CharacterName(name: String)
case class Queries(characters: List[Character],
                   character: CharacterName => Option[Character])
// resolver
val queries = Queries(getCharacters, args => getCharacter(args.name))
```

次のステップは GraphQL API 定義をつくることです。  
最初に、クエリーリゾルバーを `RootResolver` でラップします。  
`RootResolver` は Query、 Mutation、 Subscription を含むルートオブジェクトです。必須なのは Query のみです。  
次に、簡単なリゾルバー値を GraphQL API 定義に変換する `graphQL` 関数を呼び出します。  
スキーマ全体はコンパイル時に生成されます。つまり、コンパイルをすることでスキーマを提供することができます。  

```scala
import caliban.GraphQL.graphQL
import caliban.RootResolver

val api = graphQL(RootResolver(queries))
```

与えられたスキーマを可視化するために `api.render` を使えます。  
今回の場合では：
```scala
type Character {
  name: String!
  age: Int!
}

type Queries {
  characters: [Character!]!
  character(name: String!): Character
}
```
