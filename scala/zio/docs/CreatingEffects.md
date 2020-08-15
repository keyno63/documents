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
`Throwable` 、または `Exception` を拡張したクラスを用いて、多くのアプリケーションは失敗をモデル化します。
```scala
val f2 = Task.fail(new Exception("Uh oh!"))
```

他のエフェクトコンパニオンオブジェクトとは異なり、 `UIO` 値は失敗しないため、  
`UIO` コンパニオンオブジェクトには `UIO.fail` を持たないことに注意してください。

##  Scala の値から

Scalaの標準ライブラリーにあるデータ型には、ZIOエフェクトに変換できるものがあります。

### Option

`Option` は`ZIO.fromOption` を使うことで ZIO エフェクトに変換することができます。  
```scala
val zoption: IO[Option[Nothing], Int] = ZIO.fromOption(Some(2))
```

結果として生じるエフェクトのエラー型は `Option[Nothing]` であり、値が存在しない理由に関する情報は与えられません。  
`ZIO#mapError` を用いれば、`Option[Nothing]`をより特異なエラー型に変換することができます。  
```scala
val zoption2: IO[String, Int] = zoption.mapError(_ => “It wasn’t there!”)
```

結果のオプションの性質を維持しながら、他の演算子で簡単に構成することもできます（これはOptionTと同様です）。
``` scala
val maybeId: IO[Option[Nothing], String] = ZIO.fromOption(Some("abc123"))
def getUser(userId: String): IO[Throwable, Option[User]] = ???
def getTeam(teamId: String): IO[Throwable, Team] = ???


val result: IO[Throwable, Option[(User, Team)]] = (for {
  id   <- maybeId
  user <- getUser(id).some
  team <- getTeam(user.teamId).asSomeError 
} yield (user, team)).optional 
```

### Either

`Either` は `ZIO.fromEither` を使うことで ZIO エフェクトに変換することができます。  
```scala
val zeither = ZIO.fromEither(Right(“Success!”))
```

結果として生じるエフェクトのエラー型は、`Left`の場合にもつ型の何らかになり、  
成功した型は`Right` の場合にもつ型の何らかになります。

### Try

`Try` 値は `ZIO.fromTry` を使うことで `ZIO` エフェクトに変換することができます。
```scala
import scala.util.Try
val ztry = ZIO.fromTry(Try(42/0))
```

結果によるエフェクトのエラー型は常に `Throwable`になります。  
なぜなら `Try` は `Throwable` 型の値でのみ失敗するためです。

### Function

関数 `A => B` は `ZIO.fromFunction` を使うことで `ZIO` エフェクトに変換することができます。
```
val zfun: URIO[Int, Int] =
  ZIO.fromFunction((i: Int) => i *i) 
```

エフェクトの環境型は（関数のインプット型である）Aになります。  
なぜならエフェクトの実行するために、この型の値を指定する必要があるためです。


### Future

`Future` は `ZIO.fromFuture` を使うことで ZIO エフェクトに変換することができます。  
```scala
import scala.concurrent.Future

lazy val future = Future.successful(“Hello!”)

val zfuture: Task[String] =
  ZiO.fromFuture { implicit ec =>
    future.map(_ => "Goodbye!")
  }
```

`fromFuture` に渡される関数には、`ExecutionContext` が渡されます。  
これにより、ZIOは `Future` が実行される部分を管理できます（もちろん、この`ExecutionContext` は無視できます）。  

結果として生じるエフェクトのエラー型は常に `Throwable` になります。  
なぜなら `Future` は `Throwable` 型の値でのみ失敗する可能性があるためです。

## 副作用から

ZIO は同期／非同期に関らずに副作用を ZIO エフェクト（pure の値）に変換することができます。  
これらの関数を使用して手続き型コードをラップすることにより、  
レガシーな Scala、または Java コードだけでなく、サードパーティーのライブラリーに対しても  
すべてのZIOの機能をシームレスに使用できます。

