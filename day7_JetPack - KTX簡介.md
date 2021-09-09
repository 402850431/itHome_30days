## [Day7] Android - JetPack - KTX簡介

KTX是Jetpack中的一套extension，  
提供了許多簡潔、慣用的 Kotlin用法。

寫法上能夠幫你縮減function、節省程式碼、裝B用  
是個居家旅行、殺時滅code的必備良藥。 :)

---

## KTX這個減肥便當中的菜色十分多樣： 
> - Core KTX
> - Collection KTX
> - Fragment KTX
> - Lifecycle KTX
> - LiveData KTX
> - Navigation KTX
> - Palette KTX
> - Reactive Streams KTX
> - Room KTX
> - SQLite KTX
> - ViewModel KTX
> - WorkManager KTX

> - Firebase KTX
> - Google Maps Platform KTX
> - Play Core KTX

--- 

## 便當中的營養包含了：
> - 擴充函式 (Extension functions)
> - 擴充屬性 (Extension properties)
> - Lambdas 表達式
> - 參數命名(Named parameters)
> - 參數默認值(Parameter default values)
> - 協程(Coroutines)

---

接下來這幾天會介紹在公司中主要會常用到的KTX套件。  

如果你對其他套件有興趣的話，  
也可以看看[官網](https://developer.android.com/kotlin/ktx)。

---

## Core KTX

所有便當都需要有一個便當盒才能裝菜。   
正如Core KTX是所有extension的核心一般。   
因此必須先將它添加到`build.gradle`中，  
方便後續的開發運行。
```kotlin
dependencies {
    implementation "androidx.core:core-ktx:1.6.0"
}
```

---

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

- ### androidx.core.content
```kotlin
inline fun SharedPreferences.edit(
    commit: Boolean = false, 
    action: Editor.() -> Unit
): Unit
```
儲存資料至SharedPreferences的簡潔用法

用法 ex:
```kotlin
spf.edit(commit = true) {
    putString(key, value)
}
```
對 這寫法真的很舒服 😌  
註：spf的宣告
```kotlin
private val spf by lazy { 
    this.getSharedPreferences(getString(R.string.preference_file_key), Context.MODE_PRIVATE) 
}
```

---

- ### androidx.core.text
```kotlin
TextView.doAfterTextChanged(crossinline action: (text: Editable?) -> Unit)```
這是`TextWatcher`的擴充套件。  
如果你afterTextChanged下面做其他事，  
不需要再寫落落ㄉㄥˊ的
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
- ### androidx.core.view

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
直接簡化😌：
```kotlin
textView.isVisible = isLogin
```
好舒服👍

---

Core的用法先介紹到這，  
如果還有興趣，可以從[官網](https://developer.android.com/kotlin/ktx#core)中挖寶喔。
