# Validation
検証

Caliban は `gqldoc` と呼ばれる小さなマクロを提供しています。  
`gqldoc` により、 **コンパイル時** に GraphQL クエリー（正確にはドキュメント）の構文が正しいかを検証できます。  

```scala
import caliban.Macros.gqldoc

val query = gqldoc("""
  query test {
    amos: character(name: "Amos Burton") {
      name
    }
  }""")
```

**実行時**、API上で `check` メソッドを呼び出すことにより、スキーマに対してクエリーを検証できます。

```scala
def check(query: String): IO[CalibanError, Unit]
```

また、 `execute`を呼び出すときに` skipValidation = true`を渡すことで、クエリーの実行時に検証をスキップすることもできます（アダプターでも使用可能）。  
これにより、パフォーマンスが少し向上します。
