## [Day11] Android - Kotlin筆記：JetPack - Navigation KTX


## Navigation KTX
首先要在app的`build.gradle`加入:
```kotlin
dependencies {
    implementation("androidx.navigation:navigation-runtime-ktx:2.3.5")
    implementation("androidx.navigation:navigation-fragment-ktx:2.3.5")
    implementation("androidx.navigation:navigation-ui-ktx:2.3.5")
}
```
---
## by navArgs

```kotlin
class MyDestination : Fragment() {

    // 取得傳遞過來的argument bundle
    val args by navArgs<MyDestinationArgs>()

    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        view.findViewById<Button>(R.id.next)
            .setOnClickListener {
                // 找到對應的NavController並跳轉至該頁面
                findNavController().navigate(R.id.action_to_next_destination)
            }
     }
     ...

}
```


參考：
- https://developer.android.com/kotlin/ktx#kts