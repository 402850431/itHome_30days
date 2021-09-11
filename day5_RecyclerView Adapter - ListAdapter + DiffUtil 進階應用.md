## [Day5] Android - Kotlin筆記：ListAdapter + DiffUtil 進階應用 - 複數itemViewType

> ## Problem
昨天我們提到`ListAdapter + DiffUtil`在一般`RecyclerView`的基本使用。  
而實際上工作中我們經常會需要在`RecyclerView`上顯示不同的`itemView`，  
我們會添加`Footer`、`Header`，或是不同樣式的`itemView`，  
這時候該怎麼辦呢？  

---

> ## Solution
在`submitList`之前，把`dataList`中的`item`順序重新編排後，  
再使用`submitList`提交我們的`dataList`做更新。  

---

> ## 實作
> 假設我們要做一個像Line一樣的聊天室，  
> 而且底部要添加一個名為“已滑至底”的`TextView`做為`Footer`。

資料的部分會用`Message`這個`data class`來顯示訊息，
並根據`isFromMe`來判斷是否為自己發送的訊息，展示不同的`itemView
```kotlin
data class Message(
    val id: Long,
    val timeStamp: Long,
    val isFromMe: Boolean,
    val message: String
)
```
---

### 1. ListAdapter

首先，因為會展示不同的項目，我們不餵`ListAdapter`吃`data class`了，  
改吃自己創建的`sealed class` - 這邊我們命名為`DataItem`。
```kotlin
class MessageAdapter() : ListAdapter<DataItem, RecyclerView.ViewHolder>(DiffCallback()) {}
```

---

### 2. 類別控管
我們添加一個`sealed class`，  
這個`sealed class`負責用來控管不同item的型態類別。  
這邊我們列舉出`Item`和`Footer`這兩個型態(data type)。

因為`DiffUtil`需要一個參數作為判斷新舊item是否一樣的依據。  
所以我們創建一個abstract item `id`作為interface回傳判別用的數據。  

`isFromMe`則是用來判斷顯示訊息在左側還是右側的`item view`。  

```kotlin
sealed class DataItem {

    abstract val id: Long
    abstract val isFromMe: Boolean

    data class Item(val message: Message) : DataItem() {
        override val id = message.id
        override val isFromMe = message.isFromMe
    }

    object Footer : DataItem() {
        override val id = Long.MIN_VALUE
        override val isFromMe = false
    }
}
```

(關於`sealed class`後續文章有機會會講解，或是你也可以看[這篇](https://betterprogramming.pub/how-to-use-kotlin-sealed-classes-for-state-management-c1cfb81abc6a)寫得很詳盡)

這邊因為`Footer`只是作為靜態顯示layout  
因此只使用`object`而非`data class`。  

---

### 3. DiffUtil

DiffUtil也改為判斷`DataItem`中的`id`
```kotlin
    class DiffCallback : DiffUtil.ItemCallback<DataItem>() {
        override fun areItemsTheSame(oldItem: DataItem, newItem: DataItem): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: DataItem, newItem: DataItem): Boolean {
            return oldItem == newItem
        }

    }
```

---

### 4. ItemViewType

創建一個enum class列舉需展示的所有`viewType`，  
```kotlin
    enum class ItemViewType {
        MESSAGE_FROM_ME, MESSAGE_TO_ME, FOOTER
    }
```
這邊列舉`viewType`，是給Adapter判斷要展示哪個`ViewHolder`來使用的。

---
### 5. getItemViewType
因為我們有不同的型別需判斷，  
所以我們要覆寫`getItemViewType`。  
鍵盤按下`control`+`o`，選擇`getItemViewType`並覆寫他。  

```kotlin
    override fun getItemViewType(position: Int): Int {
        return when (getItem(position)) {
            is DataItem.FromMe -> ItemViewType.MESSAGE_FROM_ME.ordinal
            is DataItem.ToMe -> ItemViewType.MESSAGE_TO_ME.ordinal
            is DataItem.Footer -> ItemViewType.FOOTER.ordinal
        }
    }
```
這邊返回的int是`onCreateViewHolder`會用到的`viewType`，繼續往下看下去。

---

### 6. 新增`onCreateViewHolder`與`onBindViewHolder`判斷
```kotlin
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            ItemViewType.MESSAGE_FROM_ME.ordinal -> FromMeViewHolder.from(parent)
            ItemViewType.MESSAGE_TO_ME.ordinal -> ToMeViewHolder.from(parent)
            else -> FooterViewHolder.from(parent)
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {

            is FromMeViewHolder -> {
                val data = getItem(position) as DataItem.Item
                holder.bind(data.message)
            }

            is ToMeViewHolder -> {
                val data = getItem(position) as DataItem.Item
                holder.bind(data.message)
            }

            is FooterViewHolder -> {
            }
        }
    }
