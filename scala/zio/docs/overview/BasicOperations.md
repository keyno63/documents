# BasicOperations

[原文](https://zio.dev/docs/overview/overview_basic_operations)

## マッピング

`ZIO#map` メソッドを呼び出すことで エフェクトの成功チャンネルにマッピングすることができます。  
これによりエフェクトの成功値を変換することができます。
```scala
import zio._

val succeeded: UIO[Int] = IO.succeed(21).map(_ * 2)
```

`ZIO#mapError` メソッドを呼び出すことで、エフェクトのエラーチャンネルにマッピングすることができます。
これによりエフェクトの失敗値を変換することができます。
```scala
val failed: IO[Exception, Unit] =
  IO.fail(“No no!”).mapError(msg => new Exception(msg))
```

EitherのマッピングがEitherがLeft、およびRightを変更しないのと同じように、エフェクトの成功、およびエラーチャネルのマッピングは、エフェクトの成功、および失敗を変更しないことに注意してください。

## 連鎖

`flatMap` メソッドにより2つのエフェクトを順番に実行することができます。
これには最初のエフェクトを受け取り、この値に依存する2番目のエフェクトを返すことができるコールバックを渡す必要があります。
```scala
val sequenced =
  getStrLn.flatMap(input => putStrLn(s"you entered: $input"))
```

最初のエフェクトが失敗し、`flatMap` に渡したコールバックは呼び出されず、`flatMap` から返された合成エフェクトも失敗します。

どのようなエフェクトの連鎖でも、例外をスローするとステートメントのシーケンスが途中で終了するのと同じように、最初の障害が連鎖全体を短絡させます。

## 理解のために

ZIO データ型は `flatMap` 、および `map`の両方をサポートしているため、  
連続したエフェクトを構築するのに Scala の `for` で包むことができます。
```scala
val program =
  for {
    _    <- putStrLn("Hello! What is your name?")
    name <- getStrLn
    _    <- putStrLn(s"Hello, ${name}, welcom to ZIO!")
  }
```

## 圧縮

`ZIO#zip` をメソッドを使うことで2つのエフェクトを1つのエフェクトに結合することができます。  
結果としてのエフェクトは両方のエフェクトの成功した値を含むタプルで成功します。
```scala
val zipped: UIO[(String, Int)] = 
  ZIO.succeed("4").zip(ZIO.succeed(2))
```

`zip` は連続的に実行されることに注意してください。Left側のエフェクトはRight側のエフェクトよりも先に実行されます。  
Left側、およびRight側の両方の値でタプルを構成する必要があるため、どちらかが失敗するとどの `zip` 操作でも合成エフェクトは失敗します。  
場合によっては、エフェクトの成功値をつかうことができない場合（または例として、Unitである場合）、  
`ZIO#zipLeft`、または`ZIO#zipRight` 関数を使用する方が便利な場合があります。  
これは最初に `zip` を実行し、どちらか一方を破棄してタプルに変換します。

## 次のステップ

ZIOエフェクトの基本的な操作に慣れてきたら、  
次のステップでエラー処理について学びます。
