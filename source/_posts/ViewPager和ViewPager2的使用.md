---
title: 'ViewPager 与 ViewPager2 的使用示例'
date: '2023-08-15'
description: '介绍如何使用 ViewPager + BottomNavigationView 实现底部导航，ViewPager2 + TabLayout 实现顶部导航。'
cover: 'https://z1.ax1x.com/2023/09/25/pP7yp5j.png'
categories:

- 编程开发

tags:

- Kotlin
- Jetpack
- Android

---

# 一、概念及组成部分

## 1.1 概念

- ViewPager 和 ViewPager2 看名字就知道是一代和二代的关系，他们一同被添加到 JetPack 组件库中，不过一代目前已经不推荐使用。

## 1.2 用途

- `以可滑动的格式显示视图或 Fragment。`文档这一句话已经将用途说得很清楚了，说白了就是让界面可以左右滑动，承担视图或者 Fragment 的切换工作。当然，Fragment 也算是一种视图控件，只不过有自己独立的生命周期而已。
- 使用 ViewPager 或 ViewPager2，可以将不同的 Fragment 或 View 作为页面，并轻松地进行页面切换和管理。这对于构建需要在多个页面之间切换的应用程序非常有用，例如图片浏览器、新闻阅读器、引导页等。

# 二、基本使用

## 2.1 ViewPager + BottomNavigationView

- ViewPager 不论是一代还是二代，一般都是配合其它控件进行使用的，比如这里举一个 ViewPager + BottomNavigationView 的例子。
- BottomNavigationView 简单介绍一下，它是一个底部导航视图，通常由多个图标和标签组成，用于在应用程序的底部提供导航菜单栏。
- 可以看出，我们要实现的就是点击 BottomNavigationView 的某个 item 时，切换到指定的 Fragment，并且 ViewPager 进行同步的切换。或者，滑动 ViewPager 时，切换到指定的 Fragment，并且 BottomNavigationView 切换到对应的 item。总结一下就是 `BottomNavigationView + ViewPager + Fragment 的三者联动`。

### 2.1.1 创建 menu

- 由于 BottomNavigationView 需要配合一个 menu 进行使用，所以我们先新建一个 menu 作为底部导航栏的可选项 item。

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/todoFragment"
        android:icon="@drawable/record_24"
        android:title="@string/record" />
    <item
        android:id="@+id/accountFragment"
        android:icon="@drawable/shopping_24"
        android:title="@string/account" />
    <item
        android:id="@+id/mineFragment"
        android:icon="@drawable/account_24"
        android:title="@string/mine" />
</menu>
```

- 每个 item 的 id 用于唯一标识底部导航栏中的各个选项，通过这些唯一的 id 来区分不同的选项，并在代码中进行相应的处理。

### 2.1.2 创建主界面

- 创建一个包含 ViewPager 和 BottomNavigationView 的页面，一般将 BottomNavigationView 放在下方，其余部分用 ViewPager 填充。不过你当然也可以将 BottomNavigationView 放在你想放的任意位置，只要它拥有自己的 id 就行。
- 将上一步创建好的 menu 放入 BottomNavigationView 中，用`app:menu="@menu/your_menu"`的方式，下面是示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".activity.HomeActivity">

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/viewPager_fragments"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/bottomNavigationView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNavigationView"
        style="@style/Widget.Design.BottomNavigationView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/white"
        app:itemBackground="@color/white"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:menu="@menu/mint_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

- 这个时候在 Design 界面上就已经可以看到一些预览效果了，但是现在点击和滑动是没有任何关联的，并且 ViewPager 里面现在还没有内容呢。

### 2.1.3 配置 ViewPager 适配器

- 创建一个继承自 FragmentStatePagerAdapter 的适配器类，在稍微高一点的 SDK 版本中，可以看到 IDE 提示该类已弃用，毕竟有更好的替代了嘛。我们先不管这个提示，暂时现在还是可用的，只是未来某个 SDK 更新可能就移除掉了。

```kotlin
class HomeActivityAdapter(fm: FragmentManager, private val fragmentList: MutableList<Fragment>) :
    FragmentStatePagerAdapter(fm) {

    override fun getCount(): Int {
        return fragmentList.size
    }

    override fun getItem(position: Int): Fragment {
        return fragmentList[position]
    }

}
```

- 在创建时我们可以看到，它提示我们必须实现一个带 FragmentManager 为参数的构造器，原因是 FragmentStatePagerAdapter 内部需要使用 FragmentManager 来创建和管理 Fragment。
- 接下来，在 Activity 中将适配器设置给 ViewPager：

```kotlin
        // -- 省略 -- 
        fragmentList = ArrayList()

        val todoFragment = TodoFragment()
        val accountFragment = AccountFragment()
        val mineFragment = MineFragment()

        fragmentList.add(todoFragment)
        fragmentList.add(accountFragment)
        fragmentList.add(mineFragment)
        //实例化适配器
        homeActivityAdapter = HomeActivityAdapter(supportFragmentManager, fragmentList)
        //将适配器绑定给 ViewPager
        binding.viewPagerFragments.adapter = homeActivityAdapter
        // -- 省略 -- 
