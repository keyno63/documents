# Federation

フェデレイション（連携）

** Federation **はオプションのモジュールで、設定に組み込んでフェデレーションスキーマに登録できます。

## 依存

`caliban-federation` は `caliban-core` 依存し、非常に控えめです。  

使うために、 `build.sbt` ファイルに以下の行を追加します。  

```
libraryDependencies += "com.github.ghostdogpr" %% "caliban-federation" % "0.9.0"
```

## フェデレイション

フェデレイションにより、モデルを共有したり、ゲートウェイレベルで脆弱なコードで繋がったようなスキーマを作成したりすることなく、  
グラフをより大きなグラフの一部にすることができます。

フェデレイションについて更に詳しく知りたければ、 [ここ](https://www.apollographql.com/docs/apollo-server/federation/introduction/) が有用かもしれません。

フェデレイションは、既存のスキーマのラッパーを作成し、ゲートウェイとの相互作用をサポートするために必要なフックを追加できるようにします。  

すでにグラフがある場合は、ラッパー関数 `federate` を呼び出すだけでフェデレイションを追加できます。  

```scala
import caliban.federation._

val schema: GraphQL[R] = graphQL(RootResolver(Queries(
  characters = List(Character("Amos"))
)))

val federatedSchema: GraphQL[R] = federate(schema)
```

これにより、最小限のスキーマ追加が API に対してラップされ、ゲートウェイがスキーマを認識できるようになります。  
実際にエンティティーの解決を可能にするには、少し地道な作業を行う必要があります。

最初に、「解決可能な」型は全て、 `@key` 命令でアノテーションを付ける必要があります。  
`federation` パッケージにあるヘルパー関数を使うことで補助することができます。  

```scala
@GQLDirective(Key("name"))
case class Character(name: String)
```

`"name"` 、フィールドセレクターから外側の括弧を取り除いたものです。

別のサービスから型を拡張する必要がある場合は、現在のサービスで型のスタブバージョンを定義し、  
`@extends` アノテーションで注釈を付ける必要があります。

```scala
@GQLDirective(Key("season episode")) 
@GQLDirective(Extend)
case class Episode(@GQLDirective(External) season: Int, @GQLDirective(External) episode: Int, cast: List[Character])
```

この場合に必要な追加される注釈に注意してください。  
この型が別サービス内で定義されていることをゲートウェイに伝えるには、 `Extend` が必要です。  
`External` はこれらのフィールドに他サービスが所有しているかどうかのフラグを付けます。  
読んだ方がよい利用可能ないくつか別の注釈が存在します。  

型に注釈を付けたら、それらの型を解決する方法を `Federation` に通知する必要があります。  
フェデレーションは、標準的な GraphQL クエリーから型を解決する場合に、少し異なる仕組みを使用します。  
そのため、サポートする型ごとに、 `EntityResolver` を追加する必要があります。
```scala
EntityResolver[CharacterService, CharacterArgs, Character](args => 
  ZQuery.fromEffect(characters.getCharacter(args.name))
)  
```

上記では、 環境型 `R` をとるリゾルバーを定義する必要があります。  
implicit な `ArgBuilder` と `Option [Out]` を持つ `A`。 `Out` には implicit な `Schema[R、Out]` があります。  
上記を作成すると、次のようにこれらのリゾルバーをフェデレイションスキーマに追加できます。  

```scala
federate(schema, aResolver, additionalResolvers:_*)
```

クエリーの実行開始のために結果の `GraphQL[R]` を使えるようになりました。  
完全なコードサンプルとして [ここ](https://github.com/ghostdogpr/caliban/tree/master/examples/src/main/scala/caliban/federation) を参照することができます。
