# CreatingEffects

[原文](https://zio.dev/docs/overview/overview_creating_effects)

このセクションでは値、共通のScala型、同期／非同期の副作用から  
ZIOエフェクトを作る共通の方法をいくつか説明します。

## 成功値から
`ZIO.succeed` メソッドを使えば、指定された値を成功としたエフェクトとしてつくることができます。
```scala
val s1 = ZIO.succeed(42)
```

ZIO型エイリアスのコンパニオンオブジェクトとしてメソッドを使うこともできます。
```
val s2: Task[Int] = Task.succeed(42)
```

succeeed メソッドは名前付き引数を取り、  値の作成による偶発的な副作用を ZIO ランタイムによって適切に管理できるようにします。  
しかし、 succeeed は副作用のない値を対象としています。  
値が副作用をもっていることが分かっている場合は、 ZIO.effecTotal を使用することを検討してください。
```scala
val now = ZIO.effectTotal(System.currentTimeMillis())
```

ZIO.effectTotalから作成された成功エフェクトに含まれた値は、どうしても必要な場合にのみ作成されます。

## 失敗値から

`ZIO.fail` メソッドを使えば、失敗をモデル化するエフェクトを作ることができます。
```scala
val f1 = ZIO.fail("Uh oh!")
```

`ZIO` データ型に対して、エラー型の制限はありません。  
アプリケーションに適した文字列、例外、またはカスタムデータ型を使用できます。  
`Throwable` 、および `Exception` を拡張したクラスを用いて、多くのアプリケーションは失敗をモデル化します。
```scala
val f2 = Task.fail(new Exception("Uh oh!"))
```

他のエフェクトコンパニオンオブジェクトとは異なり、 `UIO` 値は失敗しないため、  
`UIO` コンパニオンオブジェクトには `UIO.fail` を持たないことに注意してください。

##  Scala の値から

Scalaの標準ライブラリーにあるデータ型には、ZIOエフェクトに変換できるものがあります。

### Option

### Either

### Try

### Function

### Future

## 副作用から

## 同期的な副作用をブロッキング