```

### 2.1.4 实现联动

- 滑动viewpager时，联动底部按钮：

```kotlin
            binding.viewPagerFragments.setOnPageChangeListener(object : OnPageChangeListener {
            override fun onPageScrolled(
                position: Int, positionOffset: Float, positionOffsetPixels: Int
            ) {

            }

            override fun onPageSelected(position: Int) {
                //设置顶部的 actionBar 标题变化为 bottomNavigationView 的 menu 对应 item 的"android:title"字段的值
                supportActionBar?.title = binding.bottomNavigationView.menu.getItem(position).title
                //判断 ViewPager 滑动的位置，并且根据位置更新 bottomNavigationView 的选中条目 id
                when (position) {
                    0 -> binding.bottomNavigationView.selectedItemId = R.id.todoFragment
                    1 -> binding.bottomNavigationView.selectedItemId = R.id.accountFragment
                    2 -> binding.bottomNavigationView.selectedItemId = R.id.mineFragment
                }

            }

            override fun onPageScrollStateChanged(state: Int) {

            }

        })
```

- 点击底部按钮时，联动viewpager进行滑动：

```kotlin
            binding.bottomNavigationView.setOnNavigationItemSelectedListener { item ->
            //在 Item 更新时，同步更新 ViewPager 的当前 item 项
            when (item.itemId) {
                R.id.todoFragment -> {
                    binding.viewPagerFragments.currentItem = 0
                }

                R.id.accountFragment -> {
                    binding.viewPagerFragments.currentItem = 1
                }

                R.id.mineFragment -> {
                    binding.viewPagerFragments.currentItem = 2
                }
            }
            true
        }
