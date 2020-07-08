# Interop
相互運用

ZIO の代わりに [Cats Effect](https://github.com/typelevel/cats-effect)、または [Monix](https://github.com/monix/monix) を使いたくなったとき、  
それぞれ `caliban-cats`、および `caliban-monix` モジュールを使うことができます。

`caliban-tapir` モジュールは [Tapir](https://github.com/softwaremill/tapir) エンドポイントを GraphQL API に変換することができます。  

## Cats Effect

`caliban.interop.cats.implicits._` を import し、implicit パラメーターの `zio.Runtime` をスコープ中に持つようにします。  
その際にいくつかのヘルパーが存在します。

- `GraphQL` オブジェクトは `interpreterAsync` によりリッチ化されています。  
`interpreterAsync` は `ZIO` の代わりに `F[_]: Async` を返す `interpreter` の亜種です。  
- `GraphQLInterpreter` オブジェクトは `executeAsync`、および `checkAsync` によりリッチ化されています。  
`ZIO` の代わりに `F[_]: Async` を返す、`execute`、および `check` の亜種です。  
- `Http4sAdapter` には cats-effect の亜種である `makeRestServiceF`、および`makeWebSocketServiceF` が存在します。

それに加えて、任意の `F [_]：Effect` に対する `Schema`が与えられます。  
つまり、クエリー、ミューテーション、 およびサブスクリプションに Cats IO に対する Monix タスクを返すフィールドを含めることができます。  

次の例は、Cats IOのみを使用してインタープリターを作成し、クエリを実行する方法を表しています。

```scala
import caliban.GraphQL.graphQL
import caliban.RootResolver
import caliban.interop.cats.implicits._
import cats.effect.{ ExitCode, IO, IOApp }
import zio.Runtime

object ExampleCatsInterop extends IOApp {

  implicit val runtime = Runtime.default

  case class Queries(numbers: List[Int], randomNumber: IO[Int])

  val queries     = Queries(List(1, 2, 3, 4), IO(scala.util.Random.nextInt()))
  val api = graphQL(RootResolver(queries))

  val query = """
  {
    numbers
    randomNumber
  }"""

  override def run(args: List[String]): IO[ExitCode] =
    for {
      interpreter <- api.interpreterAsync[IO]
      result      <- interpreter.executeAsync[IO](query)
      _           <- IO(println(result.data))
    } yield ExitCode.Success
}
```

この例は [examples](https://github.com/ghostdogpr/caliban/blob/master/examples/src/main/scala/caliban/interop/cats/ExampleCatsInterop.scala) プロジェクト内にあります。

## Monix

`caliban.interop.monix.implicits._` を import し、implicit パラメーターの `zio.Runtime` をスコープ中に持つようにします。  
その際にいくつかのヘルパーが存在します。

- `GraphQL` オブジェクトは `interpreterAsync` によってリッチ化されてます。  
`interpreterAsync` は `ZIO` の代わりに Monix `Task` を返す `interpreter` の亜種です。  
- `GraphQLInterpreter` オブジェクトは `executeAsync`、および `checkAsync` によりリッチ化されています。 
`ZIO` の代わりに Monix `Task` を返す、`execute`、および `check` の亜種です。  

それに加えて、 `Observable` と同様に任意の Monix `Task` に対する `Schema` が与えられます。  

次の例は、 Monix `Task` のみを使用してインタープリターを作成し、クエリーを実行する方法を表しています。

```scala
import caliban.GraphQL.graphQL
import caliban.RootResolver
import caliban.interop.monix.implicits._
import cats.effect.ExitCode
import monix.eval.{ Task, TaskApp }
import monix.execution.Scheduler
import zio.DefaultRuntime

object ExampleMonixInterop extends TaskApp {

  implicit val runtime = new DefaultRuntime {}
  implicit val monixScheduler: Scheduler = scheduler

  case class Queries(numbers: List[Int], randomNumber: Task[Int])

  val queries     = Queries(List(1, 2, 3, 4), Task.eval(scala.util.Random.nextInt()))
  val api = graphQL(RootResolver(queries))

  val query = """
  {
    numbers
    randomNumber
  }"""

  override def run(args: List[String]): Task[ExitCode] =
    for {
      interpreter <- api.interpreterAsync
      result      <- interpreter.executeAsync(query)
      _           <- Task.eval(println(result.data))
    } yield ExitCode.Success
}
```

この例は [examples](https://github.com/ghostdogpr/caliban/blob/master/examples/src/main/scala/caliban/interop/monix/ExampleMonixInterop.scala) プロジェクト内にあります。  

## Tapir

Tapir の `Endpoint`、 `ServerEndpoint` 上で `toGraphQL`  を呼び出すために、  
ビルド設定に `caliban-tapir` を追加し、コード上で `import caliban.interop.tapir._` を追加してください。  

変換規則は次のとおりです。
- `GET` エンドポイントは Query に変換されます
- `PUT`、`POST`、および `DELETE` エンドポイントは `Mutation` に変換されます 
- クエリーパスは QraphQL フィールドに命名されます  
(例えば、 `/book/add` エンドポイントは GraphQL フィールド上で `bookAdd` に変更されます)
- クエリーパラメーター、ヘッダー、Cookie、および リクエストボディーは GraphQL 引数として扱われます。  
- implicit パラメーターとして、`Schema`、および`ArgBuilder`が必要です  
`Schema` はインプット型、およびアウトプット型用で、 `ArgBuilder` はインプット型用です  
（詳細は [dedicated docs](schema.md) を参照してください）  

例をみて、 Tapir エンドポイントについて想像してください。
```scala
val addBook: Endpoint[(Book, String), Nothing, Unit, Nothing] =
  infallibleEndpoint
    .post
    .in("books")
    .in("add")
    .in(
      jsonBody[Book]
        .description("The book to add")
        .example(Book("Pride and Prejudice", 1813))
    )
    .in(header[String]("X-Auth-Token").description("The token is 'secret'"))
```

そして可能な実装としては以下
```scala
def bookAddLogic(book: Book, token: String): UIO[Unit] = ???
```

`toRoute` を呼び出し、実装を渡して http4s route を作るのと同じように、  
`GraphQL API` をつくるために `toGraphQL` を呼び出します。  

```scala
val api: GraphQL[Any] = addBook.toGraphQL((bookAddLogic _).tupled)
```
That's it! You can combine multiple `GraphQL` objects using `|+|` and expose the result using one of Caliban's adapters.
以上です。 `|+|`を使って複数の `GraphQL` オブジェクトの結合と、 Caliban アダプターの1つを使って結果を公開できます。  

GraphQL と通常の HTTP の両方で `bookAddLogic` を再利用する場合は、  
`.serverLogic` を呼び出し、 `Endpoint` を `ServerEndpoint` に変換できます。  
```scala
val addBookEndpoint: ServerEndpoint[(Book, String), Nothing, Unit, Nothing, UIO] =
  addBook.serverLogic[UIO] { case (book, token) => bookAddLogic(book, token).either }
```
これを使用し、HTTP route（http4s の `toRoutes` など）とGraphQL API（`.toGraphQL`）の両方を生成できます。  
```scala
val api: GraphQL[Any] = addBookEndpoint.toGraphQL
```

github 上に [全ての例](https://github.com/ghostdogpr/caliban/tree/master/examples/src/main/scala/caliban/tapir) があります。  

### GraphQL の制限

[GraphQL 仕様](https://spec.graphql.org/June2018/#sec-Operation-Name-Uniqueness) では全てのオペレーションに固有の名称が要求されます。

オペレーションの [名称](https://github.com/softwaremill/tapir/blob/master/core/src/main/scala/sttp/tapir/Endpoint.scala#L287) をカスタマイズするには `EndpointInfo.name` を使用します。

```scala
endpoint
  .name("overrideName")
``` 
