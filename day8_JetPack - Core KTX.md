## [Day8] Android - Kotlin筆記：JetPack - Core KTX


### Core KTX 包含的module有：

> androidx.core.animation  
> androidx.core.content  
> androidx.core.content.res  
> androidx.core.database  
> androidx.core.database.sqlite  
> androidx.core.graphics  
> androidx.core.graphics.drawable  
> androidx.core.location  
> androidx.core.net  
> androidx.core.os  
> androidx.core.text   
> androidx.core.transition  
> androidx.core.util  
> androidx.core.view  
> androidx.core.widget  

這些要一個一個說的話，大概30天都說不完。  
所以舉幾個看過比較有趣的。   
剩下的大家可以自己試試，細細品味 :)   

---

### Toast

```kotlin
//原本寫法
Toast.makeText(this,
    R.string.text,
    Toast.LENGTH_SHORT).show()
```

```
//使用core KTX
context.toast(R.string.text)
```
---
### SharedPreferences
```kotlin
inline fun SharedPreferences.edit(
    commit: Boolean = false, 
    action: Editor.() -> Unit
): Unit
```
儲存資料至的簡潔用法

用法 ex:
```kotlin
spf.edit(commit = true) {
    putString(key, value)
}
```
對 這寫法真的很舒服 (*¯︶¯*)  
使用前記得先宣告spf：
```kotlin
private val spf by lazy { 
    this.getSharedPreferences(getString(R.string.preference_file_key), Context.MODE_PRIVATE) 
}
```

---
### TextWatcher
```kotlin
TextView.doAfterTextChanged(crossinline action: (text: Editable?) -> Unit)
```

只有需要afterTextChanged底下做其他事，  
不需要再寫落落ㄉㄥˊ的：
```kotlin
        textView.addTextChangedListener(object : TextWatcher {
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
            }

            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
            }

            override fun afterTextChanged(p0: Editable?) {
                //do something
            }

        })
```
直接簡化：
```kotlin
textView.doAfterTextChanged {
    //do something
}
```
---
### isVisible
```kotlin
inline var View.isVisible: Boolean
```
以前：
```kotlin
textView.visibility = if (isLogin) {
        View.VISIBLE
    } else {
        View.GONE
    }
```
直接簡化：
```kotlin
textView.isVisible = isLogin
```
好舒服(*¯︶¯*)

---

Core的用法先介紹到這，  
如果還有興趣，可以從[官網](https://developer.android.com/kotlin/ktx#core)中挖寶喔。
