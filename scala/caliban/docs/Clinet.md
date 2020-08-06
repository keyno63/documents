# GraphQL Client
GraphQL クライアント

**Caliban-client** は Caliban から独立したモジュールで、型安全で関数的手法である Scala コードを使用して GraphQL クエリーを記述できるようにします。
これは [sttp](https://github.com/softwaremill/sttp) の上に構築されているため、選択したバックエンドを使用してリクエストを実行できます。

Caliban と同様、 `caliban-client` は純粋関数的なインターフェースを提供し、ボイラープレートを最小限に抑えます。  
これは次のよう効果があります。  
1. `caliban-codegen-sbt` ツールを使うことで、与えられた GraphQL スキーマからボイラープレートコードを生成します  
2. 生成されたコードからヘルパーを結合することで GraphQL クエリー/ミューテーション を記述します  
3. クエリー/ミューテーションを `sttp` リクエストに変換し、好きなバックエンドで実行します

## 依存

`caliban-client` を使うため、 `build.sbt` ファイルに以下を行を追記します。  

```
libraryDependencies += "com.github.ghostdogpr" %% "caliban-client" % "0.9.0"
```

Caliban-client は ScalaJS に対応しています。

## コード生成

`caliban-client` を用いて GraphQL クエリーを作成するための最初のステップは、GraphQL スキーマからボイラープレートコードを生成することです。  
そのためには、スキーマを含むファイルが必要です（バックエンドが `caliban` を使用している場合、API 上で`GraphQL＃render` を呼び出すことで取得できます）。

この機能を使用するには、 sbt プラグインである `caliban-codegen-sbt` をプロジェクトに追加して有効にします。  

```scala
addSbtPlugin("com.github.ghostdogpr" % "caliban-codegen-sbt" % "0.9.0")
enablePlugins(CodegenPlugin)
```
次に sbt コマンド `calibanGenClient` を呼び出します。  
```scala
calibanGenClient schemaPath outputPath [--scalafmtPath path] [--headers name:value,name2:value2]

calibanGenClient project/schema.graphql src/main/Client.scala
```

このコマンドは、`schemaPath` で定義された GraphQL スキーマで定義された全ての型に対して、ヘルパー関数を含む `outputPath` に Scala ファイルを生成します。  
ファイルの代わりに、イントロスペクションを使うことで得られる URL とスキーマ を作ることができます。  
生成されたコードは `--scalafmtPath` オプションで定義される設定を使って、 Scalafmt でフォーマットできます（デフォルトでは `.scalafmt.conf` を使用します）。  
`schemaPath` から URL を生成すると、`--headers` オプションでリクエストヘッダーを与えられます。  
生成コードのパッケージは `outputPath` のフォルダーから決定されます。  
このパッケージ名は `--packageName` オプションを使うことで上書きすることも可能です。

## クエリーの生成

ボイラープレートコードが生成されたら、クエリーの生成を始めることができます。  
スキーマの *型* ごとに、対応する Scala オブジェクトが作成されます。  
スキーマの *フィールド* ごとに、対応する Scala 関数が作成されます。  

例として、以下のようなスキーマを定義します。  
```graphql
type Character {
  name: String!
  nicknames: [String!]!
  origin: Origin!
}
```

以下のコードが生成されます。  
```scala
object Character {
  def name: SelectionBuilder[Character, String]            = ???
  def nicknames: SelectionBuilder[Character, List[String]] = ???
  def origin: SelectionBuilder[Character, Origin]          = ???
}
```

`SelectionBuilder[Origin, A]`  は `A` 型の結果を返す 親型 `Origin` からのセレクションです。 
この例では、 `name` は `String` を返す `Character` からのセレクションです。  

`~` セレクションを使うことで複数のセレクションを結合できます。  
新しい結果型は2つの結合された結果型からのタプルになります。  
結合できるのは同じオリジンをもつセレクションだけということに注意してください。  

```scala
val selection: SelectionBuilder[Character, (String, List[String])] =
  Character.name ~ Character.nicknames
```

複数のフィールドを組み合わせる場合は、 case class を使用してデータを表すと便利です。  
これはネストされたタプルが表示されないようにするためです。  
`mapN`を使用することで、ネストされたタプルを case class にマップできます。

```scala
case class CharacterView(name: String, nickname: List[String], origin: Origin)

val character: SelectionBuilder[Character, CharacterView] =
  (Character.name ~ Character.nicknames ~ Character.origin)
    .mapN(CharacterView)
```

オブジェクト型を返すフィールドは、別の `SelectionBuilder` である内部セレクションを要求します。  
次の `Query` 型と比べてみましょう。  

```graphql
type Query {
  characters: [Character!]!
}
```
`characters` を呼び出すときは、 返された `Character` 上で選択されたフィールドを示すために  
`SelectionBuilder[Character, ?]` を与える必要があります。  

```scala
val query: SelectionBuilder[RootQuery, List[CharacterView]] =
  Query.characters {
    (Character.name ~ Character.nicknames ~ Character.origin)
      .mapN(CharacterView)
  }
```

もしくは 作成した `character` セレクションを再利用するなら
```scala
val query: SelectionBuilder[RootQuery, List[CharacterView]] =
  Query.characters {
    character
  }
```

これは Scala コードなので、 GraphQL フラグメントを気にする必要なく、複数の場所でセレクションを簡単に再利用できます。  
Scala コンパイラーは、意味のあるフィールドのみを組み合わせることも保証します。

フィールドが引数が必要な場合、フィールドに対するヘルパーメソッドにも引数が必要です。  
クエリーをリッチ化すると次のようになります。  

```graphql
type Query {
  characters(origin: Origin!): [Character!]!
}
```

`characters` を呼んだ時に、`Origin` を与えることが必要になると、
```scala
val query: SelectionBuilder[RootQuery, List[CharacterView]] =
  Query.characters(Origin.MARS) {
    character
  }
```

## リクエストの実行

クエリ、およびミューテーションが作成されれば、それを実行する時がきました。  
これを行うために、`.toRequest` を呼び出して `SelectionBuilder` を `sttp` リクエストに変換できます。  
この関数は GraphQL サーバーのURLと オプションの boolean `useVariables` を引数に取ります。  
`useVariables` は GraphQL 引数が変数を使うかどうかで決定します。デフォルト値は false です。  

選んだバックエンドで `sttp` リクエストを簡単に実行できます。  
あまり詳しくないのであれば、  [sttp docs](https://sttp.readthedocs.io/en/latest/) を参照してください。  

`ZIO` に `AsyncHttpClient` バックエンドを使用する例を次に示します。
```scala
import sttp.client._
import sttp.client.asynchttpclient.zio.AsyncHttpClientZioBackend

AsyncHttpClientZioBackend().flatMap { implicit backend =>
  val serverUrl = uri"http://localhost:8088/api/graphql"
  val result: Task[List[CharacterView]] =
    query.toRequest(serverUrl).send().map(_.body).absolve
  ...
}
```

結果として、戻り値の型が `SelectionBuilder` と同じ ZIO `Task` を取得します。  
sttp リクエストには、送信するリクエストだけが含まれているわけではありません。  
期待されるタイプへの応答のパースも行います。

[examples](https://github.com/ghostdogpr/caliban/tree/master/examples/) プロジェクトには、  
例として作った GraphQL バックエンドをクエリーすることのできる、実行可能なサンプルコードがあります。

::: 注意すべき制限  
サポートされているのは、 Query と Mutation のみです。  
Subscription は将来的にサポートに追加される予定です。  
コード生成ツールによる型の拡張はサポートされていません。  
:::
