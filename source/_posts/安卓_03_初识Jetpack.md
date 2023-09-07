---
title: '初识 Jetpack'
date: '2023-05-12'
description: '初步了解Jetpack的概念和目标，并介绍LiveCycle、LiveData和ViewModel组件。'
cover: 'https://www.helloimg.com/images/2023/05/12/oxIXog.jpg'
categories:

- 编程开发

tags:

- Kotlin
- Android
- Jetpack

---

# 一、Jetpack 的概念和目标

## 1.1 概念

- Jetpack是一个由Google官方推出的Android应用程序开发工具包。
- 举个通俗易懂的例子：Jetpack就像是一个庞大的工具箱，里面装满了各种适用于Android应用程序开发的工具。这些工具可以帮助开发者解决诸如生命周期管理、界面设计、数据库访问、数据绑定、通讯等各种常见问题，让开发者能够更专注于业务逻辑的编写，减少样板代码的编写和一般性问题的处理。
- 通过Jetpack提供的各种组件和库，可以较为轻松地完成包括基本的Activity和Fragment开发，到Room Database、Data Binding、ViewModel、LiveData、WorkManager、Navigation等高级组件的开发，Jetpack是提高Android应用程序开发效率和质量的重要工具之一。

## 1.2 目标

- 旨在提供一系列库和工具，帮助开发者`更快`、`更简便`地构建`高质量`的Android应用程序。

# 二、Jetpack 的诞生和发展

## 2.1 诞生

- Jetpack，本身这个单词的含义是喷气背包、喷气发动机组件。在2018年的Google I/O大会上，谷歌将其最新推出的开发工具包命名为Jetpack，也许是为了强调其在应用程序开发中的"引擎"或"助推器"的作用，同时也传递了一个信息：Jetpack可以帮助开发者更快地构建高质量的Android应用程序。Jetpack应该被视为一种可靠的、可扩展的、高效的开发工具，可以帮助开发者实现目标并推动应用程序向前发展。
- 最初的 Jetpack 图标是一个背着喷气背包的 Android 机器人，可以说是非常形象生动了。

<img src="https://www.helloimg.com/images/2023/05/12/oxIHzc.png" width="30%">

## 2.2 发展

- 当Jetpack最初推出时，它只包含少量的库，比如Lifecycle和ViewModel等。这些库主要是为了帮助开发人员更轻松地编写高质量的应用程序，同时还可以在不同版本的Android操作系统之间保持一致性。
- 随着时间的推移，Jetpack逐渐得到了扩展和完善。目前，Jetpack已经成长为一个包含多个库和工具的全面开发平台，其中包括：Room、WorkManager、Navigation、Paging、Data Binding等，涵盖了众多常见的开发需求和场景。
- 2019年，Google宣布Jetpack Compose库，这是一种基于Kotlin语言的声明式UI开发工具，旨在通过简化UI组件的创建和交互，改进应用程序开发的速度和质量。
- 2021年，还推出了Jetpack Compose for Web和Jetpack Compose for Desktop，这些新领域的Compose库进一步拓展了Jetpack的应用范围，让开发者可以更轻松地构建跨平台的应用程序。
- 总的来说，Jetpack作为谷歌为Android应用程序开发者提供的全面工具包和平台，不断发展，不断更新，为开发者提供了更多便利和效率，也促进了Android生态系统的健康发展。


# 三、Jetpack 的运用

- 说明：
  1. 由于我也是初步接触到 Jetpack，所以对很多组件的认知有限。因此，今天就我目前接触到的一些组件做一些学习分享，可能会有错误的地方，多多担待。
  2. 为了与较新的技术接轨，同时提高自己的 kotlin 编码能力，所以后续涉及到 Jetpack 的内容（包括后续的博客），编程语言都会使用`Kotlin`。

## 3.1 LiveCycle

- Lifecycle 这个词的含义就是生命周期，在各种各样的开发中，生命周期这个词出现的频率非常之高。在安卓开发领域，Activity 和 Fragment 等组件的生命周期管理可能是开发者每天都要接触到的事。
- 作为 Jetpack 库中的一个组件，Lifecycle 主要用于管理Activity和Fragment等组件的生命周期。通过使用Lifecycle组件，开发者可以更方便地编写响应生命周期事件的代码，避免内存泄漏、资源浪费等问题。
- Lifecycle 组件提供了一个LifecycleOwner接口和一个LifecycleObserver接口，分别表示具有生命周期的组件和观察者，开发者可以通过实现这些接口来实现对组件的生命周期管理。

下面是代码示例：

