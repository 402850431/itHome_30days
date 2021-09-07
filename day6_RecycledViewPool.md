## [Day6] Andoroid - Kotlin筆記：RecycledViewPool

> ## 什麼是RecycledViewPool
`RecycledViewPool`是`RecycledView`的緩存池。
簡單來說，當你有很多`RecycledView`共用同個`ViewHolder`時，
就能使用`RecycledViewPool`來讓他們共用相同的`ViewHolder`。

---

> ## 使用情境

- ### 情境一：`ViewPager` + `RecyclerView`  
  
我們在做分頁時常常會搭配`ViewPager`或`ViewPager2`去做頁面間的切換。  
而頁面間切換的`RecyclerView`的layout有時大同小異，  
如果多個頁面間，都存在相同類型的ViewHolder，就可以使用`RecycledViewPool`。   
使用方法很簡單：

```kotlin
val rvPool by lazy { RecycledViewPool() }

recyclerView1.setRecycledViewPool(rvPool)
recyclerView2.setRecycledViewPool(rvPool)
recyclerView3.setRecycledViewPool(rvPool)
...
```

---

- ### 情境二：提前創建ViewHolder 
  

```kotlin
itemView.recyclerView.apply {

    //先把pool和viewholder創建出來，viewType則填入ViewHolder對應的ItemViewType
    val viewHolder = leagueOddAdapter.createViewHolder(league_odd_list, viewType);

    rvPool.putRecycledView(viewHolder);
    setRecycledViewPool(rvPool);
    layoutManager = LinearLayoutManager(this.context)
}
            
```

---

參考：
- https://blog.csdn.net/momo_ibeike/article/details/83900915