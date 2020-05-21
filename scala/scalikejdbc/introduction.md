# Scalikejdbc の使い方

Scala っぽくかける jdbc ライブラリ(DB 接続、操作)   
一旦自分の環境用に記載する  

## 最初に

使い方としては以下参照  
* [公式サイト](http://scalikejdbc.org/)  あり  
* [scalikejdbc-cookbook](https://github.com/scalikejdbc/scalikejdbc-cookbook) というのもある  

## 導入

scalikejdbc と db ドライバが必要(h2 の時はいらんのやっけ？)  
build.sbt に以下
```
    "org.scalikejdbc" %% "scalikejdbc" % "3.4.1",
    "org.scalikejdbc" %% "scalikejdbc-config"           % "3.4.1",
    "org.scalikejdbc" %% "scalikejdbc-play-initializer" % "2.8.0-scalikejdbc-3.4",
    "org.postgresql" % "postgresql" % "42.2.12"
```
postgresql を使いたいので、この構成です  
version は 2020/05 現在のもの  

## 使い方
まずは普通に class, object から呼び出す方法  
### 初期化
```scala
    import scalikejdbc._

    // initialize JDBC driver & connection pool
    Class.forName("org.postgresql.Driver")

    // 接続先の情報
    val url = "jdbc:postgresql://localhost:5432/sample_db"
    val user = "fujiwara"
    val pass = "fujiwara"
    ConnectionPool.singleton(url, user, pass)
```
これは以下の DB に接続する場合の書き方  
* host : localhost
* port : 5432
* db 名 : sample_db
* user 名 : fujiwara
* pass : fujiwara

### SQL の書き方

以下のパターン  
* DML
* SQL インターポレーション
* QueryDSL(type safe query buikder)
* Auto Macro

オススメは「SQL インターポレーション」らしい
