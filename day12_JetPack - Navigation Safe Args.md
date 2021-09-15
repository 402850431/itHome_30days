## [Day11] Android - Kotlin筆記：JetPack - Fragments在Navigation中的參數傳遞 - SafeArgs

### Fragments在Navigation中的參數傳遞 - SafeArgs
繼上篇我們得知如何運用`Navigation`在`Fragment`中做頁面切換。  
而今天要講的是`Safe Args`，  
他的作用是在`Fragment`中頁面切換時，傳遞參數(`Arguments`)。

---

首先要在app的`build.gradle`加入:
```kotlin
dependencies {
    implementation("androidx.navigation:navigation-runtime-ktx:2.3.5")
    implementation("androidx.navigation:navigation-fragment-ktx:2.3.5")
    implementation("androidx.navigation:navigation-ui-ktx:2.3.5")
}
```
module層的`build.gradle`
```kotlin
apply plugin: "androidx.navigation.safeargs.kotlin"
```
---

我們將昨天的`navigation.xml`打開，會看到所有頁面。
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/aFragment"
    android:id="@+id/nav_graph">
    <fragment
        android:id="@+id/aFragment"
        android:name="com.example.cashdog.cashdog.AFragment"
        android:label="@string/label_blank"
        tools:layout="@layout/fragment_blank" >
        <action
            android:id="@+id/action_aFragment_to_bFragment"
            app:destination="@id/bFragment" />
    </fragment>
    <fragment
        android:id="@+id/bFragment"
        android:name="com.example.cashdog.cashdog.BFragment"
        android:label="@string/label_blank_2"
        tools:layout="@layout/fragment_blank_fragment2" />
</navigation>
```
假設我們要從`AFragment`傳遞參至`BFragment`，
會在`BFragment`底下加入：
```kotlin
<argument
        android:name="userName"
        app:argType="string" 
        app:nullable="false"/>
```
- `android:name`: 可以視為`argument`的id
- `app:nullable`: `AFragment`頁面轉到`BFragment`時，是否為必傳值。`false`為必填，`true`可不填。
- `app:argType`: `argument`的型態。

`app:argType`能選擇的類型有：
![](https://402850431.github.io/argType.png)

加入`navigation.xml`中，會像這樣：
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/aFragment"
    android:id="@+id/nav_graph">
    <fragment
        android:id="@+id/aFragment"
        android:name="com.example.cashdog.cashdog.AFragment"
        android:label="@string/label_blank"
        tools:layout="@layout/fragment_blank" >
        <action
            android:id="@+id/action_aFragment_to_bFragment"
            app:destination="@id/bFragment" />
    </fragment>
    <fragment
        android:id="@+id/bFragment"
        android:name="com.example.cashdog.cashdog.BFragment"
        android:label="@string/label_blank_2"
        tools:layout="@layout/fragment_blank_fragment2">
        <argument
            android:name="userName"
            app:argType="string" 
            app:nullable="false"/>
    </fragment>
</navigation>
```
---
### 傳遞`args`的方式

直接將要傳遞的參數寫入actionAFragmentToBFragment(`argument`)中，
```kotlin
val action = AFragmentDirections.actionAFragmentToBFragment(userName)
```

```kotlin
class AFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        view.findViewById<Button>(R.id.next)
            .setOnClickListener {
                val action = AFragmentDirections.actionAFragmentToBFragment(userName)
                view.findNavController().navigate(action)
            }
    }
}
```

---
### 取得`args`的方式 - by navArgs


在`BFragment`中取得傳遞過來的`argument`包裹：
```kotlin
val args by navArgs<BFragmentArgs>()
```

直接獲取`argument`中的內容方式如下：
```kotlin
val userName = args.userName
```


```kotlin
class BFragment : Fragment() {

    private val args by navArgs<BFragmentArgs>()

    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val userName = args.userName
    
     }
     ...

}
```
比起以前的寫法相較快很多：
```kotlin
val userName = arguments?.getString("userName") ?: ""
```

參考：
- https://developer.android.com/kotlin/ktx#kts
- https://willy2016.pixnet.net/blog/post/216922818-android-kotlin-java-jetpack-navigation-(%E4%BA%8C)-safe-args-%E4%BB%8B