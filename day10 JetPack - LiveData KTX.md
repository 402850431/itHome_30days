## [Day10] Android - Kotlin筆記：JetPack - LiveData KTX


## LiveData KTX
首先要在app的`build.gradle`加入:
```kotlin
dependencies {
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.4.0-alpha03")
}
```

#### 這篇主要會介紹：  
- `ViewModel scope`、  
- `liveData & emit()`、  
- `switchMap`、  
- `emitSource`   
這四個功能。

---
## ViewModel scope
這是在`ViewModel`中最方便使用`coroutine`(協程)的方式之一，  
使用[viewModelScope](https://developer.android.com/topic/libraries/architecture/coroutines#viewmodelscope)來執行耗時工程時，
會隨著`ViewModel`被清除而自動取消。

> 使用方式：
```kotlin
class MyViewModel : ViewModel() {

    init {
        viewModelScope.launch {
            //做某些事 例如callApi
            callApi()
        }    
    }
}
```

---

## liveData & emit()
以往我們在更新`LiveData`時，在`ViewModel`中的做法為：
```kotlin
//沒有liveData KTX的寫法
class MyViewModel : ViewModel() {
    private val _result = MutableLiveData<String>()
    val result: LiveData<String> = _result

    init {
        viewModelScope.launch {
            val apiResult = callApi()
            _result.value = apiResult
          }
      }
  }
```

> 使用方式：
```kotlin
//KTX寫法
class MyViewModel : ViewModel() {
    val result = liveData {
        emit(callApi())
    }
}
```
這方式可以直接取得immutable(不可變的) `LiveData`，  
並透過`emit`提交改變。   
比以前的`_result.value = apiResult`或`_result.postValue(apiResult)`，  
更為簡單明瞭。  

覺得寫法上變的簡單很多> <  

---

## switchMap
當我們需要在`LiveData`發生變化後，啟動`coroutine`(協程)時，會使用`switchMap`。  
例如：當你需要在得到使用者登入得到`token`後，刷新首面的`api`。

> 使用[Transformations.switchMap](https://developer.android.com/reference/android/arch/lifecycle/Transformations#switchMap(android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20android.arch.lifecycle.LiveData%3CY%3E%3E))十分方便：
```kotlin   
    private val token = MutableLiveData<String>()
    //得到token時才執行
    val result = token.switchMap { token -> 
        liveData { emit(reloadHomePageApi(token)) }
    }
```
`result`是一個immutable `LiveData`。  
在每次`token`更新時，`result`會得到一個在執行完`reloadHomePageApi(token)`後，  
回傳的新資料。

---

## emitSource
使用時機是：當你需要先行提交初始化value，  
等成功呼叫後再回傳一次值時。  
(使用頻率較小)。  

> 使用方式：
```kotlin
liveData(Dispatchers.IO) {
    emit(LOADING_STRING)
    emitSource(dataSource.fetchWeather())
}
```

在這邊一樣只簡單列出幾個常用的方法，  
如果有興趣研究更多可以看看[官網](https://developer.android.com/topic/libraries/architecture/coroutines#livedata)的介紹喔。


參考：
- [LiveData with Coroutines and Flow — Part II: Launching coroutines with Architecture Components](https://medium.com/androiddevelopers/livedata-with-coroutines-and-flow-part-ii-launching-coroutines-with-architecture-components-337909f37ae7)
- [Youtube - LiveData with Coroutines and Flow (Android Dev Summit '19)](https://www.youtube.com/watch?v=B8ppnjGPAGE&ab_channel=AndroidDevelopers)