## [Day4] Andoroid - Kotlin筆記：RecyclerView Adapter - ListAdapter + DiffUtil

來介紹一下DiffUtil
---

> 以往我們在使用`RecyclerView`時最常使用的是  
> `RecyclerView.Adapter`及其更新方式`notifyDataSetChanged()`。  
> 此篇會介紹新的Adapter - `ListAdapter`以及新的更新方法 - `DiffUtil`  

`ListAdapter`+`DiffUtil`在動態更新上，  
效能比以往大提升，撰寫上也方便且省code。  
 
---

我們以前使用`notifyDataSetChanged()`在每次更新時，  
都會去重新綁定list中的每一筆item的`viewHolder`，  
以及重新繪製item的`view`(包含未顯示在螢幕上的item)的耗能。  
此外`notifyDataSetChanged()`也容易會造成大量螢幕更新，  
顯示上會使user看到螢幕閃爍、滾動時的彈跳。  

而為了解決這件事，會使用RecyclerView提供的工具來取代`notifyDataSetChanged()`：  
``` 
notifyItemInserted(position)
notifyItemChanged(position)
notifyItemMoved(fromPosition, toPosition)、
notifyItemRemoved(position)

notifyItemRangeInserted(position, count)
notifyItemRangeChanged(position, count)
notifyItemRangeRemoved(position, count)
notifyItemRangeChanged(position, count, payload)
```
手動找出list修改的部分，以進行新增、刪除、移動、更新那些變動的item。  

現在，我們可以使用`DiffUtil`自帶的演算法，  
搭配`ListAdapter`來幫你搞定一切更新。  

---

## DiffUtil介紹

> `DiffUtil`是為了改善RecyclerView的更新效能而生。  
> 他能自動幫你判斷新進來的資料與舊資料是否相同、移動、新增、刪除⋯⋯等等。  

--- 

### ListAdapter介紹

`ListAdapter`是繼承`RecyclerView.Adapter`的adapter：  
> - 優點1: 它包含了初始化`DiffUtil.ItemCallback`的方式在裡面  
> - 優點2: 它內建了`submitList(list)`的function，  
可以取代每次我們在`RecyclerView.Adapter`中都需要撰寫用來更新的dataList  
> - 優點3: 不需再覆寫`getItemCount()`，`ListAdapter`會幫你計算好size  

```kotlin
    //用ListAdapter就不用再每次重寫他了
    var dataList = listOf<T>()
        set(value) {
            field = value
            notifyDataSetChanged()
        }
```

---

接下來我們就來下實作範例：  

## 實作`ListAdapter`以及`DiffUtil.ItemCallback<DataClass>`  
(以`Person`為`DataClass`作範例)。  


---

### ListAdapter實作


先新增一個`Person`作為本文範例list的`DataClass`：  

```kotlin
data class Person(
    val id: Long?,
    val name: String?
)
```

改寫上我們將原本的`RecyclerView.Adapter`  
```kotlin
class RvAdapter() :
    RecyclerView.Adapter<RecyclerView.ViewHolder>() {

```
替換為  
```kotlin
class RvAdapter() : 
    ListAdapter<Person, RecyclerView.ViewHolder>(DiffCallback())

```
就可以了。  

註：
- `DiffCallback()`是`DiffUtil.ItemCallback`的class  

---

### DiffUtil實作

新增`DiffCallback`class，extends `DiffUtil.ItemCallback<DataClass>()`，  
並implement需要覆寫的方法：  

```kotlin
    class DiffCallback : DiffUtil.ItemCallback<Person>() {
        //1
        override fun areItemsTheSame(oldItem: Person, newItem: Person): Boolean {
            TODO("Not yet implemented")
        }
        //2
        override fun areContentsTheSame(oldItem: Person, newItem: Person): Boolean {
            TODO("Not yet implemented")
        }
    }
```

1.`areItemsTheSame`用來判斷item是否被編輯、移動、或刪除，  
在這邊我們使用`id`來作為判斷。  
（註：如果`id`一樣的話，他會被當成一樣的item來判斷）  
此方法能使`DiffUtil`自動幫我們判斷這筆id現在位置在哪，  
來使用最佳方法更新數據及展示相對應的更新動畫。  

2.`areContentsTheSame`是為了避免在發生更改時重新設計整個列表，  
只會更新兩個列表之間具有不同值的項目。  


```kotlin
    class DiffCallback : DiffUtil.ItemCallback<Person>() {
        override fun areItemsTheSame(oldItem: Person, newItem: Person): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Person, newItem: Person): Boolean {
            return oldItem == newItem
        }
    }

```

`areItemsTheSame`用來確認新舊item是否有相同`id`(也可以是不同的值)  
`areContentsTheSame`來確認新舊item內容是否完全相等。  

---

### 更新ListAdapter中的data


以往取值方式為：  
```kotlin

    var dataList = listOf<Person>()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
       val name = dataList.getOrNull(position)
    }
```

使用`ListAdapter`，直接用`getItem`就可取得值：  
```kotlin
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
       val name = getItem(position)
    }
```

---

### Data在ListAdapter的使用

在`ListAdapter`中我們拔除了以前拿來更新用的dataList，  
提交新list時，只要使用`submitList`，讓`DiffUtil`演算法來計算就可以了。  


```kotlin
        //activity、fragment
        viewModel.personListResult.observe(viewLifecycleOwner, { resultList ->
                rvAdapter.submitList(resultList)
        })

```


全代碼範例展示：  
```kotlin
data class Person(
    val id: Long?,
    val name: String?
)

class PersonAdapter : ListAdapter<Person, RecyclerView.ViewHolder>(DiffCallback()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return ItemViewHolder.from(parent)
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {
            is ItemViewHolder -> {
                holder.bind(getItem(position))
            }
        }
    }

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

    class DiffCallback: DiffUtil.ItemCallback<Person>() {
        override fun areItemsTheSame(oldItem: Person, newItem: Person): Boolean {
            return oldItem == newItem
        }

        override fun areContentsTheSame(oldItem: Person, newItem: Person): Boolean {
            return oldItem == newItem
        }

    }

}

```

參考資源：
- Lesson 8 of https://www.udacity.com/course/developing-android-apps-with-kotlin--ud9012
- https://www.raywenderlich.com/21954410-speed-up-your-android-recyclerview-using-diffutil