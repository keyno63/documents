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

独自型のサポート方法は [独自型](#独自型) の章を参照してください。  

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

## Enum, 複合, インターフェース

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

## 引数

引数をとるフィールドを宣言するには、引数を表す専用の case class を作成し、フィールドをクラスから結果の型への _関数_ にします。  

```scala
case class FilterArgs(origin: Option[Origin])
case class Queries(characters: FilterArgs => List[Character])
```

以上のスニペットは、以下の GraphQL 型を生成します。

```graphql
type Queries {
  characters(origin: Origin): [Character!]!
}
```

Caliban は `Int`、 `String`、 `List`、 `Option`、などのような一般的な型への自動生成を提供します。  
`caliban.schema.ArgBuilder` の implicit インスタンスを与えることにより、独自型をサポートすることも可能です。

::: ヒント  
タプルへの `ArgBuilder` はありません。 複数の引数を持つ場合、タプルの代わりに全ての引数を含んだ case class を使ってください。
:::

## 作用

フィールドは ZIO の作用を返すことができます。  
これにより、ZIO が提供する全ての機能（タイムアウト、リトライ、ZIO環境へのアクセス、メモ化など）を活用することができます。  
対応するフィールドを必要とするクエリが実行されるごとに作用が発生します。

```scala
case class Queries(characters: Task[List[Character]],
                   character: CharacterName => RIO[Console, Character])
```

ZIO 環境（`R`=`Any`） を使用しない場合、特別にすることはありません。

ZIO 環境が必要な場合、`graphQL(...)`  を呼び出す際にスコープ内に独自の `R` の`caliban.schema.GenericSchema[R]` の content が必要です。

```scala
object schema extends GenericSchema[MyEnv]
import schema._
```

## アノテーション

Caliban はリッチ化されたデータ型へのアノテーションをいくつかサポートしています。 

- `@GQLName("name")` を使用すると、データ型、およびフィールドに別の名称を指定できます。
- `@GQLInputName("name")` を使用すると、入力として使用されるデータ型に別の名称を指定できます（デフォルトでは、プレフィックスに `Input` が型名に追加されます）。
- `@GQLDescription("description")` を使用すると、データ型、およびフィールドの説明を追加できます。この説明はスキーマが内部参照された場合に表示されます。
- `@GQLDeprecated("reason")` を使用すると、フィールドや enum 値を deprecateting(廃止) にすることができます。
- `@GQLInterface` ユニオンの代わりにインターフェイスを生成する sealed trait を強制します。
- `@GQLDirective(directive: Directive)` を使用すると、フィールドや型を明示することができます。

## 独自型

Caliban は `Int`、 `String`、 `List`、 `Option`、などのような一般的な型への自動生成を提供します。  
`caliban.schema.Schema` の implicit インスタンスを与えることにより、独自型をサポートすることも可能です。  

これを行うための簡単な方法は、既存のインスタンスを再利用し、`contramap`を使用して定義した型から元の型にマッピングすることです。  
以下は [refined](https://github.com/fthomas/refined) の `NonEmptyString` のインスタンスを作成し、`String`型の既存インスタンスを再利用する例です。

```scala
import caliban.schema._
implicit val nonEmptyStringSchema: Schema[Any, NonEmptyString] = Schema.stringSchema.contramap(_.value)
```

`scalarSchema`ヘルパーを使用して独自のスカラー型を作成し、型から名前、オプションの説明、および関数を` ResponseValue`に指定することもできます。

```scala
import caliban.schema._
implicit val unitSchema: Schema[Any, Unit] = scalarSchema("Unit", None, _ => ObjectValue(Nil))
```

入力の一部として独自型を使用している場合は、 `caliban.schema.ArgBuilder`の implicit instance も生成する必要があります。  
たとえば、 `java.time.LocalDate`を扱う方法は次のとおりです。

```scala
implicit val localDateArgBuilder: ArgBuilder[LocalDate] = {
  case StringValue(value) =>
    Try(LocalDate.parse(value))
      .fold(ex => Left(ExecutionError(s"Can't parse $value into a LocalDate", innerThrowable = Some(ex))), Right(_))
  case other => Left(ExecutionError(s"Can't build a LocalDate from input $other"))
}
```

デフォルトでは値型はラップされていません。

## コードの生成

Caliban は GraphQL スキーマから Scala コードを自動生成できます。

この機能を使うためには、`caliban-codegen-sbt` をプロジェクトに追加し、有効にしてください。

```scala
addSbtPlugin("com.github.ghostdogpr" % "caliban-codegen-sbt" % "0.9.0")
enablePlugins(CodegenPlugin)
```

次に sbt コマンドの `calibanGenSchema` を実行します。
```scala
calibanGenSchema schemaPath outputPath [--scalafmtPath path] [--headers name:value,name2:value2] [--packageName name] [--effect fqdn.Effect]

calibanGenSchema project/schema.graphql src/main/MyAPI.scala
```

このコマンドは、 `outputPath` にScalaファイルを作成します。  
ここには`schemaPath`で定義された GraphQL スキーマにある全ての型が含まれています。    
ファイルの代わりに、URLを指定すると、内部参照を使用してスキーマが取得されます。

生成されたコードは、 `--scalafmtPath`オプションで定義された設定を使用して Scalafmt でフォーマットされます（デフォルト：` .scalafmt.conf`）。  
`schemaPath`のURLを与えた場合、` --headers`オプションを使用してリクエストヘッダーを指定することができます。

生成されたコードのパッケージは、 `outputPath`のフォルダーから分岐します。  
これは、 `--packageName`オプションで代替パッケージを指定するでオーバーライドできます。

デフォルトでは、それぞれの Query と Mutation は `zio.UIO`作用にラップされます。  
これは、 `--effect`オプションで代替作用を指定することでオーバーライドできます。  
