# HandlingErrors

[原文](https://zio.dev/docs/overview/overview_handling_errors)  

このセクションでは失敗の発見、および対応するいくつかの一般的な方法についてみていきます。

## Either

`ZIO#either` を使うことで 失敗を表面化することができます。  
これは`ZIO[R, E, A]` をつかい、`ZIO[R, Nothing, Either[E, A]]` を生成します。  
```scala
val zeither: UIO[Either[String, Int]] =
  IO.fail("Uh oh!").either
```

`ZIO.absolve`   を使うことで失敗を覆い隠すことができます。  
これは`either` の反対であり、`ZIO[R, Nothing, Either[E, A]]` を `ZIO[R, E, A]` に変換します。  
```scala
def sqrt(io: UIO[Double]): IO[String, Double] =
  ZIO.absolve(
    io.map(value =>
      if (value < 0.0) Left("Value must be >= 0.0")
      else Right(Math.sqrt(value))
    )
  )
```

## 全てのエラーを捕捉する

全てのエラー型を捕捉して復旧したくなり、効果的に復旧を試みたい場合、`catchAll`メソッドを使うことができます。
```scala
val z: IO[IOException, Array[Byte]] =
  openFile("primary.json").catchAll(_ =>
    openFile("backup.json"))
```

コールバックを `catchAll` に渡された場合、異なるエラー型（もしくは `Nothing` ）と一緒にエフェクトを返す可能性があります。  
これは `catchAll` から返されるエフェクトの型に影響されます。

## いずれかのエラーを捕捉する

いずれかの例外の型のみを捕捉して復旧したくなり、効果的に復旧を試みたい場合、`catchSome`メソッドを使うことができます。
```scala
val date: IO[IOException, Array[Byte]] =
  openFile("primary.date").catchSome {
    case _: FileNotFoundException =>
      openFile("backup.data")
  }
```

`catchAll` とは違い、 `catchSome` はエラー型を削減したり排除したりはできません。  
しかし、エラーの種類をより広い範囲のエラーに広げることができます。

## フォールバック

`orElse` コンビネータを使用して、1つの効果を試すか、失敗した場合は別のエフェクトを試すことができます。

```scala
val primaryOrBackupData: IO[IOException, Array[Byte]] = 
  openFile("primary.data").orElse(openFile("backup.data"))
```

## 畳み込み

Scala の `Option`、および  `Either`  データ型は成功と失敗を同時に操作できる `fold`メソッドを持っています。
似たような方法として、`ZIO` エフェクトにも失敗と成功の両方を操作するいくつかのメソッドを持っています。
最初のメソッド `fold` は非エフェクト的に両方の値を操作します。 ケース毎にエフェクトのないハンドラーを提供します。
```scala
lazy val DefaultData: Array[Byte] = Array(0, 0)

val primaryOrDefaultData: UIO[Array[Byte]] =
  openFile("primary data").fold(
    _    => DefaultData,
    data => data)
```

2番目の畳み込みメソッド `foldM` は両方の値をエフェクト的に操作します。  
それぞれのケースに対して（純粋なまま）エフェクトのあるハンドラーを提供します。
```scala
val primaryOrSecondaryData: IO[IOException, Array[Byte]] =
  openFile("primary.data").foldM(
    _       => DefaultData,
    data => data)
```
 ほとんどすべてのエラー操作のメソッドは、強力かつ高速なため、`foldM` の条件で定義されています。  

以下の例では、`foldM` は `readUrls` メソッドの両方の値を操作するのに使われています。
```scala
val urls: UIO[Content] =
  readUrls("urls.json").foldM(
    error   => IO.succeed(NoContent(error)), 
    success => fetchContent(success)
  )
```

## リトライ

失敗したエフェクトをリトライするため、ZIOデータ型にはいくつかの有用なメソッドがあります。  
もっとも基本的なものは `ZIO#retry` です。  
これは`Schedule` を取り、指定したポリシーに従って、 失敗した場合に最初のエフェクトをリトライする新しいエフェクトを返します。
```
import zio.clock._

val retriedOpenFile: ZIO[Clock, IOException, Array[Byte]] =
  openFile(primary.data).retry(Schedule.recurs(5))
```

次に強力な機能は `ZIO#retryOrElse` です。  
これはエフェクトが指定したポリシーで成功しなかった場合、 指定した フォールバックを使用することができます。
```scala
openFile("primary.data").retryOrElse(
    Schedule.recurs(5),
    (_, _) => ZIO.succeed(DefaultData))
```

最後のメソッド `ZIO#retryOrElseEither` はフォールバックに対して異なる型を返すことができます。

スケジュールを構築することに関して、さらに知りたい場合は [Schedule]() のドキュメントを参照してください。


## 次のステップ

基本的なエラー操作することに慣れてきたら、  
次のステップで安全にリソース操作について学びます。
