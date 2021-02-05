# Gradle  


## 依存性の確認  
build.gradle で引き込んだパッケージと、  
それに依存して引き込まれるパッケージを tree 形式で表示  
```sh
./gradlew dependencies
```

## 実行される依存タスクの除外

gradle コマンドからタスクを実行すると、依存タスクも実行される  
例えば、`build` を実行すると、以下のタスクが実行される
```
Selected primary task 'build' from project :
Tasks to be executed: [task ':compileJava', task ':processResources', task ':classes', task ':bootJarMainClassName', task ':bootJar', task ':jar', task ':assemble', task ':compileTestJava', task ':processTestResources', task ':testClasses', task ':test', task ':check', task ':build']
```

この中で、`compileJava` などは実行したいのだけれど、`test` は実行をしたくない場合、  
オプション引数で不要なタスクを実行しないようにできる。  

その場合、オプションの `--exclude-task`（または `-x`）を使用する
```sh
./gradle build --exclude-task test
```
