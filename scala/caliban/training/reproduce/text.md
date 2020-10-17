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
