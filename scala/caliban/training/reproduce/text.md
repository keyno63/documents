# 模写

caliban の模写した時のメモ  
クラス単位で模写していた

## class

### GraphQL

基本構成について
メンバー変数として以下の３つが必要
- schemaBuilder
- wrappers
- additionDirectives

schemaBuilder, additionDirectives は  
生成メソッドに渡された変数から生成・代入をする  
wrappers　は一旦からの状態で持つ


以下の class を先に定義する必要があった
- SubscriptionSchema
- RootSchemaBuilder
- Schema
- __Directive
- Wrapper

実装機能について
実装されている機能は以下
- render
  定義した gql スキーマの表示
- toDocument
  gql のドキュメント化
- interpreter
  interpreter（変換器）の生成  
  Validationを実施してから生成が行われる
- withWrapper
  Wrapper の連結（というか、追加）
- @@
  withWrapper メソッドの alias
- combine
  GraphQL の連結
- |+|
  combine メソッドの alias
- rename
  root query の再定義

追加で以下のクラス定義などが必要
- GraphQLRequest
- GraphQLResponse
- GraphQLInterpreter
- CalibanError
- Document
- SchemaDefinition
- SourceMapper
- Validator
- Parser
- OperationType

### GraphQLRequest

生成器として case class  
以下の class 定義が先に必要
- InputValue

object GraphQLRequest を定義  
implicit 変数を宣言するための object っぽい。
後続で GraphQLRequestCirce, GraphQLRequestPlayJson を定義しているので、  
それを使う

JSON parser の戻り値は interop package に書く

### Json(circe/playJson)

IsCirceDecoder, IsPlayJsonReads などを定義する  

以下の class などを先に定義しておかないといけない  
- Schema
- __Type
- schema.Types
- Step
- InputValue
- ResponseValue
- Step.PureStep