```

- 到此为止，就实现了一个简单的底部导航 + 滑动视图切换的功能。不过上面用到的大多数方法或类都已经弃用，包括`setOnPageChangeListener`和`setOnNavigationItemSelectedListener`和前面提到的`FragmentStatePagerAdapter`。这样一看全是弃用的 API，如果是用在比较大的项目中显然是不合适的。

## 2.2 ViewPager2 + TabLayout

- 接下来介绍一下 ViewPager2 的使用，相比于一代，性能更优秀，使用更方便，当然也不会有那么多被弃用的 API 了。与其联动的控件不再选择 BottomNavigationView 而选择 TabLayout，当然喽，选择 BottomNavigationView 也是完全 ok 的，这里只是多举一些其他的情况。
- TabLayout 也简单介绍一下，看名字和前面的 BottomNavigationView 像是两个相反作用的控件，实际也确实如此，TabLayout 是 Material Design 中的一个标签栏控件，用于显示多个选项卡，并与 ViewPager2 进行联动。功能和 BottomNavigationView 几乎一致，不过一般用于顶部的选项卡切换。
- TabLayout 和 BottomNavigationView 在我看来其实是差不多的，他们所处的位置完全可以自己决定，样式也可以通过配置变得完全一样，但是还是尽量按照约定来行使各自的功能，毕竟我们的代码不仅要自己能看懂，也要尽量让别人看得舒服。

### 2.2.1 创建页面

- 直接构建一个包含 TabLayout 和 ViewPager2 的页面：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/fragment_account"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".fragment.account.AccountFragment">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/TabLayout"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <com.google.android.material.tabs.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="Monday" />

        <com.google.android.material.tabs.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="Tuesday" />

        <com.google.android.material.tabs.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="Wednesday" />

    </com.google.android.material.tabs.TabLayout>

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/ViewPager2"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/TabLayout" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

- 我这里可以直接使用这两个控件，如果不能使用的话，去 Build.Gradle 文件中导入对应的依赖即可。

### 2.2.2 实现联动

- 没错，我们不需要再单独创建适配器了，可以直接使用 FragmentStateAdapter 创建一个匿名类实现适配器功能：

```kotlin
        // -- 省略 --
        fragmentsList = ArrayList()

        fragmentsList.add(PayFragment())
        fragmentsList.add(IncomeFragment())
        fragmentsList.add(ChartFragment())

        binding.ViewPager2.adapter = object : FragmentStateAdapter(requireActivity()) {
            //里面有几个页面
            override fun getItemCount() = fragmentsList.size

            //每一个位置对应哪个 Fragment
            override fun createFragment(position: Int) = when (position) {
                0 -> PayFragment()
                1 -> IncomeFragment()
                else -> ChartFragment()
            }

        }

        //协调 viewpager2 与 TabLayout 的辅助类
        TabLayoutMediator(
            binding.TabLayout, binding.ViewPager2
        ) { tab, position ->
            when (position) {
                0 -> tab.text = resources.getString(R.string.pay)
                1 -> tab.text = resources.getString(R.string.income)
                else -> tab.text = resources.getString(R.string.chart)
            }
        }.attach()

        // -- 省略 --
```

- 上面代码就实现了 TabLayout + ViewPager2 的联动效果，具体来说：
  1. 定义了一个继承自 FragmentStateAdapter 的匿名适配器类。在适配器中，重写了 getItemCount 方法，返回 fragmentsList 的大小，即页面的数量。然后，重写了 createFragment 方法，根据位置返回相应的 Fragment 实例。
  2. 使用 TabLayoutMediator 类来协调 TabLayout 和 ViewPager2 之间的联动。通过传入 TabLayout 和 ViewPager2 的实例，以及一个 lambda 表达式，关联了 TabLayout 中的选项卡和 ViewPager2 中的页面。lambda 表达式中的 tab 参数表示当前选项卡，position 参数表示当前选项卡的位置，在 lambda 表达式中根据位置设置选项卡显示的文本。
  3. 最后，调用 attach() 方法将 TabLayout 和 ViewPager2 进行关联。
- 代码量来说确实少一些，不过差异主要集中在 TabLayout 和 BottomNavigationView 的实现上，实际上两个 ViewPager 的实现代码的代码量差别并不大，但是还是更推荐使用较新的 ViewPager2 来进行 Fragment 的管理。

# 三、总结

- 当开发 Android 应用程序时，我们经常需要在不同的页面之间进行切换。为了满足这种需求，我们可以使用 ViewPager 或 ViewPager2 作为页面容器控件，并结合 BottomNavigationView 或 TabLayout 实现页面切换的联动效果。
- ViewPager 和 ViewPager2 都是用于实现页面切换的容器控件。ViewPager 适用于简单场景，而 ViewPager2 提供了更多的功能和更好的性能。BottomNavigationView 和 TabLayout 则分别为两者提供了与之联动的导航栏控件。无论选择哪种组合方式，都可以实现简洁、高效的页面切换效果，为用户提供良好的交互体验。