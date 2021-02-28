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

## 別の画面へのパラメーター渡し

別の画面への遷移時、前の画面処理で取得した値を  
次の画面で取得する方法.  

パラメーターを渡しているというより、前の画面で変数化し、  
次の画面で参照しているような？

```kotlin
class MainActivity: AppCompatActivity() {
    fun onButtonSearch(view: View?) {
        val intent = Intent(this, SearchActivity::class.java)

        // 次の画面で取得したい値の定義
        val text = "any_value"

        // 取得するための値の設定
        intent.putExtra("SEARCH_RESULT", text)
        startActivity(intent)
    }
}
```

参照したい画面では以下のように書く
```kotlin
class SearchActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_search)

        val intent = getIntent()
        // 遷移前の画面の変数の取得
        val message = intent.extras?.getString("SEARCH_RESULT")?:""
        editTextTextPersonName.setText(message)
    }
}
```
