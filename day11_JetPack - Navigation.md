## [Day11] Android - Kotlin筆記：JetPack - Navigation 


## Navigation 
`Navigation`是Jetpack中的頁面轉換元件。
用來取代`FragmentTransaction`，  
以GUI方式呈現頁面流程，  
能更方便的管理各`Fragment`之間的跳轉關係。  
如圖：  
![](https://developer.android.com/images/topic/libraries/architecture/navigation-graph_2x-callouts.png)

---

### 要做到有簡單幾個步驟：  
假設`A Fragment`跳轉到`B Fragment`

1. `Gradle`新增`dependencies`
2. 新增`navigation.xml`
3. 在`Activity`的`xml`中，新增一個`<FragmentContainerView/>`元件，  
並綁定`navigation.xml`。
4. 從`A Fragment`找到內部已透過2綁定對應的`NavController`，執行頁面跳轉。

接著就來實做看看吧

---

### 1. `Gradle`新增`dependencies`

project層的`build.gradle`
```kotlin
dependencies {
  val nav_version = "2.3.5"

classpath 
"androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"

  // Java language implementation
  implementation("androidx.navigation:navigation-fragment:$nav_version")
  implementation("androidx.navigation:navigation-ui:$nav_version")

  // Kotlin
  implementation("androidx.navigation:navigation-fragment-ktx:$nav_version")
  implementation("androidx.navigation:navigation-ui-ktx:$nav_version")

  // Feature module Support
  implementation("androidx.navigation:navigation-dynamic-features-fragment:$nav_version")

  // Testing Navigation
  androidTestImplementation("androidx.navigation:navigation-testing:$nav_version")

  // Jetpack Compose Integration
  implementation("androidx.navigation:navigation-compose:2.4.0-alpha08")
}
```

module層的`build.gradle`
```kotlin
apply plugin: "androidx.navigation.safeargs.kotlin"
```
---
### 2. 新增`navigation.xml`

1. 在`res`按下右鍵，新增`Android resource file`。
2. `Resource Type`選擇`Navigation`。
![](https://i.postimg.cc/rpqRjccg/2021-09-14-9-43-07.png)

3. 在`xml`中新增對應的`fragment`:
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
`<action>`的作用為`A Fragment`跳轉至`B Fragment`。

---

### 3. 在要放入`A Fragment`和`B Fragment`的`Activity`底下，新增一個`<FragmentContainerView/>`元件，並綁定`navigation.xml`。
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">


    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

</FrameLayout>
```
 - `android:name="androidx.navigation.fragment.NavHostFragment"`  
用來宣告實現此 `Fragment` 為 `NavHostFragment`
 
- `app:defaultNavHost="true"`
`true`會攔截返回按鍵，讓頁面返回上一層，  
不會退出`Activity`。  
上一層可以是`Fragment`或`Activity`。

- `app:navGraph="@navigation/nav_graph"`
作用是將`activity_main.xml`中的`Fragment`
與`navigation`連接起來。

---

### 4. 從`A Fragment`找到內部已透過2綁定對應的`NavController`，執行頁面跳轉。

`NavController`的方法如下：
> - Activity.findNavController(viewId: Int)
> - Fragment.findNavController()
> - View.findNavController()

寫法：
```kotlin
val navHostFragment =
        supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
val navController = navHostFragment.navController
```
`navController`為`fragment`的`navigation`。  

假設透過按下`AFragment`中的某個按鈕跳轉到`BFragment`：
```kotlin
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        view.findViewById<Button>(R.id.next)
            .setOnClickListener {
                val action = AFragmentDirections.actionAFragmentToBFragment()
                view.findNavController().navigate(action)
            }
    }
```
就會切換到下一頁了。

參考：
- https://medium.com/@wsrew2000/android-navigation-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-%E4%B8%80-%E5%9F%BA%E7%A4%8E%E4%BD%BF%E7%94%A8-3c1607ce4d38
- https://willy2016.pixnet.net/blog/post/216922818-android-kotlin-java-jetpack-navigation-%28%E4%BA%8C%29-safe-args-%E4%BB%8B