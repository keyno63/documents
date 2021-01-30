# Hot Reload 設定方法

SpringBoot で Hot Reload （自動更新）の設定方法

## 依存関係の追加

`gradle.build` に以下を追加します。
```gradle
dependencies {
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
}
```

開発環境でのみつかえるので、developmentOnly で追加します

## IDE の設定

IDE から起動する場合、自動ビルドなどの設定が必要になります。  
Intellij IDEA の場合、以下が必要
- 自動ビルド設定の有効
- Registry 設定で実行時の自動コンパイル設定の有効

### 自動ビルド設定
[File] -> [Settings] で設定画面起動  
[Build, Execution, Development] -> [Compiler] を選択  
以下の項目をチェックし、有効化します。  
`Build project automatically`

### 自動コンパイル設定
[Preferences...] 
Ctr + Shift + A を押し、Action 検索から Registry を検索する.  
以下の項目にチェックを入れます  
`compiler.automake.allow.when.app.running`

## 起動

起動するときは bootRun から起動します  
- mac/linux  
`./gradlew bootRun`  
- Win  
`./gradlew.bat bootRun`  
  
