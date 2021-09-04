## [Day3] Andoroid - Kotlin筆記：高階函式與 lambda

來理解一下lambda
---


> 以下簡單介紹lambda的演進
---

我們一般寫一個function：
```kotlin
    fun sayHi() {
        //doSomething
    }
```

⬇️

在kotlin也可寫作變數一樣，這時fun可寫成匿名函數來展現：
```kotlin
    val sayHi = fun() { // <-匿名函數:沒有名字的function，像這個
        println("hello world")
    }
```

⬇️

變數的型態宣告實際上長這樣：
>`()`:沒有傳入參數，`Unit`:沒有返回值
```kotlin
    val sayHi: () -> Unit = fun() {
        println("hello world")
    }
```

⬇️

因為匿名函數沒有名稱，所以省去他會變這樣，這就是lambda：
```kotlin
    val sayHi: () -> Unit = {  println("hello world")  }
```

⬇️

沒有傳入值與返回值，因此型態宣告也可以省略，最簡寫的樣子：
```kotlin
    val sayHi = { println("hello world") }
```

---

## 帶入參數與返回值的lambda
> 理解了lambda的簡單概念後，來更進一步運用

今天我們要將值以幣種為前綴來顯示，
以前會這樣寫：
```kotlin
    fun moneyFormat ( money: Int): String {
        return "NT.$money"
    }
```

⬇️

了解lambda後：
```kotlin

    //原本變數命名方式，全部展開會像這樣
    val moneyFormat: (money: Int) -> String = fun(money: Int): String { 
        return "NT.$money"
    }
    
```

⬇️

將匿名函數省去：(簡易省去版)

>在`->`前面的變數會被編譯器認作為函數的帶入值，
>而function中的最後一行則會被當作函數的返回值。

>`moneyFormatOmitVer`更簡寫，
>在只有一個帶入參數的情況下，可以選擇不命名他，
>編譯器會將其默認命名為`it`來使用，非常省時。

```kotlin
    val moneyFormat: (money: Int) -> String = { money -> "NT.$money" }
    
    val moneyFormatOmitVer: (money: Int) -> String = { "NT.$it" }
```

⬇️

由於帶入值及返回值都可寫在fun中，
因此我們發現更簡潔的寫法：(終極簡寫版)

```kotlin
    val moneyFormat = { money: Int -> "NT.$money" }
```

---

## 帶入複數參數
> 理解了上述觀念後，玩玩看帶入兩個參數

你以前會這樣寫：
```kotlin
    fun plus (firstNum: Int, secondNum: Int): Int {
        return firstNum + secondNum
    }
```

現在可以寫更少字、更簡易：
```kotlin
    //lambda
    val plus = { firstNum: Int, secondNum: Int ->
        firstNum + secondNum
    }
```

---

## 高階函式
理解lambda後，高階函式就更易懂了。
> 定義：帶入值或返回值型別是一個函式的函式。

舉例：
```kotlin
    fun calculate(firstNum: Int, secondNum: Int, operator: (Int, Int) -> Int): String {
        return "${operator(firstNum, secondNum)}"
    }
    
    val plus = { firstNum: Int, secondNum: Int ->
        firstNum + secondNum
    }
    
    val minus = { firstNum: Int, secondNum: Int ->
        firstNum - secondNum
    }
    
```

`calculate`就是一個將函式帶入函式的高階函式。
這時當我們執行`println(calculate(1, 2, plus))`時，
會得到：`3`
執行`println(calculate(1, 2, minus))`時，
會得到：`-1`

當然我們也能合併，把calculate簡化成lambda:
```kotlin
    val calculate = { firstNum: Int, 
                             secondNum: Int, 
                             operator: (Int, Int) -> String ->
        "${operator(firstNum, secondNum)}"
    }
```

---

也能像這樣，讓他先去做某件事：
```kotlin
    val multiply = { firstNum: Int, secondNum: Int, doFirst: () -> Unit ->
        doFirst.invoke() //.invoke()可省略，用doFirst()也是可以的
        firstNum * secondNum
    }

    fun test () {
        multiply(1, 2, {
           Toast.makeText(context, "hello bro", Toast.LENGTH_SHORT).show()
        })
    }
```


參考資源：
- https://louis383.medium.com/%E5%88%9D%E6%8E%A2-kotlin-lambda-%E8%A1%A8%E9%81%94%E5%BC%8F-cfe8796c9fac
- https://ithelp.ithome.com.tw/articles/10239141