---
title: 'Navigation 组件'
date: '2023-05-13'
description: 'Android Jetpack 组件之 Navigation'
cover: 'https://www.helloimg.com/images/2023/05/13/oxuaj9.png'
categories:

- 编程开发

tags:

- Kotlin
- Android
- Jetpack

---

> Navigation 是一个框架，用于在 Android 应用中的“目的地”之间导航，该框架提供一致的 API，无论目的地是作为 Fragment、Activity 还是其他组件实现。

# 一、Navigation  

## 1.1 概念

- Jetpack Navigation 组件是 Android Jetpack 中的一部分，它提供了一种简单而强大的方式来实现在应用程序中进行导航和交互的标准化方法。
- 使用 Jetpack Navigation 组件可以轻松实现复杂的 UI 导航功能。与传统的 Fragment 事务相比，它们更加标准化、可重用和易于维护。此外，该组件还可以自动生成深度链接和测试导航行为。在现代的 Android 应用程序设计中，Jetpack Navigation 组件已成为 UI 导航的首选方法之一。

## 1.2 组成部分

- 该组件包括以下几个基本部分：

  1. 导航图 （Navigation Graph）：一个 XML 文件，它定义了应用程序中的所有目标和路径，以及如何在应用程序中移动。
  2. 目标（Destination）：表示应用程序中的屏幕或 UI 视图。例如，一个应用程序可能有一个主屏幕，一个设置屏幕和一个详细信息屏幕。
  3. 动作（Action）：表示从一个屏幕到另一个屏幕之间的转换。例如，从主屏幕转到详细信息屏幕，用户需要单击列表项。
  4. 导航控制器（NavController）：负责跟踪应用程序的当前目标和路径，并处理导航操作以更改这些值。
  5. 依赖注入（Dependency Injection）：Navigation 组件提供了 ViewModel、SavedStateHandle 等支持依赖注入的类，方便开发者使用。

# 二、Navigation 的使用

## 2.1 添加依赖
- 在 build.gradle(app) 中添加依赖（kotlin）：
```groovy
dependencies {
  
    implementation 'androidx.navigation:navigation-fragment-ktx:2.5.2'
    implementation 'androidx.navigation:navigation-ui-ktx:2.5.2'

}
```

## 2.2 创建两个 Fragment 示例
- 直接 new -> Fragment -> Fragment(Blank) 即可，一个取名 HomeFragment 作为主页，一个取名 DetailFragment 作为跳转的详情页。
- 下面是 HomeFragment 的示例：
```kotlin
class HomeFragment : Fragment() {

    private lateinit var binding: FragmentHomeBinding
  
    override fun onCreateView(
      inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View? {
      // Inflate the layout for this fragment
      binding = FragmentHomeBinding.inflate(inflater, container, false)
      return binding.root
    }
  
}
```
- 生成的多余内容可以删掉，只保留 onCreateView 即可。我这里使用了 ViewBinding，也可以使用 DataBinding 或者就 findViewById 都行。
- 布局文件非常简单，两个页面都放置一个 TextView 和 一个 Button 即可。如下图：

<img src="https://www.helloimg.com/images/2023/05/13/oxur3M.png" width="30%">

## 2.3 创建导航图(Navigation Graph)

- 导航图是一个 XML 文件，它定义了应用程序中的所有目标和路径，以及如何在应用程序中移动。
- 创建： res -> new -> Android Resource File
- 名称可以任意取一个满足要求的即可，这里取 my_nav.xml，重要的是资源类型 Resource Type，选择`Navigation`。

<img src="https://www.helloimg.com/images/2023/05/13/oxuVMP.png">

- 导入刚才创建的 Fragment：

<img src="https://www.helloimg.com/images/2023/05/13/oxy3p1.png">

- `tips`: 如果发现主页不是 HomeFragment，可以点击左侧Component Tree中的 HomeFragment，然后右键选择Set as Start Destination 设置其为主页。更多的设置项可以自己探索。

<img src="https://www.helloimg.com/images/2023/05/13/oE5kD6.png" width="30%">

## 2.4 建立导航图与 NavHostFragment 的关联

- 在我们的宿主 Activity 的布局页面中，插入一个 NavHostFragment 布局控件：
- 在弹出框中选择我们刚才创建好的Navigation Graph，即 my_nav.xml，并将其命名为 fragment (可使用默认名称)，完成后效果如下：

<img src="https://www.helloimg.com/images/2023/05/13/oE5RtE.jpg">

