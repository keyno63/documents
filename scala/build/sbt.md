# sbt
build ツール
Lightbend 製の公式ビルドツール

## インストール

sdkman でインストール
```
sdk list java
sdk install java 11.0.4.hs-adpt
sdk install sbt
```

バージョン指定インストール、更新
```
sdk ls sbt
sdk install sbt 1.3.12
```
当事最新は 1.3.12

## 実行

### sbt シェル
インタラクティブモード、対話形式で実行をすすめる
```
$ sbt
[info] Loading global plugins from /home/XXXXX/.sbt/1.0/plugins
[info] Loading project definition from /home/XXXXX/project
[info] Set current project to XXXXX (in build file:/home/XXXXX/)
[info] sbt server started at local:///home/XXXXX/.sbt/1.0/server/1f0c0b47a789295c47d9/sock
sbt:XXXXX>
```

### バッチモード
引数に値をわたして sbt に処理を実行してもらう
```
$ sbt build
```

## コマンド一覧
