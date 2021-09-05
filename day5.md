## [Day5] Andoroid - Kotlin筆記：RecyclerView Adapter - ListAdapter + DiffUtil 進階應用

> ## Problem
昨天我們提到`ListAdapter + DiffUtil`在一般`RecyclerView`的基本使用。  
而實際上工作中我們經常會需要在`RecyclerView`上顯示不同的`itemView`，  
我們會添加`Footer`、`Header`，或是不同樣式的`itemView`，  
這時候該怎麼辦呢？  

> ## Solution
我們在`submitList`之前，在`dataList`中將`item`的順序重新編排後，  
再使用`submitList`提交我們的`dataList`。  

> ## 實作
> 簡單例子假設：在`RecyclerView`的最後一項，  
> 要添加一個名為“已滑至底”的`TextView`，來做為`Footer`

資料的部分會繼續沿用`Person`這個`data class`做舉例
```kotlin
data class Person(
    val id: Long?,
    val name: String?
)
```
---
### 1.
首先，我們先添加一個`sealed class`，這個`sealed class`負責用來控管所有的item類別。  
並列舉出`Item`和`Footer`這兩個型態(data type)。
```kotlin
sealed class DataItem {
    data class Item(val person: Person) : DataItem() {
    }

    object Footer : DataItem() {
    }
}
```

(關於`sealed class`後續文章有機會會講解，或是你也可以看[這篇](https://betterprogramming.pub/how-to-use-kotlin-sealed-classes-for-state-management-c1cfb81abc6a))

這邊因為`Footer`只是作為靜態顯示layout  
因此只使用`object`而非`data class`。  

---

### 2.
而因為我們的`DiffUtil`需要一個參數，  
作為判斷新舊item是否一樣的依據。
因此我們創建一個abstract item `id`作為interface回傳判別用的數據。
```kotlin
sealed class DataItem {

    abstract val id: String

    data class Item(val person: Person) : DataItem() {
        override val id = person.id
    }

    object Footer : DataItem() {
        override val id: String = ""
    }

}
```

---

### 3.
為`Item`及`Footer`創建對應的`ViewHolder`
```kotlin
    //item
    class ItemViewHolder private constructor(val view: View) : RecyclerView.ViewHolder(view) {
        fun bind(item: Person) {
            itemView.tv_play_name.text = item.name
        }

        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val binding = LayoutInflater.from(parent.context)
                    .inflate(R.layout.itemview_person, parent, false)
                return ItemViewHolder(binding)
            }
        }
    }

    //footer
    class FooterViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        companion object {
            fun from(parent: ViewGroup) =
                FooterViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.item_footer, parent, false))
        }
    }
```

--- 

### 4.

