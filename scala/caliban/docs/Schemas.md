# Schemas

GraphQL スキーマは、リゾルバーで与えられた型から（リフレクションなしで）コンパイル時に自動的に生成されます。  
次の表は、一般的な Scala 型から GraphQL 型への変換を表しています。  

| Scala 上の型                                         | GraphQL 上の型                                                    |
| --------------------------------------------------- | ---------------------------------------------------------------- |
| Boolean                                             | Boolean                                                          |
| Int                                                 | Int                                                              |
| Float                                               | Float                                                            |
| Double                                              | Float                                                            |
| String                                              | String                                                           |
| java.util.UUID                                      | ID                                                               |
| Unit                                                | Unit (custom scalar)                                             |
| Long                                                | Long (custom scalar)                                             |
| BigInt                                              | BigInt (custom scalar)                                           |
| BigDecimal                                          | BigDecimal (custom scalar)                                       |
| Case Class                                          | Object                                                           |
| Sealed Trait                                        | Enum or Union                                                    |
| Option[A]                                           | Nullable A                                                       |
| List[A]                                             | List of A                                                        |
| Set[A]                                              | List of A                                                        |
| Seq[A]                                              | List of A                                                        |
| Vector[A]                                           | List of A                                                        |
| A => B                                              | A and B                                                          |
| (A, B)                                              | Object with 2 fields `_1` and `_2`                               |
| Either[A, B]                                        | Object with 2 nullable fields `left` and `right`                 |
| Map[A, B]                                           | List of Object with 2 fields `key` and `value`                   |
| ZIO[R, Nothing, A]                                  | A                                                                |
| ZIO[R, Throwable, A]                                | Nullable A                                                       |
| Future[A]                                           | Nullable A                                                       |
| ZStream[R, Throwable, A]                            | A (subscription) or List of A (query, mutation)                  |
| Json (from [Circe](https://github.com/circe/circe)) | Json (custom scalar, need `import caliban.interop.circe.json._`) |
| Json (from [play-json](https://github.com/playframework/play-json)) | Json (custom scalar, need `import caliban.interop.play.json._`) |

独自型のサポート方法は [Custom Types](#custom-types) の章を参照してください。  

もし Caliban にサポートして欲しい他の標準の型があれば、遠慮なく PR や [issue の作成](https://github.com/ghostdogpr/caliban/issues)  をしてください。

::: スキーマ生成の問題に関する注意  

Magnolia(※) は多くのネストされた型や、複数の箇所で再利用される型を含むスキーマを生成する際に問題が発生する場合があります。  
対処方法として、case class と seald trait を明示的に宣言します。  
※コンパイル時にスキーマ生成するために使用しているライブラリ

```scala
implicit val roleSchema      = Schema.gen[Role]
implicit val characterSchema = Schema.gen[Character]
```

`graphQL(...)` を呼び出す時にこれらの implicit がスコープ中で有効であることを確認してください。  
これにより、クラスに対するスキーマの事前生成や、必要に場合に再利用する場合に、 Magnolia のジョブは簡単になります。  
そうすれば、コンパイル時間を短縮し、生成されるバイトコードも圧縮されます。

:::

## Enums, unions, interfaces

sealed trait は、その内容に応じて別の GraphQL 型に変換されます。

- case object だけに継承される sealed trait は `ENUM` に変換されます
- case class だけに継承される sealed trait は`UNION` に変換されます

そのため、 sealed trait の継承先として case class と case object が混在する場合、  
複合型が生成され、 case object はクエリできない”偽”のフィールド名 `_` を持つことになります。 

```scala
sealed trait ORIGIN
object ORIGIN {
  case object EARTH extends ORIGIN
  case object MARS  extends ORIGIN
  case object BELT  extends ORIGIN
}
```

以上のスニペットは、次の GraphQL 型を生成します。

```graphql
enum Origin {
  BELT
  EARTH
  MARS
}
```

以下は複合の例です。

```scala
sealed trait Role
object Role {
  case class Captain(shipName: String) extends Role
  case class Engineer(specialty: String) extends Role
  case object Mechanic extends Role
}
```

以上のスニペットは次のような GraphQL 型を与えます。

```graphql
union Role = Captain | Engineer | Mechanic

type Captain {
  shipName: String!
}

type Engineer {
  specialty: String!
}

type Mechanic {
  _: Boolean!
}
```

`Union` 型の代わりに `Interface` を使用する場合、`@GQLInterface` アノテーションを sealed trait に追加します。  
同じ型を返し続ける限り、sealed class を拡張した case class に共通する全てのフィールドを使用したインターフェースが生成されます。

