
MVVM由三項組成。  
分別為(Model、View、ViewModel)  

先來上MVVM架構圖，方便下文介紹  
文末會附上簡單例子講解。  
![](https://402850431.github.io/mvvm.png)

---

> ## View

- `View`就是使用者看到的頁面，  
`xml`以及我們負責顯示`xml`數據的`Activity`和`Fragment`都屬於`View`層。

---

> ## ViewModel

- `ViewModel`負責接收`View`的需求，  
  撰寫邏輯與處理耗時工程，  
  並向`Model`讀出或寫入資料。   

- `ViewModel`的生命周期：  
使用一個空的`Fragment`來進行生命周期管理。    
生命週期會與`View`共存，但和`View`互不影響。   
(更準確地來說，`Activity`在前台被銷毀時，得到的是同個`ViewModel`，  
`Activity`在後台被銷毀時，就會是重建的`ViewModel`)  
詳細介紹可以看[這裡](https://www.jianshu.com/p/47224fd1088e)
   
      
> ### LiveData
- 簡單來說可以把它想成一個live(活著)的data。  
`LiveData`能夠動態偵測到`Data`的改變。

- 因為`ViewModel`是處理邏輯的地方，  
我們會把`LiveData`寫在`ViewModel`中。   

- 在`ViewModel`跑完邏輯後，會將結果寫入`LiveData`。  
這時在`View`層，能夠透過`LiveData`的`observe`，  
來取得更改後的資料，更新UI。

---


> ### 簡單舉例：  
> 在登入頁面按下登入按鈕回到首頁，首頁要能判別狀態為已登入，並展示使用者名稱。

資料以`LoginResult`這個`Data Class`做範例：
```kotlin
@JsonClass(generateAdapter = true)
data class LoginResult(

    @Json(name = "token")
    val token: Long,

    @Json(name = "userName")
    val userName: String? = null,

}
```


流程是這樣走的：  

1. `View` 層 (activity or fragment)

> 點擊事件觸發login的API，  
> call API這行為跟介面無關，是耗時流程，  
> 因此會寫在`ViewModel`。
```kotlin
  fun initOnClick() {

      btn_login.setOnClickListener {
          viewModel.login()
      }

  }
```

2. `ViewModel` 層

>
```kotlin
    val loginResult: LiveData<LoginResult>
        get() = _loginResult

    fun login(account: String, password: String) {

        //送出post request的api參數
        val loginRequest = LoginRequest(
            account = account,
            password = password,
        )
        
        viewModelScope.launch {
            doNetwork(androidContext) {
              //這邊做api邏輯，根據專案用volley、retrofit、okhttp，寫法會不同
              //後面章節會再帶到call api的部分。
                callLoginApi(loginRequest)
            }?.let { result ->

                //使用`postValue`來更新LiveData
                _loginResult.postValue(result)
            }
        })
    }
```
1. `View` 層 (activity or fragment)
> 同時在`View`層，透過觀測`LiveData`，  
> 我們會接收已經拿到資料的`loginResult`，  
> 在`View`層展示使用者的名字。  
> (`View`只做和UI有關的事)
```kotlin
  fun initObserver() {
        //activity用this, fragment用viewLifecycleOwner
        viewModel.loginResult.observe(this, { result ->
            tv_name.text = result.name
        })

  }
```
  註：
- `viewLifecycleOwner`是和`onCreateView()、onDestroyView()`綁在一起。
- `this`則是與`onCreate()、onDestroy()`整個生命週期相關，會存活更久一些。

---

### 下篇會再繼續介紹並加入 `Model` 的部分。