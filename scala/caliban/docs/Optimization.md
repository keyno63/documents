# Query optimization
クエリーの適正化  

GraphQL クエリーは同じりゾルバーが使っている複数のフィールドを要求することがあります。  
リゾルバーが単なる値の場合は問題ないのですが、  
リゾルバーが（データベース読み込みなどの）作用を伴いながら実行される場合、  これは最適ではありません。  

次のことを行います:
- 同じクエリーをキャッシュする
- 同じ資源へのクエリーをバッチ処理する

Caliban では [`ZQuery`](https://github.com/zio/zquery) データ型を用いて、これを実現しています。  

## ZQuery の導入

`ZQuery[R, E, A]` は1つ以上のリソースへのリクエストを含む可能性のある作用的なクエリーの純粋関数表現です。  
`ZIO[R, E, A]` と同じように、環境 `R`、失敗に`E`、成功時に`A` が必要です。  
シーケンシャルに実行する必要がない全てのリクエストは自動的にバッチ処理され、積極的なデータソース固有の最適化が可能になります。  
リクエストも自動的に重複排除されてキャッシュされます。

これにより、クエリーが自動的に最適化されることを確信しながら、  
高水準の構成形式のクエリーを書くことができます。  
たとえば、ユーザーサービスからきた次のクエリーについて考えてみます。

```scala
val getAllUserIds: ZQuery[Any, Nothing, List[Int]] = ???
def getUserNameById(id: Int): ZQuery[Any, Nothing, String] = ???

for {
  userIds   <- getAllUserIds
  userNames <- ZQuery.foreachPar(userIds)(getUserNameById)
} yield userNames
```

この処理には通常、N + 1個のクエリーが必要です。  
1つは`getAllUserIds`、もう1つは`getUserNameById`の呼び出しです。  
対照的に、バッチ処理をサポートするユーザーサービスの実装を前提として、 `ZQuery`はこれを自動的に2つのクエリに最適化します。  
1つは` userIds`用、もう1つは `userNames`用です。

## データソースの構築

リクエストを実行する `ZQuery` を構築するために、最初に `DataSource` を構築する必要があります。  
`DataSource[R, E, A]`  は型 `A` のリクエストを実行する方法を定義し、それには2つのものを要求します。

- データソースを一意に区別する`identifier`（異なるデータソースからのリクエストは一緒にバッチ処理されません）  
- リクエストの `Iterable` から、リクエストとレスポンスの `Map` への作用的な関数 `run`  

前の例から `getUserNameById` について考えてみましょう。  
与えられたレスポンス型から `zquery.Request` を拡張し、  
対応したリクエスト型を定義する必要があります。  

```scala
case class GetUserName(id: Int) extends Request[Throwable, String]
```

次に、対応した`DataSource` を構築します。  
次の関数を実装する必要があります。  

```scala
val UserDataSource = new DataSource[Any, GetUserName] {
  override val identifier: String = ???
  override def run(requests: Iterable[GetUserName]): ZIO[Any, Throwable, CompletedRequestMap] = ???
}
```

識別子として `UserDataSource` を使います。  
この名称は他のデータソースでの再利用はすべきではありません。  

```scala
override val identifier: String = "UserDataSource"
```

単リクエストを受け取るか、複数のリクエストを同時に受け取るかに応じて、  2つの異なる動作を定義します。  
それぞれのリクエストに対して、 `Either`型の値をリザルトマップに追加する必要があります。  
(`Either`: 結果がエラーの場合は`Left`、成功した場合は `Right` になる型)

```scala
override def run(requests: Iterable[GetUserName]): ZIO[Any, Nothing, CompletedRequestMap] = {
  val resultMap = CompletedRequestMap.empty
  requests.toList match {
    case request :: Nil =>
      // get user by ID e.g. SELECT name FROM users WHERE id = $id
      val result: Task[String] = ???
      result.either.map(resultMap.insert(request))
    case batch =>
      // get multiple users at once e.g. SELECT id, name FROM users WHERE id IN ($ids)
      val result: Task[List[(Int, String)]] = ???
      result.fold(
        err => requests.foldLeft(resultMap) { case (map, req) => map.insert(req)(Left(err)) },
        _.foldLeft(resultMap) { case (map, (id, name)) => map.insert(GetUserName(id))(Right(name)) }
      )
  }
}
```

ここから `ZQuery` を構築するには、`ZQuery.fromRequest` を使い、リクエストとデータソースを渡すだけです。  

```scala
def getUserNameById(id: Int): ZQuery[Any, Throwable, String] =
  ZQuery.fromRequest(GetUserName(id))(UserDataSource)
```

`ZQuery` を実行するために、`ZIO[R, E, A]` を返す `ZQuery#run` を単純に使いました。  

## ZQuery のコンストラクターとオペレーター

`ZQuery` の作り方にはいくつか方法があります。  
`ZQuery.fromRequest` について触れましたが、他にも次の方法があります。  
- `ZQuery.succeed` で純粋な値から作成します  
- `ZQuery.fromEffect` で作用値から作成  
- `ZQuery.collectAllPar`と` ZQuery.foreachPar`、  
およびそれらのシーケンシャルに相当する `ZQuery.collectAll`と` ZQuery.foreach`を使用して、  
複数のクエリーから作成します


`ZQuery` オブジェクトがあれば、以下を使うこともできます。
- `map`、および `mapError` を使って、返された結果やエラーを変更
- `flatMap`、または `zip` を使って、他の `ZQuery` オブジェクトと結合
- `provide`、および `provideSome` を使って、要求される`R`のいくつかを削除

`ZQuery` を実行するにはいくつかの方法があります。  
- `runCache`は、事前に取得した特定のキャッシュを使用してクエリーを実行します。  
これは、新規リクエストを実行せずにクエリーを確実に「再現」するのに便利です。  
- `runLog`はクエリーを実行し、実行されたすべてのリクエストとその結果の完全なログを含むキャッシュとともに結果を返却します。  
これは、クエリー実行のロギング、および分析するのに便利です。  
- `run`はクエリーを実行し、その結果を返します。

## Caliban での ZQuery の使用

Caliban で `ZQuery` を使うには、 API 定義に `ZQuery` 型のフィールドを単純に追加するだけです。  

```scala
case class Queries(
  users: ZQuery[Any, Nothing, List[User]],
  user: UserArgs => ZQuery[Any, Nothing, User])
```

クエリーの実行中、Caliban は `ZQuery` を返すリクエストされた全てのフィールドを1つの `ZQuery` にマージします。  
その際に可能な限り全ての最適化が適応される様にします。  

[examples](https://github.com/ghostdogpr/caliban/tree/master/examples) プロジェクトは [GraphQL の最適化について](https://blog.apollographql.com/optimizing-your-graphql-request-waterfalls-7c3f3360b051)で説明されている問題の2つのバージョンを提供します。  

- [naive](https://github.com/ghostdogpr/caliban/tree/master/examples/src/main/scala/caliban/optimizations/NaiveTest.scala) バージョンでは、47 リクエストの結果としてフィールドは単に `IO` を返す
- [optimized](https://github.com/ghostdogpr/caliban/tree/master/examples/src/main/scala/caliban/optimizations/OptimizedTest.scala) バージョンでは、8リクエストのみの結果として、フィールドは`ZQuery`を返す 
