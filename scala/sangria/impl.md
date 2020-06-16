# impl
sangria の実装方法をまとめる

play で動かす場合

## build.sbt
おなじみの設定から play の引き込み

project/plugins.sbt
```scala
resolvers += "Typesafe repository" at "https://repo.typesafe.com/typesafe/releases/"

addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.8.1")
```

必要なライブラリを build.sbt に追加します.  
必要、というのは、以下のライブラリ
- play関係 (play plugin, guice)  
- graphql 関係 (org.sangria-graphql~)
- json parser (circe を使用)

build.sbt
```scala
lazy val `scala_play` = (project in file(".")).enablePlugins(PlayScala)

libraryDependencies ++= Seq(specs2 % Test , guice ) ++
  Seq(
    // sangria
    "org.sangria-graphql" %% "sangria" % "2.0.0",
    "org.sangria-graphql" %% "sangria-slowlog" % "0.1.8",
    "org.sangria-graphql" %% "sangria-circe" % "1.2.1",
    "org.sangria-graphql" %% "sangria-spray-json" % "1.0.2",
    // circe(json libs)
    "io.circe" %% "circe-core" % "0.12.1",
    "io.circe" %% "circe-parser" % "0.12.1",
    "io.circe" %% "circe-generic" % "0.12.1",
    "io.circe" %% "circe-optics" % "0.9.3"
  ) :+ ("com.dripower" %% "play-circe" % "2812.0")
```

バージョンは 2020/06 のものを使用

