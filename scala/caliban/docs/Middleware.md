# Middleware

Caliban `Wrapper` の概念を使って、クエリ処理の様々なレベルでのアクションの追加を実現できます。
ラッパーを使用すると次のようなことができるようになります。
- クエリーが限界に達していないかを検証（深さ、複雑さ）
- 実行前のクエリー変更
- クエリーやフィールドにタイムアウトを追加
- 各フィールドの実行時間のログ
- [Apollo Tracing](https://github.com/apollographql/apollo-tracing)、 [Apollo Caching](https://github.com/apollographql/apollo-cache-control) 、および類似機能のサポート
- その他

## ラッパー型

ラッパーには5つの基本型があります。

- `OverallWrapper` はクエリー処理全体をラップします。
- `ParsingWrapper` はクエリーのパースのみをラップします。
- `ValidationWrapper` はクエリー検証のみをラップします。
- `ExecutionWrapper` はクエリー実行のみをラップします。
- `FieldWrapper` はそれぞれのフィールドの実行のみをラップします。

それぞれのラッパーは、いくつかの（クエリ文字列などの）コンテキスト情報と一緒に `ZIO`、および` ZQuery`評価を実行し、別の評価を返す関数が必要です。

処理に1分以上かかる場合にクエリ全体をタイムアウトするラッパーを実装する方法を見てみましょう。

```scala
val wrapper = OverallWrapper { process => (request: GraphQLRequest) =>
  process(request)
    .timeout(1 minute)
    .map(
      _.getOrElse(
        GraphQLResponse(
          NullValue,
          List(ExecutionError(s"Query was interrupted after timeout of ${duration.render}:\n$query"))
        )
      )
    )
  }
```

You can also combine wrappers using `|+|` and create a wrapper that requires an effect to be run at each query using `EffectfulWrapper`.

To use your wrapper, call `GraphQL#withWrapper` or its alias `@@`.

`| + |`を使用してラッパーを結合することができますし、  
`EffectfulWrapper`を使用して各クエリで作用を実行する必要があるラッパーを作成することもできます。

ラッパーを使用するには、 `GraphQL＃withWrapper`またはそのエイリアス` @@ `を呼び出します。

```scala
val api = graphQL(...).withWrapper(wrapper)
// or
val api = graphQL(...) @@ wrapper
```

## 事前定義されたラッパー

Caliban には `caliban.wrappers.Wrappers` 内に事前に作成されたラッパーを備えています。
-`maxDepth`は、指定された値よりも大きい深さのクエリーに失敗したラッパーを返します。
-`maxFields`は、指定された値よりも大きいフィールド数のクエリーに失敗したラッパーを返します。
-`timeout`は、指定された時間以上に実行時間がかかるクエリーに失敗したラッパーを返します。
-`printSlowQueries`は遅いクエリーを出力するラッパーを返します。
-`onSlowQueries`は、遅いクエリー上で特定の関数を実行できるラッパーを返します。

さらに、Calibanはいくつかの非仕様だが標準のラッパーも同梱しています。
-`caliban.wrappers.ApolloTracing.apolloTracing`は、[Apollo Tracing]（https://github.com/apollographql/apollo-tracing）形式に従って各レスポンスの` extensions`フィールドにトレースデータを追加するラッパーを返します。
-`caliban.wrappers.ApolloCaching.apolloCaching`は、[Apollo Caching]（https://github.com/apollographql/apollo-cache-control）形式を使用して、適切に注釈が付けられたフィールドにキャッシュヒントを追加するラッパーを返します。
-`caliban.wrappers.ApolloPersistedQueries.apolloPersistedQueries`は、[Apollo Persisted Queries]（https://github.com/apollographql/apollo-link-persisted-queries）形式のハッシュを使用してクエリーをキャッシュおよび取得するラッパーを返します。

これらは以下のように使うことができます。
```scala
val api =
  graphQL(...) @@
    maxDepth(50) @@
    timeout(3 seconds) @@
    printSlowQueries(500 millis) @@
    apolloTracing @@
    apolloCaching
```

## インタープレターのラップ

上記のすべてのラッパーでは、環境「R」と、常に「CalibanError」であるエラー型を変更しないようにする必要があります。  
`wrapExecutionWith`を呼び出すことで` GraphQLInterpreter`をラップすることも可能です。  
このメソッドは関数 `f`を受け取り、この関数` f`で `execute`メソッドをラップする新しい` GraphQLInterpreter`を返します。

`mapError`（エラーをカスタマイズ）と` provide`（環境の排除）を実装するために内部的に使用されますが、  
一般的なタイムアウトの追加、応答時間のロギングなど、他の用途にも使用できます。

```scala
val i: GraphQLInterpreter[MyEnv, CalibanError] = ???

// change error type to String
val i2: GraphQLInterpreter[MyEnv, String] = i.mapError(_.toString)

// provide the environment
val i3: GraphQLInterpreter[Any, CalibanError] = i.provide(myEnv)

// add a timeout on every query execution
val i4: GraphQLInterpreter[MyEnv with Clock, CalibanError] =
  i.wrapExecutionWith(
    _.timeout(30 seconds).map(
      _.getOrElse(GraphQLResponse(NullValue, List(ExecutionError("Timeout!"))))
    )
  )
```

## エラーレスポンスのカスタマイズ

クエリー実行時の様々な段階で、エラーが発生する可能性があります。  
Caliban は `CalibanError` の色々なインスタンスを GraphQL 仕様に準拠したレスポンスに表示します。  
これはエラーを作用のエラーチャンネルに入れてしまうため、ある時点で `ExecutionError` に遭遇する可能性が高くなります。  
Caliban がクエリー実行中に発生したエラーに関する基本的なメッセージを表示できるようにするため、エラーに `Throwable` を継承することは重要です。  

より効果的なエラーハンドリングをするため、GraphQL の仕様上ではエラーレスポンスに [`extension`](http://spec.graphql.org/June2018/#example-fce18) オブジェクトを使うことができます。  
たとえば、このオブジェクトにはフロントエンドで処理できる列挙型のエラーコードをモデル化するための `code` 情報を含めることができます。  
この情報を生成するため、`GraphQLInterpreter` 上の `mapError` 関数を使うことができます。
`ExecutionError` 内の独自ドメインエラーを意味のあるエラーコードに変換する例を以下になります。  

```scala
sealed trait ExampleAppEncodableError extends Throwable {
    def errorCode: String
}
case object UnauthorizedError extends ExampleAppEncodableError {
    override def errorCode: String = "UNAUTHORIZED"
}

def withErrorCodeExtensions[R](
  interpreter: GraphQLInterpreter[R, CalibanError]
): GraphQLInterpreter[R, CalibanError] = interpreter.mapError {
  case err @ ExecutionError(_, _, _, Some(exampleError: ExampleAppEncodableError), _) =>
    err.copy(extensions = Some(ObjectValue(List(("errorCode", StringValue(exampleError.errorCode))))))
  case err: ExecutionError =>
    err.copy(extensions = Some(ObjectValue(List(("errorCode", StringValue("EXECUTION_ERROR"))))))
  case err: ValidationError =>
    err.copy(extensions = Some(ObjectValue(List(("errorCode", StringValue("VALIDATION_ERROR"))))))
  case err: ParsingError =>
    err.copy(extensions = Some(ObjectValue(List(("errorCode", StringValue("PARSING_ERROR"))))))
}
```
