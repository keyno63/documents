# scalafmt

scala 用の formatter.  


## plugin の追加  

`<project root>/project/plugins.sbt` に以下を追加  
```sbt
addSbtPlugin("org.scalameta"      % "sbt-scalafmt"             % "2.4.2")
```

## 設定ファイル
ルートに `.scalafmt.conf` を配置.

### サンプル
使いたいやつ
```
version = "2.4.2"

maxColumn = 120
align = most
continuationIndent.defnSite = 2
assumeStandardLibraryStripMargin = true
docstrings = JavaDoc
lineEndings = preserve
includeCurlyBraceInSelectChains = false
danglingParentheses = true
spaces {
  inImportCurlyBraces = true
}
optIn.annotationNewlines = true

rewrite.rules = [SortImports, RedundantBraces]
```

## 実行  

sbt による実行.
- `sbt scalafmtSbt`: `.sbt` ファイルの format を実行
- `sbt scalafmt`: ソースコードの fotmat を実行  
