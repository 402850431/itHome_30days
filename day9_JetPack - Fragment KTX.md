## [Day9] Android - Kotlin筆記：JetPack - Fragment KTX


### Fragment KTX
首先要在app的`build.gradle`加入:
```kotlin
dependencies {
    implementation("androidx.fragment:fragment-ktx:1.3.6")
}
```
---

### viewModels
```kotlin
@MainThread inline fun <reified VM : ViewModel> Fragment.viewModels(
    noinline ownerProducer: () -> ViewModelStoreOwner = { this }, 
    noinline factoryProducer: () -> ViewModelProvider.Factory = null
): Lazy<VM>
```
> 這個套件可以使我們直接透過  
> `by activityViewModels()`或`by viewModels()`  
> 取用`ViewModel`。

`by activityViewModels()`：取得`fragment`所在的`activity`的`viewModel`
```kotlin
class MyFragment : Fragment() {
    val viewmodel: MyViewModel by activityViewModels()
    //或者這樣寫 
    //val viewmodel: MyViewModel by activityViewModels()
}
```
`by viewModels()`：取得`fragment`的`viewModel`
```kotlin 
class MyFragment : Fragment() {
    val viewmodel: MyViewModel by viewModels()
}
```
---

`fragment`間的頁面切換也可以從以往的：
```kotlin
supportFragmentManager.beginTransaction()
            .addToBackStack("...")
            .setCustomAnimations( R.anim.enter_anim, R.anim.exit_anim)
            .replace(
                R.id.fragment_container,
                myFragment,
                FRAGMENT_TAG
            )
            .commitAllowingStateLoss()
```
改寫為：
```kotlin
supportFragmentManager.commit {
            addToBackStack("...")
            setCustomAnimations( R.anim.enter_anim, R.anim.exit_anim)
            replace(
                R.id.fragment_container,
                myFragment,
                FRAGMENT_TAG
            )
        }     
```
---
在這邊只簡單列出幾個常用的方法，  
如果有興趣研究更多可以看看[官網](https://developer.android.com/reference/kotlin/androidx/fragment/app/package-summary)喔。