## 2.5 使用 NavController 完成跳转

- 在 HomeFragment 中，重写 onViewCreated() 方法，我们在里面进行按钮点击事件的监听。
- onViewCreated() 是 Fragment 的生命周期方法之一，用于在 Fragment 的视图层次结构被创建后执行自定义逻辑。在该方法中，您可以访问 Fragment 的根视图和子视图，并执行任何与视图相关的操作，例如初始化 UI 元素、设置监听器、加载数据等等。
```kotlin
class HomeFragment : Fragment() {

  //复写 onViewCreated() 方法
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //设置按钮监听
        binding.todetail.setOnClickListener {
              //通过传入 it 参数 (即按钮的 View 对象) 获取与当前 Fragment 关联的 NavController。
              val controller = Navigation.findNavController(it)
              //调用 NavController 的 navigate() 方法将应用程序从 HomeFragment 导航到 DetailFragment。
              controller.navigate(R.id.action_homeFragment_to_detailFragment)
        }
        
    }
}
```
- 重点是这两行代码：
```kotlin
    val controller = Navigation.findNavController(it)
    controller.navigate(R.id.action_homeFragment_to_detailFragment)
```
- 首先是第一行，用以获取与当前 Fragment 关联的 NavController。
- `NavController` 是 Navigation 组件的核心部分，负责管理应用程序的导航。每个 Destination（目标）都有一个相关的 NavGraph（导航图），NavController 将根据用户操作将应用程序从一个 Destination 导航到另一个 Destination。
- 第二行代码即调用 NavController 的 navigate() 方法，将应用程序从 HomeFragment 导航到 DetailFragment。
- `R.id.action_homeFragment_to_detailFragment`是自动（刚才拖动视图右侧圆点绑定跳转动作时）生成的，意为从 HomeFragment 到 DetailFragment 的动作。

- 在 DetailFragment 中进行同样的操作即可，这里不再给出代码。

## 2.6 将导航过程与应用程序的 ActionBar 集成起来。

- 现在已经可以通过点击按钮实现 Fragment 跳转了，现在需要实现 ActionBar 栏的点击返回箭头实现返回主页的功能。

<img src="https://www.helloimg.com/images/2023/05/13/oxygB1.jpg" width="30%">

- 在 Activity 中添加以下代码：
```kotlin
class NavigationActivity : AppCompatActivity() {

    private lateinit var binding: ActivityNavigationBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityNavigationBinding.inflate(layoutInflater)
        setContentView(binding.root)
      
        // 首先通过 supportFragmentManager 获取 Fragment 的管理器，并使用 R.id.fragment 找到当前活动的 Fragment。
        // 然后，我们调用 Fragment 的 findNavController() 方法获取与之关联的 NavController
        val navController = supportFragmentManager.findFragmentById(R.id.fragment)?.findNavController()
      
        if (navController != null) {
            //使用 NavigationUI 类的 setupActionBarWithNavController() 方法将指定 Activity 的 ActionBar 与 NavController 关联起来
            NavigationUI.setupActionBarWithNavController(this, navController)
        }
      
    }
    //重新获取与 Activity 中当前 Fragment 关联的 NavController，然后调用 navigateUp() 方法以返回上一个 Fragment。
    override fun onSupportNavigateUp(): Boolean {
        val controller = Navigation.findNavController(this, R.id.fragment)
        return controller.navigateUp()
    }
}
```
- 实现的效果：

<img src="https://www.helloimg.com/images/2023/05/13/oxyDKc.gif"  width="30%">

- 目前为止，就实现了两个 Fragment 之间的随意切换，再也不用频繁的进行 Fragment 事务的处理了！
- 使用 Navigation 这种方式，使得添加 Fragment 非常简单，还可以自由的传递数据、设置跳转动画效果等等。

# 三、总结

- 在大型项目中，如果没有一个良好的导航体系，那么我们很容易陷入混乱，无法有效地管理 Fragment。而使用 Navigation 组件，我们可以轻松实现 Fragment 的模块化和复用，减少重复代码的编写，提高代码的可维护性和扩展性。
- 可能只有两个 Fragment 还体现不出 Navigation 强大之处，但是能想象得出，在 Fragment 增多时，这种可视化的操作会使代码更加简洁易懂，让我们的逻辑也更为清晰。
- 当然，本文只是简单介绍了 Navigation 的使用，Navigation 还有更多的功能等待我们进一步的探索！