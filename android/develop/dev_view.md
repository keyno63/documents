# 画面の開発

## 画面の作成
### 最初の画面を作る場合

Project の作成から作れる.  

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

### 画面の追加

[File] -> [New] -> [Activity] から追加したい画面を追加する.  

```kotlin
class SubActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

### 画面遷移について  
呼び出し元の Intent を初期化して、Activity を開始する.  

ボタンを押して移動したい場合は  
画面にボタンを追加し、 `onClick` 処理に以下のメソッドを追加する  
```kotlin
fun onButtonStart(view: View?) {
    val intent = Intent(this, SubActivity::class.java)
    startActivity(intent)
}
```

戻りたい場合は、移動先の画面に以下のメソッドを追加し、  
ボタンなどのアクション処理（`onClick` など）から呼び出す
```kotlin
fun onButtonStart(view: View?) {
    finish()
}
```