# HandlingResources

[原文](https://zio.dev/docs/overview/overview_handling_resources)  

このセクションではZIOを使って安全にリソースを操作する一般的な方法をいくつかみていきます。
 ZIOのリソース管理機能は同期、非同期、 並行、および他のエフェクト型にわたって動作し、  
 アプリケーションに障害、中断、または欠陥があった場合でも強力な保証を提供します。

## ファイナライザー

`ZIO#ensuring` メソッドを使うことで、 ZIO は  `try`、および `finally` と似た機能を提供します。  
`try`、 `finally` のように、保証化機能はエフェクトの実行が開始され後、（なんらかの理由で）中断されたときにファイナライザーが実行を開始します。  
```scala
val finalizer =
  UIO.effectTotal(println("Finalizing!"))
// finalizer: UIO[Unit] = zio.ZIO$EffectTotal@54646a0e

val finalized: IO[String, Int] =
  IO.fail("Failed!").ensuring(finalizer)
// finalized: IO[String, Unit] = zio.ZIO$ZIO$CheckInterrupt@4f4adf76
```

ファイナライザーは失敗することはできません。これは内部的にすべてのエラーを制御しないといけないことを意味します。  

`try`、`finally` と同様に、ファイナライザーはネストすることができます。そして内部のファイナライザーの失敗は他のファイナライザーに影響を与えません。  
ネストされたファイナライザーは、逆順かつ線形に実行されます（並列ではなく）。  

`try`、`finally` とは異り、 非同期、同期を含む、エフェクトのすべての型を確実に機能します。

## ブラケット

`try`、`finally` の一般的な用途は、新しいソケット接続や開かれたファイルなどのリソースを安全に取得および解放することです。
```scala
val handle = openFile(name)

try {
  processFile(handle)
} finally closeFile(handle)
```

ZIOはこの共通パターンを`ZIO＃bracket` でカプセル化します。  
これにより、リソースを取得する取得エフェクト、リソースを解放する解放エフェクト、リソースを使用する使用エフェクトを指定できます。  

エラーや中断があった場合でも、解放エフェクトは実行環境システムによって実行されることが保証されています。
```scala
val groupedFileData: IO[IOException, Unit] = 
  openFile("data.json").bracket(closeFile(_)) { file =>
    for {
      data    <- decodeData(file)
      grouped <- groupData(data)
    } yield grouped
  }

```
確認と同様に、ブラケットには構成セマンティクスがあるため、  
1つのブラケットが別のブラケット内にネストされ、外側のブラケットがリソースを取得する場合、外側のブラケットの解放が常に呼び出されます。  
例として、外側のブラケットのリリースが失敗した場合が該当します。

## 次のステップ

リソース操作に慣れてきたら、
次のステップでは基本的な並行について学びます。