```kotlin
import androidx.lifecycle.LifecycleObserver
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.OnLifecycleEvent

class MyLifecycleObserver : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate(owner: LifecycleOwner) {
        // 执行 onCreate 事件
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onStart(owner: LifecycleOwner) {
        // 执行 onStart 事件
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun onResume(owner: LifecycleOwner) {
        // 执行 onResume 事件
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun onPause(owner: LifecycleOwner) {
        // 执行 onPause 事件
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onStop(owner: LifecycleOwner) {
        // 执行 onStop 事件
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy(owner: LifecycleOwner) {
        // 执行 onDestroy 事件
    }
}
```
- 上述代码创建了一个创建了一个名为MyLifecycleObserver的类，并实现了LifecycleObserver接口。该类提供了一些方法，用于响应不同的生命周期事件。在每个方法上，我们使用@OnLifecycleEvent注解标记要响应的事件类型，并在方法体中编写对应的业务逻辑。
- 下面演示如何使用：

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var myObserver: MyLifecycleObserver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 创建 MyLifecycleObserver 实例
        myObserver = MyLifecycleObserver()
        
        // 将 myObserver 添加到 Activity 的生命周期观察者列表中
        lifecycle.addObserver(myObserver)
    }
}
```
- 在不需要时，将MyLifecycleObserver实例从Activity的生命周期观察者列表中移除。如下代码所示：

```kotlin
override fun onDestroy() {
    super.onDestroy()
    //从观察者列表中移除
    lifecycle.removeObserver(myObserver)
}
```

- 如此一来，开发者可以更容易地管理和响应组件生命周期事件，减少代码的冗余和复杂性。

## 3.2 LiveData 和 ViewModel

- 由于 LiveData 和 ViewModel 在一般情况下都是一起使用，因此这里将他们放在一起讲解。
- LiveData 的作用主要是将数据从 ViewModel 传递到相应的 View 上，并保证这种数据传递的安全性和正确性。是一种可观察的数据持有类，它可以感知生命周期并在数据变化时通知观察者。
- ViewModel 是 MVVM 架构中的一部分，用于管理应用程序的数据和业务逻辑。它通常与 LiveData 一起使用，可以将业务逻辑和 UI 组件进行解耦，使得数据持久性和业务逻辑不受 UI 生命周期的影响。
- 在 Android 中，每个 Activity 或 Fragment 都有其自己的生命周期，当这些 UI 组件因为`旋转手机、配置更改（如切换语言）、资源内存不足`等原因被销毁并重新创建时，会导致其中包含的数据丢失，从而影响用户体验。而 ViewModel 的引入，则可以帮助我们在这些 UI 组件被销毁重建时，保持其中的数据状态不变。
下面演示一下横屏导致数据丢失的情况：

<img src="https://www.helloimg.com/images/2023/05/12/oxIfwn.gif" width="30%">

- 下面展示如何使用 ViewModel 和 LiveData 技术来实现数据的绑定和更新。

1. 先自定义一个继承自 ViewModel 的 DataBindingViewModel 类
```kotlin
// 定义一个 MyViewModel 类，继承自 ViewModel
class MyViewModel : ViewModel() {
  // 定义一个 MutableLiveData 类型的 number 变量，用于存储需要展示到 UI 上的数字
  var number: MutableLiveData<Int> = MutableLiveData()

  // addOne 方法，用于将 number 自增 1
  fun addOne() {
    // 使用 Elvis 运算符获取 number 当前的值，如果为 null 则默认为 0
    number.value = (number.value ?: 0) + 1
  }
}
```
2. 在 Activity 中使用
```kotlin
class ViewModelActivity : AppCompatActivity() {
    // 定义一个 lateinit 的属性 binding，类型为 ActivityViewModelBinding
    private lateinit var binding: ActivityViewModelBinding
    // 定义一个属性 myViewModel，使用了 viewModels() 函数来创建 MyViewModel 对象
    private val myViewModel: MyViewModel by viewModels()
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 使用 ActivityViewModelBinding 类中的 inflate 函数来创建 binding 对象
        binding = ActivityViewModelBinding.inflate(layoutInflater)
        // 设置界面布局为 binding.root
        setContentView(binding.root)
    
        // 观察 myViewModel 中的 number 属性的变化
        myViewModel.number.observe(this, Observer {
          // 将 number 的值显示在 textView 上
          binding.textView.text = myViewModel.number.value.toString()
        })
    
        // 给 button 的点击事件添加监听器
        binding.button.setOnClickListener {
          // 在 myViewModel 中执行 addOne 函数
          myViewModel.addOne()
        }
    }
}
```
3. 实现效果

<img src="https://www.helloimg.com/images/2023/05/12/ox6Pwr.gif" width="30%">

- 上述示例中，我们使用了 LiveData 和 ViewModel 这两个组件来实现 Activity 销毁并重建时数据不会丢失的功能。
- 总的来说，LiveData 和 ViewModel 是 Android Jetpack 组件库中非常重要的两个组件，它们能够帮助我们实现应用程序中大量的业务逻辑和提高应用程序性能，并且它们的生命周期与其关联的组件相对应，能够更好地处理组件销毁等情况，从而避免数据丢失或内存泄漏等问题。

# 四、总结

- 由于目前只接触到这几个组件，先分享到这里，体验下来的感觉可以说非常奇妙。原来数据还能这样更新，原来视图还能这样绑定...或许接触新知识的最大的乐趣就在于此吧。
- 这两天使用 kotlin 进行编码，也让我对 Kotlin 语法'太甜了'的感觉有所改善，确实在某些时候能感觉到这门语言的便利之处，希望在后续学习中能见识到这门语言更强大的地方。