```

---

### 7. 新增對應的`ViewHolder`

為`FromMe`、`ToMe`及`Footer`創建對應的`ViewHolder`
```kotlin
    class FromMeViewHolder private constructor(val binding: ItemAccountHistoryNextContentBinding) :
        RecyclerView.ViewHolder(binding.root) {

        fun bind(message: Message) {
            itemView.apply {
                tv_message_from_me.text = message.content
            }
        }

        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_message_from_me, parent, false)
                return ToMeViewHolder(view)
            }
        }


    }

    class ToMeViewHolder (view: View) : RecyclerView.ViewHolder(view) {

        fun bind(message: Message) {
            itemView.apply {
                tv_message_to_me.text = message.content
            }
        }

        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_message_to_me, parent, false)
                return ToMeViewHolder(view)
            }
        }

    }

    class FooterViewHolder (view: View) : RecyclerView.ViewHolder(view) {
        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_footer, parent, false)
                return FooterViewHolder(view)
            }
        }
    }
```

--- 

### 8. submitList

最後一步了！  
現在因為`ListAdapter`吃的是DataItem，  
我們只要創建一個function - `addFooterAndSubmitList`來取代原本的`submitList`，  
利用`DataItem`整理過後再交給`submitList`處理就大功告成。  

```kotlin
    private val adapterScope = CoroutineScope(Dispatchers.Default)

    fun addFooterAndSubmitList(list: List<Message>) {
        adapterScope.launch {
            val items = list.map { 
                if (it.isFromMe) DataItem.FromMe(it)
                else DataItem.ToMe(it)
            } + listOf(DataItem.Footer)
            withContext(Dispatchers.Main) { //update in main ui thread
                submitList(items)
            }
        }
    }
```
只要透過呼叫`addFooterAndSubmitList`就能讓`ListAdapter`成功運作，
讓`DiffUtil`自動去篩選判斷更新的內容。

```kotlin
    rvAdapter.addFooterAndSubmitList(dataList)
```
---

## 全程式碼展示：

- ### Adapter

```kotlin
data class Message(
    val id: Long,
    val timeStamp: Long,
    val isFromMe: Boolean,
    val content: String
)

class MessageAdapter() : ListAdapter<DataItem, RecyclerView.ViewHolder>(DiffCallback()) {

    enum class ItemViewType {
        MESSAGE_FROM_ME, MESSAGE_TO_ME, FOOTER
    }

    private val adapterScope = CoroutineScope(Dispatchers.Default)

    fun addFooterAndSubmitList(list: List<Message>) {
        adapterScope.launch {
            val items = list.map {
                if (it.isFromMe) DataItem.FromMe(it)
                else DataItem.ToMe(it)
            } + listOf(DataItem.Footer)
            withContext(Dispatchers.Main) { //update in main ui thread
                submitList(items)
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            ItemViewType.MESSAGE_FROM_ME.ordinal -> FromMeViewHolder.from(parent)
            ItemViewType.MESSAGE_TO_ME.ordinal -> ToMeViewHolder.from(parent)
            else -> FooterViewHolder.from(parent)
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {

            is FromMeViewHolder -> {
                val data = getItem(position) as DataItem.FromMe
                holder.bind(data.message)
            }

            is ToMeViewHolder -> {
                val data = getItem(position) as DataItem.ToMe
                holder.bind(data.message)
            }

            is FooterViewHolder -> {
            }
        }
    }

    override fun getItemViewType(position: Int): Int {
        return when (getItem(position)) {
            is DataItem.FromMe -> ItemViewType.MESSAGE_FROM_ME.ordinal
            is DataItem.ToMe -> ItemViewType.MESSAGE_TO_ME.ordinal
            is DataItem.Footer -> ItemViewType.FOOTER.ordinal
        }
    }


    class FromMeViewHolder private constructor(val binding: ItemAccountHistoryNextContentBinding) :
        RecyclerView.ViewHolder(binding.root) {

        fun bind(message: Message) {
            itemView.apply {
                tv_message_from_me.text = message.content
            }
        }

        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_message_from_me, parent, false)
                return ToMeViewHolder(view)
            }
        }


    }

    class ToMeViewHolder (view: View) : RecyclerView.ViewHolder(view) {

        fun bind(message: Message) {
            itemView.apply {
                tv_message_to_me.text = message.content
            }
        }

        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_message_to_me, parent, false)
                return ToMeViewHolder(view)
            }
        }

    }

    class FooterViewHolder (view: View) : RecyclerView.ViewHolder(view) {
        companion object {
            fun from(parent: ViewGroup): RecyclerView.ViewHolder {
                val view = LayoutInflater.from(parent.context).inflate(R.layout.itemview_footer, parent, false)
                return FooterViewHolder(view)
            }
        }
    }

    class DiffCallback : DiffUtil.ItemCallback<DataItem>() {
        override fun areItemsTheSame(oldItem: DataItem, newItem: DataItem): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: DataItem, newItem: DataItem): Boolean {
            return oldItem == newItem
        }

    }
}

sealed class DataItem {

    abstract val id: Long
    abstract val isFromMe: Boolean

    data class FromMe(val message: Message) : DataItem() {
        override val id = message.id
        override val isFromMe = message.isFromMe
    }

    data class ToMe(val message: Message) : DataItem() {
        override val id = message.id
        override val isFromMe = message.isFromMe
    }

    object Footer : DataItem() {
        override val id = Long.MIN_VALUE
        override val isFromMe = false
    }

}
```
