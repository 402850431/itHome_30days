##  [Day2] Andoroid - Kotlin筆記： lazy

來介紹一下lazy
---


> 以下以`TextView`為例

---

我們一般`init`(初始化一個元件)：
```kotlin
    private var textView: TextView? = null
```
此方式在後續使用到```textView```的程式碼都要加上```?```才能使用

/

或是使用`lateinit`的話(參數延遲初始化)：
```kotlin
    private lateinit var textView: TextView
```
假設任何情況下，在後面程式碼沒有將`textView`成功初始化的話，
就會crash報錯：`lateinit property tv has not been initialized`

---

## lazy
> 使用時機：宣告變數
> 第一次使用時才會執行內部程式碼，
> 且只有在會用到他時才會產生該值。


使用`lazy`，可以直接在內部宣告其初始狀態。
```kotlin
    private val tv: TextView by lazy {
        findViewById(R.id.textView)
    }
```

```kotlin
class MainActivity : AppCompatActivity() {

    private val tv: TextView by lazy {
        findViewById(R.id.textView)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tv.text = "hello world"
    }
}
```

lazy只會在第一次使用時會被呼叫執行，
所以假設註解了
```kotlin
//    tv.text = "hello world"
```
則`tv`這個值就不會被產生

---

> 比起`init`、`lateinit`，
`lazy`更為方便且安全，
也能更有效地節省空間。

目前宣告各式各樣的變數，基本上都用lazy，
覺得很方便所以分享一下。

第一次寫文章，各方面都有些彆扭，不好意思了。


參考資源：
- https://blog.mindorks.com/learn-kotlin-lateinit-vs-lazy
- https://carterchen247.medium.com/kotlin%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97-%E5%8D%81%E4%B8%80-lateinit-vs-lazy-1ef96bc5b3b3