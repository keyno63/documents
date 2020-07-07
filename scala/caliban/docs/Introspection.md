# Introspection
内部検査

内部検査クエリーは完全にサポートされています。  
つまり、お気に入りのツールを使用してスキーマを検査し、無料でドキュメントを生成できます。

[Altair GraphQL Client](https://altair.sirmuel.design/) の内部検査によって生成されたドキュメントの例が以下になります。  

![altair screenshot](/caliban/altair.png)

クエリー実行時に、 `execute`を呼び出した際に `enableIntrospection = false` を渡すことで内部検査を無効にすることができます（アダプターでも使用可能）。
