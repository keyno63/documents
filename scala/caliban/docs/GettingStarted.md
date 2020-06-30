# Getting Started

[原文](https://ghostdogpr.github.io/caliban/docs/)  

Caliban は Scala で GraphQl バックエンドを作成するための純粋な関数型ライブラリーです。  
[Magnolia](https://github.com/propensive/magnolia) を用いてデータ型から GraphQL スキーマを自動的に作成し、  
[Fastparse](https://github.com/lihaoyi/fastparse) を用いてクエリーを解析し、  
[ZIO](https://github.com/zio/zio) を用いて様々な作用を扱います。 


このライブラリの設計原理は次のとおりです。
* 純粋なインターフェイス: エラーや作用は明示的に返却され(例外は投げられることなく)、  
全ての返される型は参照透過です（`Future` でくるまれていません）。
* 「スキーマ定義」と「実装」の明確な分離： スキーマの定義と検証は、  
（リフレクションは使用せず）コンパイル時に Scala 標準の型を用いて行われ、  
リゾルバーは実行時に用いられる単純な値です。
* 最小のボイラープレート：APIで使う全ての型に対して、手動での定義は必要ありません。

Calibanは、フロントエンドのGraphQL構築にも使用できます。詳細については、専用セクションを参照してください。  
