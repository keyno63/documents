# Introspection
イントロスペクション

caliban は「イントロスペクション」クエリーを完全にサポートしています。  
つまり、お気に入りのツールを使用してスキーマを検査し、自由にドキュメントを生成できます。

[Altair GraphQL Client](https://altair.sirmuel.design/) のイントロスペクションによって生成されたドキュメントの例が以下になります。  

![altair screenshot](/caliban/altair.png)

クエリー実行時に、 `execute`を呼び出した際に `enableIntrospection = false` を渡すことで  
イントロスペクションを無効にすることができます（アダプターでも使用可能）。
