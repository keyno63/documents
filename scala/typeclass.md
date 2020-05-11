# 型クラスについて

# はじめに
Scala というプログラミング言語に興味があったので、いろいろ調べたりしていました。  
その中で「型クラス」という言葉を何度か耳にしたのですが、全然理解できていませんでした。  

何度か知らべて「ああ、そういうことね」と分かった気になっていたのですが、  
それが気のせいだったと気付くのはいつもずっと後だったのです。

もしかしたら今回の「理解した」も気のせいかもしれませんが、  
理解したと思った以下の内容について書いてみます。
* implicit パラメーターによるオブジェクトの操作

# 型クラスとは
## 型クラスについて
そもそも型クラスとは？  
調べた中では以下の内容について書いてあるのがありました
* アドホック多相性
* 型のグループ化

単語をみるとわかったようなわからないような。  
ピンとこない理由を色々考えたのですが、ざっくりすると以下が頭の中で整理できてないのかなと思いました。
* 型クラスがあると何が嬉しいのか
* 何ができるのか（出来ないのか）
* 実際にどういうコードになっていればいいのか

## 実際にどういうコードになっていればいいのか
先にコードから書いてしまいますが、こんな感じです

```scala
trait Convert2String[T] {
  def convert(value: T): String
}  

val convertString = new Convert2String[String] {
  override def convert(value:  String): String = s"convert string ${value}"
}

val convertInt = new Convert2String[Int] {
  override def convert(value:  Int): String = s"convert int ${value.toString}"
}

def convert[T](value: T)(implicit ip: Convert2String[T]) = {
  println(ip.convert(value))
}

convert(100) // メソッドの呼出1
convert("sample String") // メソッドの呼出2
```

`Convert2String` というジェネリクス付きの trait を用意しておいて、  
ジェネリクスパラメーターが String 型の`convertString`, Int 型の`convertInt` を  
implicit パラメーターで実装,実体にしました。  
`convert` というメソッドがその implicit パラメーターを受け取って変換処理をやってくれる。  
という感じです。

## 型クラスがあると何が嬉しいのか

「ある型（のオブジェクト）に対して、操作ができる」という点です。  
この書き方だと普通みたいですね。ただ、他にいい書き出しが思いつかなかった。

オブジェクト指向の「ポリモーフィズム」のようなイメージです。

サンプルコードで書いた内容は、具体的には以下の挙動になります
```scala
// 関数呼び出し1
convert(100)(convertInt)
// => println(convertInt.convert(100))
// > "convert int 100"

// 関数呼び出し2
convert("sample String")(convertString)
// => println(convertString.convert("sample String"))
// > "convert string sample String"
```


ではオブジェクト指向の Interface ではダメなのか？という話なのですが、
Interface でも出来る事もあるし、出来ないこともあるというのが自分の中の結論です。

できること
自作クラスに振る舞い（メソッド）を追加する。というのは出来そうです。
こんな感じ
```scala
trait ToConvertString {
  def toConvert(): String
}

class RichInt(val value: Int) extends ToConvertString {
  override def toConvert(): String = s"convert int ${value.toString}" 
}

class RichString(val value: Int) extends ToConvertString {
  override def toConvert(): String = s"convert string ${value}" 
}

val ri = new RichInt(100)
println(ri.toConvert())
```
違いとしては「実装クラスの方にメソッドを追加、実行する」という部分ですかね。  
デメリットとしては継承したクラスの仕様に継承先が依存してしまうことかなと思います。  
`ToConvertString` の仕様が変わって、メソッドが追加とかされたら  
全てのクラスに追加しないといけなくなりそうです（人によってはそこまで問題ではないと思いますが）

できないこと
振る舞いのクラスを限定的にすること。は出来なさそうです。
自身と同じクラスだった場合に何かをしたい、としても無理そう
```scala
trait Compare {
  def same(value: Compare): String
}

class RichString(val value: String) extends Compare {
  override def same(value:  Compare): String = ??? // RichString を指定できない
  def same(value:  RichString): String = ??? // 一応書けるが、override ではない
}
```

```scala
trait Compare[T] {
  def same(value: T): String
}

case class RichString(val value: String)

implicit val compareString = new Compare[RichString] {
  override def same(value:  RichString): String = s"do something rich string ${value.toString}"
}

def same[T](value: T)(implicit compare: Compare[T]) = {
  println(compare.same(value))
}

val rs = RichString("rich string")
same(rs)
```
## 何ができるのか（出来ないのか）

「出来ないこと」はちょっとわかりませんでした。  
「類似の機能ではさらにこういうことが出来る」とか、  
「ここまではできるけど、さらにこういうことができといいが、それはできない」、  
という内容をまとめられるとよかったのですが、それはちょっとわかりません。

