# sangria

graph ql libs for scala.

# document  
https://sangria-graphql.org/learn/

# for use

## libs

 sbt に以下、依存ライブラリに以下追加（バージョンは 2020/06 現在）
 ```scala
libraryDependencies += "org.sangria-graphql" %% "sangria" % "2.0.0"
```

## sample

- akka example  
https://github.com/sangria-graphql/sangria-akka-http-example

- play example  
なさそう
公式ドキュメント曰、[sample server](http://try.sangria-graphql.org/)   があったらしいが、期限切れ  
理由は察してみることも必要

## 仕様

https://graphql.org/learn/serving-over-http/

- Request type  
POST で送信  

- Contents type  
```
application/json
```