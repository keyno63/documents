# GraphQL Client
GraphQL クライアント

**  Caliban-client** は Caliban から独立したモジュールで、型安全で関数的手法な Scala コードを使用して GraphQL クエリーを記述できるようにします。
これは [sttp]（https://github.com/softwaremill/sttp） の上に構築されているため、選択したバックエンドを使用してリクエストを実行できます。

Caliban と同様、 `caliban-client` は純粋関数的なインターフェースを提供し、ボイラープレートを最小限に抑えます。  
これは次のように働きます。  
1. `caliban-codegen-sbt` ツールを使うことで、与えられた GraphQL スキーマからボイラープレートコードを生成します  
2. 生成されたコードからヘルパーを結合することで GraphQL クエリー/ミューテーション を記述します  
3. クエリー/ミューテーションを `sttp` リクエストに変換し、好きなバックエンドで実行します

## 依存

`caliban-client` を使うため、 `build.sbt` ファイルに以下を行を追記します。  

```
libraryDependencies += "com.github.ghostdogpr" %% "caliban-client" % "0.8.3"
```

Caliban-client は ScalaJS に対応しています。

## コード生成

`caliban-client` を用いて GraphQL クエリーを作成するための最初のステップは、GraphQL スキーマからボイラープレートコードを生成することです。  
そのためには、スキーマを含むファイルが必要です（バックエンドが `caliban` を使用している場合、API 上で`GraphQL＃render` を呼び出すことで取得できます）。

この機能を使用するには、 sbt プラグインである `caliban-codegen-sbt` をプロジェクトに追加して有効にします。  

```scala
addSbtPlugin("com.github.ghostdogpr" % "caliban-codegen-sbt" % "0.8.3")
enablePlugins(CodegenPlugin)
```
次に sbt コマンド `calibanGenClient` を呼び出します。  
```scala
calibanGenClient schemaPath outputPath [--scalafmtPath path] [--headers name:value,name2:value2]

calibanGenClient project/schema.graphql src/main/Client.scala
```

このコマンドは、`schemaPath` で定義された GraphQL スキーマで定義された全ての型に対して、ヘルパー関数を含む `outputPath` に Scala ファイルを生成します。  
ファイルの代わりに、内部検査を使うことで得られる URL とスキーマ を作ることができます。  
生成されたコードは `--scalafmtPath` オプションで定義される設定を使って、 Scalafmt でフォーマットできます（デフォルトでは `.scalafmt.conf` を使用します）。  
`schemaPath` から URL を生成すると、`--headers` オプションでリクエストヘッダーを与えられます。  
生成コードのパッケージは `outputPath` のフォルダーから決定されます。  
このパッケージ名は `--packageName` オプションを使うことで上書きすることも可能です。