### 同期的な副作用

同期的な副作用は `ZIO.effect` を使うことで ZIO エフェクトに変換することができます。  
```scala
import scala.io.StdIn

val getStrIn: Task[String] = 
  ZIO.effect(StdIn.readLine())
```

結果によるエフェクトのエラー型は常に `Throwable`になります。  
なぜなら副作用は `Throwable` 型のなんらかの値による例外を投げる可能性があるためです。  


もし与えられた副作用が何も例外が投げられていないことがわかっていれば、  
`ZIO.effectTotal`  を使うことで副作用は ZIO エフェクトに変換することができます。  
```scala
def putStrLn(line: String): UIO[Unit] =
  ZIO.effectTotal(println(line))
```

`ZIO.effectTotal` を使う時は注意してください。副作用が完全であることが疑わしい場合、エフェクトを変換するのに `ZIO.effect` を優先してください。  
（他のエラーを致命的なものとして扱うことによって）エフェクトのエラー型を調整したい場合、`ZIO＃refineToOrDie` メソッドを使用することができます。  
```scala
import java.io.IOException
val getStrLn2: IO[Exception, String] =
  ZIO.effect(StdIn.readLine()).refineToOrDie[IOException]
```

### 非同期的な副作用

コールバックベースAPIを用いた非同期的な副作用は`ZIO.effectAsync`を使うことで ZIO エフェクトに変換することができます。
```scala
object legacy {
  def login(
    onSuccess: User => Unit,
    onFailure: AuthError => Unit): Unit = ???
}

val login: IO[AuthError, User] =
  IO.effectAsync[AuthError, User] { callback =>
      legacy.login(
        user => callback(IO.succeed(user)),
        err    => callback(IO.fail(IO.fail(err))
      )
    }
```
 
非同期的なZIOエフェクトはコールバックベースAPIより扱いやすく、  
相互作用、リソース安全性、エラー操作のようなZIOの特性による利益があります。

## ブロッキング同期的な副作用

一部の副作用はブロッキングIOを使用するか、スレッドを待機状態にします。  
注意深く管理しないと、これらの副作用により、アプリケーションのメインスレッドプールからスレッドが枯渇し、 ワーキングスレッドが不足する可能性があります。

ZIO には `zio.blocking` パッケージが存在し、  
ブロッキングする副作用をZIOエフェクトに安全に変換して使うことができます。

ブロッキング副作用は `effectBlocking`メソッドを用いて ZIOエフェクトブロッキングに変換することが可能です。
```scala
import zio.blocking._
 
val sleeping = 
  effectBlocking(Thead.sleep(Long.MaxValue))
```

結果によるエフェクトはブロッキングエフェクト専用に設計されたスレッドプールで実行されます。  

`effectBlockingInterrupt`メソッドを使うことで `Thread.interrupt` の呼び出し、ブロッキング副作用は中断されます。

一部のブロッキング副作用の中断は取消エフェクトの呼び出しからのみ行われます。  
`effectBlockingCancelable` メソッドを使うことでこれらの副作用を変換することができます。
```scala
import java.net.ServerSocket
import zio.UIO
 
def accept(l: ServerSocket) =
  effectBlockingCancelable(l.accept())(UIO.effectTotal(l.close()))
```

副作用がすでに ZIO エフェクトに変換されている場合、`effectBlocking` の代わりに、  
`blocking` メソッドを使用して、ブロッキングスレッドプールで確実にエフェクトが実行されるようにすることができます。
```scala
import scala.io.{ Codec, Source }

def download(url: String) =
  Task.effect {
    Source.fromURL(url)(Codec.UTF8).mkString
  }

def safeDownload(url: String) =
  blocking(download(url))
```

## 次のステップ

値、Scala データデータ型、および副作用からエフェクトを作成することに慣れてきたら、  
次のステップで、エフェクトの基本的な操作を学びます。
