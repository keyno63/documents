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
