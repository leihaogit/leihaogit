---
title: '使用 BRVAH 更简单的操作 RecyclerView'
date: '2023-09-26'
description: '介绍第三方开源库 - BaseRecyclerViewAdapterHelper，实现更方便的 Recyclerview 操作。'
cover: 'https://z1.ax1x.com/2023/09/25/pP7Y8rn.png'
categories:

- 编程开发

tags:

- Kotlin
- Android

--- 

# 一、概述

## 1.1 BRVAH

- BRVAH 也就是 [BaseRecyclerViewAdapterHelper](https://github.com/CymChad/BaseRecyclerViewAdapterHelper) 的缩写，是目前 GitHub 上拥有 23.8k stars 的一个很受欢迎的开源项目，旨在精简我们对 Recyclerview 的操作。
- Recyclerview 作为安卓开发者最常用的控件之一，功能十分强大，可以显示动态的列表、网格以及瀑布流数据。但是相应的使用难度也就有所上升，它不能像普通控件一样拖到界面上就可以使用了，还需要专门设置数据适配器、布局管理器等等。

## 1.2 BaseQuickAdapter

- 在日常开发中我们常用的 Recyclerview 适配器有两种：`Recyclerview.Adapter<VH>` 和 `ListAdapter<T,VH>`，后者扩展性更强一些，一般简单的需求使用前者，略微复杂的需求一般会使用后者。
- 不过，虽然原生的适配器能实现大部分功能，但是像 item 的点击事件、适配器动画、头部和尾部的添加、item 的拖拽等等需要我们自己花费不少的精力去实现，原生并未提供相应的 API 支持。
- `BaseQuickAdapter<T,VH>` 是 BRVAH 中非常重要的一个适配器，将上面说的大部分功能都已经做好了，我们只需要使用对应的 API 进行非常简单的操作即可实现强大的扩展功能。

# 二、简单使用

## 2.1 导入依赖

- 这里只介绍 4.0+ 版本的使用，遵循 Kotlin First 原则，和之前的版本肯定会有很多不同，之前版本的学习可以去 [BRVAH官网](http://www.recyclerview.org/) 查看详细文档。
- 在 build.gradle(app) 下添加依赖：

```groovy
dependencies {
    implementation 'io.github.cymchad:BaseRecyclerViewAdapterHelper:4.0.1'
}
```

## 2.2 创建实体类

- 我们这里实现一个图片 + 文字的简单列表条目的数据呈现，创建一个实体类 Emoji：

```kotlin
data class Emoji(
    //名称
    val name: String,
    //描述
    val desc: String,
    //资源id
    val resId: Int
)
```

## 2.3 创建布局和适配器

- 创建一个 item 的布局，包含一张图片和两个文字，很简单就不给出了。
- 创建一个继承自`BaseQuickAdapter<T,VH>`的适配器，需要实现其必要方法 onBindViewHolder 和 onCreateViewHolder：

```kotlin
class EmojiAdapter : BaseQuickAdapter<Emoji, EmojiAdapter.VH>() {

    override fun onBindViewHolder(holder: VH, position: Int, item: Emoji?) {
        val binding = holder.binding as CellEmojiItemBinding
        item?.let {
            binding.imageViewIcon.setImageResource(item.resId)
            binding.textViewDesc.text = item.desc
            binding.textViewName.text = item.name
        }
    }

    override fun onCreateViewHolder(context: Context, parent: ViewGroup, viewType: Int): VH {
        //根据ViewType返回不同的 binding，这里只有一个
        val binding =
            CellEmojiItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return VH(binding)
    }

    // 自定义ViewHolder类
    class VH(
        val binding: ViewBinding,
    ) : RecyclerView.ViewHolder(binding.root)

}
```

- 这里我们自己写了一个 ViewHolder，其实不写也是可以的，如果你的Adapter特别简单，不想用ViewBinding，可以使用QuickViewHolder：

```kotlin
class TestAdapter : BaseQuickAdapter<Status, QuickViewHolder>() {

    override fun onCreateViewHolder(context: Context, parent: ViewGroup, viewType: Int): QuickViewHolder {
        // 返回一个 ViewHolder
        return QuickViewHolder(R.layout.layout_animation, parent)
    }

    override fun onBindViewHolder(holder: QuickViewHolder, position: Int, item: Status?) {
        // 设置item数据
        holder.getView(R.id.xxxx)
    }

}
```

## 2.4 设置适配器

- 最后在 Activity 或者 Fragment 中，将 Recyclerview 的适配器和布局管理器设置好即可：

```kotlin
        val adapter = EmojiAdapter()
val emojiList = listOf(
    Emoji("愤怒", "表示生气，被惹恼", R.mipmap.emoji_84),
    Emoji("机器人", "一个呆头呆脑的机器人", R.mipmap.emoji_81),
    Emoji("呕吐", "我吐了", R.mipmap.emoji_89),
    Emoji("小丑", "而你，我的朋友，你才是真正的小丑", R.mipmap.emoji_93),
    Emoji("发呆", "呆萌可爱", R.mipmap.emoji_66),
    Emoji("吃惊", "太黑人了", R.mipmap.emoji_76),
    Emoji("愤怒", "表示生气，被惹恼", R.mipmap.emoji_84),
    Emoji("机器人", "一个呆头呆脑的机器人", R.mipmap.emoji_81),
    Emoji("呕吐", "我吐了", R.mipmap.emoji_89),
    Emoji("小丑", "而你，我的朋友，你才是真正的小丑", R.mipmap.emoji_93),
    Emoji("发呆", "呆萌可爱", R.mipmap.emoji_66),
    Emoji("吃惊", "太黑人了", R.mipmap.emoji_76)
)
adapter.submitList(emojiList)
binding.recyclerView.adapter = adapter
binding.recyclerView.layoutManager = LinearLayoutManager(this)
```

- 效果也就出来了：

<img src="https://z1.ax1x.com/2023/09/26/pP7zoXq.jpg" width="20%">

# 三、BaseQuickAdapter

- 看上面适配器的代码可以发现，代码量几乎是没什么变化的，那这个`BaseQuickAdapter`好在哪里呢？
- 首先我们能感觉到 BaseQuickAdapter<T,VH> 好像是个加强版的 ListAdapter<T,VH>，因为他也是通过 submitList 来提交数据，只是少了一个 DiffUtil.ItemCallback<T>？。
- 其实不是的，我们看源码能发现，BaseQuickAdapter 其实是继承自 RecyclerView.Adapter 的，submitList 只是其实现的一个扩展方法而已，如果我们要实现自定义的比较逻辑来提高性能，需要用另一个适配器：[BaseDifferAdapter](https://github.com/CymChad/BaseRecyclerViewAdapterHelper/wiki/BaseDifferAdapter)
- 参考源码：

```kotlin
abstract class BaseQuickAdapter<T, VH : RecyclerView.ViewHolder>(
    open var items: List<T> = emptyList()
) : RecyclerView.Adapter<RecyclerView.ViewHolder>()

/* ...   */
open fun submitList(list: List<T>?) {
    val newList = list ?: emptyList()
    if (list === items) return

    mLastPosition = -1

    val oldDisplayEmptyLayout = displayEmptyView()
    val newDisplayEmptyLayout = displayEmptyView(newList)

    if (oldDisplayEmptyLayout && !newDisplayEmptyLayout) {
        items = newList
        notifyItemRemoved(0)
        notifyItemRangeInserted(0, newList.size)
    } else if (newDisplayEmptyLayout && !oldDisplayEmptyLayout) {
        notifyItemRangeRemoved(0, items.size)
        items = newList
        notifyItemInserted(0)
    } else if (oldDisplayEmptyLayout && newDisplayEmptyLayout) {
        items = newList
        notifyItemChanged(0, EMPTY_PAYLOAD)
    } else {
        items = newList
        notifyDataSetChanged()
    }
}
```

- 除了上面提到的两个适配器，BRVAH 还提供了很多个其他的适配器应对不同的场景，但是说得越多越乱，复杂的功能就先不看，我们先来体验一下`BaseQuickAdapter`为我们提供了哪些方面的扩展功能。

## 3.1 空视图

- v4 版本官网中对于 BaseQuickAdapter 是这么描述的：

<img src="https://z1.ax1x.com/2023/09/26/pPHS55D.png" style="border: #cacaca solid">

- 也就也是说，目前 v4 版本的 BaseQuickAdapter 作为最基本且纯粹的适配器，只提供空布局、数据操作、点击事件三个功能了。
- 我们先来体验一下"空视图"功能，非常简单：
- 首先，是否使用空布局（默认false），需要将其打开：

```kotlin
adapter.isEmptyViewEnable = true
```

- 然后有两种方式设置空布局：

1. 使用Layout Id：

```kotlin
adapter.setEmptyViewLayout(context, R.layout.loading_view)
```

2. 使用 View

```kotlin
mAdapter.emptyView = view
```

## 3.2 数据操作

- 设置数据集合

```kotlin
adapter.submitList(list)
```

- 修改某一位置的数据

```kotlin
//修改index为1处的数据
adapter[1] = data
```

- 新增数据

```kotlin
// 尾部新增数据
adapter.add(data)

// 在指定位置添加一条新数据
adapter.add(1, data)

// 添加数据集
adapter.addAll(list)

// 指定位置添加数据集
adapter.addAll(1, list)
```

- 删除数据

```kotlin
// 删除数据
adapter.remove(data)

// 删除指定位置数据
adapter.removeAt(1)
```

- 交换数据位置（仅仅是这两个数据的位置交换）

```kotlin
// 交换两个位置的数据，只有1和3会变化
adapter.swap(1, 3)
```

- 移动数据位置（注意和 swap 的区别）

```kotlin
// 交换两个位置的数据，1跑到3的位置，原来的2和3都会往前移一位，3后面的不会发生变化，这个必须使用可变的集合
adapter.move(1, 3)
```

- 获取Item数据的索引

```kotlin
// 如果返回 -1，表示不存在
adapter.itemIndexOfFirst(data)
```

- 根据索引，获取Item数据

```kotlin
// 如果返回 null，表示没有数据
adapter.getItem(1)
```

## 3.3 动画

- 动画是指 item 出现或者消失时候的动画效果，内部内置了5种默认动画

```kotlin
/**
* BaseQuickAdapter.AnimationType.AlphaIn
* BaseQuickAdapter.AnimationType.ScaleIn
* BaseQuickAdapter.AnimationType.SlideInBottom
* BaseQuickAdapter.AnimationType.SlideInLeft
* BaseQuickAdapter.AnimationType.SlideInRight
*/
adapter.setItemAnimation(BaseQuickAdapter.AnimationType.AlphaIn)
```

- 自定义动画 需继承 ItemAnimator 实现自定义动画

```kotlin
class CustomAnimation1 : ItemAnimator {
    override fun animator(view: View): Animator {
        // 创建三个动画
        val alpha: Animator = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f)
        val scaleY: Animator = ObjectAnimator.ofFloat(view, "scaleY", 1.3f, 1f)
        val scaleX: Animator = ObjectAnimator.ofFloat(view, "scaleX", 1.3f, 1f)
        
        scaleY.interpolator = DecelerateInterpolator()
        scaleX.interpolator = DecelerateInterpolator()

        // 多个动画组合，可以使用 AnimatorSet 包装
        val animatorSet = AnimatorSet()
        animatorSet.duration = 350
        animatorSet.play(alpha).with(scaleX).with(scaleY)
        return animatorSet
    }
}

// 设置动画
adapter.itemAnimation = CustomAnimation1()
```

- 重写动画执行操作

```kotlin
class TestAdapter : BaseQuickAdapter<Status, QuickViewHolder>() {

    ...

    override fun startItemAnimator(anim: Animator, holder: RecyclerView.ViewHolder) {
        
    }
}
```

- 是否打开动画

```kotlin
adapter.animationEnable = true
```

## 3.4 点击事件

- 使用 BaseQuickAdapter 后，我们不再需要自己写接口实现 item 的各种触摸事件

- item 点击事件

```kotlin
adapter.setOnItemClickListener { adapter, view, position ->
    Tips.show("onItemClick $position")
}
```

- item 长按事件

```kotlin
adapter.setOnItemClickListener { adapter, view, position ->
    Tips.show("onItemClick $position")
}
```

- item 长按事件

```kotlin
adapter.setOnItemLongClickListener { adapter, view, position ->
    Tips.show("onItemLongClick $position")
    true
}
```

- item 子控件点击事件

```kotlin
// 需要传递控件 id
adapter.addOnItemChildClickListener(R.id.iv_num_add) { adapter, view, position ->
    Tips.show("onItemChildClick:  add $position")
}
```

- item 子控件长按事件

```kotlin
// 需要传递控件 id
adapter.addOnItemChildLongClickListener(R.id.btn_long) { adapter, view, position ->
    Tips.show("onItemChildLongClick $position")
    true
}
```

## 3.5 拖拽功能

- 这个功能的实现相对复杂，但是相较于原生编写，代码量任然有所降低。

1. 实现 DragAndSwipeDataCallback 接口

- adapter 实现 DragAndSwipeDataCallback 接口，并重写 dataSwap() 和 dataRemoveAt() 方法

```kotlin
class EmojiAdapter : BaseQuickAdapter<Emoji, EmojiAdapter.VH>() , DragAndSwipeDataCallback {
  
    /*  ...  */
  
    override fun dataMove(fromPosition: Int, toPosition: Int) {
      swap(fromPosition, toPosition)
      notifyItemMoved(fromPosition, toPosition)
    }

  override fun dataRemoveAt(position: Int) {
     removeAt(position)
  }

}
```

2. 创建 QuickDragAndSwipe 对象

```kotlin
        val quickDragAndSwipe = QuickDragAndSwipe().apply {
            setDragMoveFlags(ItemTouchHelper.UP.or(ItemTouchHelper.DOWN))//可进行上下拖动，交换位置。 ItemTouchHelper.LEFT 允许向左拖动，ItemTouchHelper.RIGHT 允许向右拖动
            setSwipeMoveFlags(ItemTouchHelper.LEFT.or(ItemTouchHelper.RIGHT))//可进行左右滑动删除
        }
```

3. 附加到 RecyclerView 中

```kotlin
        quickDragAndSwipe.attachToRecyclerView(binding.recyclerView).apply {
  setDataCallback(adapter)
  setItemDragListener(object : OnItemDragListener {
    override fun onItemDragStart(viewHolder: RecyclerView.ViewHolder?, pos: Int) {
      Log.e("TAG", "拖拽前: $emojiList")
    }

    override fun onItemDragMoving(
      source: RecyclerView.ViewHolder,
      from: Int,
      target: RecyclerView.ViewHolder,
      to: Int
    ) {

    }

    override fun onItemDragEnd(viewHolder: RecyclerView.ViewHolder, pos: Int) {
      Log.e("TAG", "拖拽后: $emojiList")
    }

  })
  setItemSwipeListener(object : OnItemSwipeListener {
    override fun onItemSwipeStart(
      viewHolder: RecyclerView.ViewHolder?,
      bindingAdapterPosition: Int
    ) {

    }

    override fun onItemSwipeEnd(
      viewHolder: RecyclerView.ViewHolder,
      bindingAdapterPosition: Int
    ) {

    }

    override fun onItemSwiped(
      viewHolder: RecyclerView.ViewHolder,
      direction: Int,
      bindingAdapterPosition: Int
    ) {
      Log.e("TAG", "删除后: $emojiList")
    }

    override fun onItemSwipeMoving(
      canvas: Canvas,
      viewHolder: RecyclerView.ViewHolder,
      dX: Float,
      dY: Float,
      isCurrentlyActive: Boolean
    ) {

    }

  })
}
```

- 看看最终实现的效果：

<img src="https://i.postimg.cc/gjJpSyW2/Sep-26-2023-17-24-29.gif">