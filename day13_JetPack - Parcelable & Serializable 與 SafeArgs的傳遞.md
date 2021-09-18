### Day13 - Parcelable & Serializable 與 SafeArgs

---

這邊簡單介紹兩者差異和選擇：

`Parcelable`：
- 效能比`Serializable`好，在記憶體開銷方面較小。
- 資料傳輸時推薦使用`Parcelable`，如`activity`、`fragment`間的資料傳遞。

`Serializable`：
- `Serializable`可將資料持久化儲存。
- 需要儲存或網路傳輸資料時選擇`Serializable`。

---

由於昨天提到`argsType`  
今天順便舉例每個型別應該如何傳遞  
尤其是`Parcelable`和`Serializable`  

![](https://402850431.github.io/argType.png)
以及`array`

---
#### Array (integer array為例)
```kotlin
<argument
  android:name="myInt"
  app:argType="integer[]"
  app:nullable="true"
  android:defaultValue="＠null" />
```
1. 可以使用 `app:nullable="true"`。   
2. 僅支持一個默認值，即`@null`。
---

#### Integer
```kotlin
<argument
  android:name="myInt"
  app:argType="integer"
  app:nullable="false"
  android:defaultValue="0" />
```
---
#### Float
```kotlin
<argument
  android:name="myInt"
  app:argType="float"
  app:nullable="false"
  android:defaultValue="0" />
```
---
#### Long
```kotlin
<argument
  android:name="myInt"
  app:argType="long"
  app:nullable="false"
  android:defaultValue="0L" />
```
---
#### Boolean
```kotlin
<argument
  android:name="myInt"
  app:argType="boolean"
  app:nullable="false"
  android:defaultValue="false" />
```
---
#### String
```kotlin
<argument
  android:name="myString"
  app:argType="string"
  app:nullable="true"
  android:defaultValue="@null" />
```
---

#### resource reference

```kotlin
//明天補上>_<
```
---
#### Enum
```kotlin
enum class MyGenderEnum {
    MALE, FEMALE
}
```

```kotlin
 <argument
    android:name="navigateFrom"
    app:argType="com.example.MyGenderEnum" 
    />                      
```
---
#### Parcelable
```kotlin
@Parcelize
data class MyData(
    val id: Int,
    val name: String,
) : Parcelable
```
```kotlin
<argument
    android:name="editBankCard"
    android:defaultValue="@null"
    app:argType="com.example.MyData"
    app:nullable="true" />        
```
---
#### Serializable
```kotlin
data class MyData(
    val id: Int,
    val name: String,
) : Serializable
```
```kotlin
<argument
    android:name="editBankCard"
    android:defaultValue="@null"
    app:argType="com.example.MyData"
    app:nullable="true" />        
```
---
