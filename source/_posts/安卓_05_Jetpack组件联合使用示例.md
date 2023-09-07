---
title: 'Jetpack 组件联合使用示例'
date: '2023-06-27'
description: '总结目前学过的 Jetpack 组件，并使用他们写一个口算练习App'
cover: 'https://www.helloimg.com/images/2023/05/12/oxIXog.jpg'
categories:

- 编程开发

tags:

- Kotlin
- Android
- Jetpack

---

# 一、概念汇总

## 1.1 ViewModel

- 官方定义：[官方文档](https://developer.android.google.cn/jetpack/androidx/explorer?hl=zh-cn)中并没有明确的定义。
- 按我的理解它就是用于存储和管理与用户界面相关的数据的，通过将数据与 UI 控制器（如 Activity 或 Fragment）分离，以更好地支持应用程序的生命周期管理。
- 简而言之，就是`帮我们管理数据，让数据变得清晰明白，知根知底。`

## 1.2 Navigation

- 官方定义：构建和组织应用内界面，处理深层链接以及在屏幕之间导航。
- 之前使用传统的 FragmentManager 和 FragmentTransaction 进行 Fragment 切换时，存在一些问题，如 Fragment 堆栈管理困难、Fragment 之间通信复杂等。Navigation 组件提供了更简单且一致的方式来管理 Fragment，处理这些问题更加容易。
- 简而言之，就是`让界面之间的跳转变得更统一、更清晰（可视化）。`

## 1.3 DataBinding

- 官方定义：使用声明性格式将布局中的界面组件绑定到应用中的数据源。
- 通过使用 DataBinding，我们可以在布局文件中直接引用应用程序的数据，并通过特定的语法将数据与界面元素进行绑定。这样，在数据发生变化时，布局中绑定的视图会自动更新，无需手动编写大量的 findViewById()、setText()、setOnClickListener() 等操作。
- 简而言之，就是`让你不需要写很多重复的代码来更新界面上的数据。`

# 二、实际运用

- 目前，管理数据的、管理页面的、绑定页面和数据的组件我们都有了，那么他们如何组合使用呢？我们通过一个`口算练习App`案例来实际运用一下他们。

## 2.1 导入依赖

- 我们要添加的只有一个 DataBinding 的依赖（navGraph 的依赖在创建导航图时会自动帮我们添加），在我们新建项目的的`app`级别的`build.gradle`文件中添加：

```groovy
android {
    buildFeatures {
        //开启DataBinding
        dataBinding = true
    }
}

dependencies {
    // 添加 DataBinding 的依赖项
    implementation 'androidx.databinding:databinding-runtime:7.3.1'
}
```

## 2.2 创建页面

### 2.2.1 文件

- 创建一个 Activity 页面，作为我们的 Fragment 容器。
- 创建四个 Fragment 页面，分别是 TitleFragment（主界面）、QuestionFragment（答题界面）、WinFragment（胜利界面）、LoseFragment（失败界面）。
- 创建好之后结构如下：

<img src="https://www.helloimg.com/images/2023/06/27/o41vaQ.png">

### 2.2.2 布局

- 这里不给出具体的布局文本，因为本身布局很简单，并且这并不是这次的重点，这里只提供一下预览视图，具体布局可以任意变化。
- 首先是 TitleFragment（主界面），内容很少，如下：

<img src="https://www.helloimg.com/images/2023/06/27/o41wbt.png" width="30%">

- 然后是 QuestionFragment（答题界面），主要用于答题，界面如下：

<img src="https://www.helloimg.com/images/2023/06/27/o410Du.png" width="30%">

- 最后是成功（超过最高分）和失败（未超过最高分）的界面，二者布局几乎一样：

<img src="https://www.helloimg.com/images/2023/06/27/o41QGv.png" width="30%"> <img src="https://www.helloimg.com/images/2023/06/27/o41pAE.png" width="30%">

## 2.3 设置导航图(navGraph)

### 2.3.1 新建导航图

- 点击 res 资源路径，右键 -> new -> Android Resource File -> 类型选择 Navigation(这一步会提示导入依赖，如果失败多试几次)

<img src="https://www.helloimg.com/images/2023/06/27/o417j9.png">

### 2.3.2 编辑导航图

- 将四个 Fragment 添加进来，并设置 TitleFragment 为`Start Destination`，然后将所有的跳转逻辑`action`添加进来，直接在视图中拉箭头指向就可以了。

<img src="https://www.helloimg.com/images/2023/06/27/o41U1P.png">

- 在 CalculationActivity 的布局文件中添加一个 NavHostFragment 控件，选择我们刚才创建的 nav_calc，将控件命名为fragment（也可以不改，都行）。

<img src="https://www.helloimg.com/images/2023/06/27/o41IEX.png">

- 最后的效果如图：

<img src="https://www.helloimg.com/images/2023/06/27/o4169g.png">

## 2.4 ViewModel

- 在项目目录下新建 MyViewModel 类，继承自`AndroidViewModel`，代码如下（有其他需求修改对应部分即可）：
```kotlin
class MyViewModel(application: Application, savedStateHandle: SavedStateHandle) :
    AndroidViewModel(application) {

    private val handle: SavedStateHandle

    private val spf: SharedPreferences
    
    var winFlag = false

    companion object {
        const val KEY_HIGH_SCORE = "key_high_score"
        const val KEY_LEFT_NUMBER = "key_left_number"
        const val KEY_RIGHT_NUMBER = "key_right_number"
        const val KEY_OPERATOR = "key_operator"
        const val KEY_ANSWER = "key_answer"
        const val KEY_CURRENT_SCORE = "key_current_score"
        const val SAVE_SHP_DATA_NAME = "save_shp_data_name"
    }

    init {
        this.handle = savedStateHandle
        spf = getApplication<Application>().getSharedPreferences(
            SAVE_SHP_DATA_NAME, Context.MODE_PRIVATE
        )
        if (!handle.contains(KEY_HIGH_SCORE)) {
            handle[KEY_HIGH_SCORE] = spf.getInt(KEY_HIGH_SCORE, 0)
            handle[KEY_LEFT_NUMBER] = 0
            handle[KEY_OPERATOR] = "+"
            handle[KEY_RIGHT_NUMBER] = 0
            handle[KEY_OPERATOR] = "+"
            handle[KEY_ANSWER] = 0
            handle[KEY_CURRENT_SCORE] = 0
        }
    }

    fun getLeftNumber(): MutableLiveData<Int> {
        return handle.getLiveData(KEY_LEFT_NUMBER)
    }
    
    fun getRightNumber(): MutableLiveData<Int> {
        return handle.getLiveData(KEY_RIGHT_NUMBER)
    }

    fun getOperator(): MutableLiveData<String> {
        return handle.getLiveData(KEY_OPERATOR)
    }

    fun getHighScore(): MutableLiveData<Int> {
        return handle.getLiveData(KEY_HIGH_SCORE)
    }

    fun getCurrentScore(): MutableLiveData<Int> {
        return handle.getLiveData(KEY_CURRENT_SCORE)
    }

    fun getAnswer(): MutableLiveData<Int> {
        return handle.getLiveData(KEY_ANSWER)
    }

    //生成算式
    fun generator() {
        val level = 100
        val random = Random()
        val x = random.nextInt(level) + 1
        val y = random.nextInt(level) + 1

        if (x % 2 == 0) {
            getOperator().value = "+"
            if (x > y) {
                getAnswer().value = x
                getLeftNumber().value = y
                getRightNumber().value = x - y
            } else {
                getAnswer().value = y
                getLeftNumber().value = x
                getRightNumber().value = y - x
            }
        } else {
            getOperator().value = "-"
            if (x > y) {
                getAnswer().value = x - y
                getLeftNumber().value = x
                getRightNumber().value = y
            } else {
                getAnswer().value = y - x
                getLeftNumber().value = y
                getRightNumber().value = x
            }
        }


    }

    //保存新纪录
    fun save() {
        val edit = spf.edit()
        getHighScore().value?.let { edit.putInt(KEY_HIGH_SCORE, it) }
        edit.apply()
    }


    //答对处理
    fun answerCorrect() {
        val currentScore = getCurrentScore().value ?: 0
        val highScore = getHighScore().value ?: 0
        val updatedCurrentScore = currentScore + 1
        //赋值最新得分结果
        getCurrentScore().value = updatedCurrentScore
        //如果超过最高分，就更新记录
        if (updatedCurrentScore > highScore) {
            getHighScore().value = updatedCurrentScore
            winFlag = true
        }
        generator()
    }
}
```
- `注：`AndroidViewModel 是 ViewModel 的一个子类，它专门用于与 Android 系统相关的操作，如访问 Application 的上下文（Context）和共享数据。通过继承 AndroidViewModel，我们可以在 MyViewModel 中获取到 Application 对象，并使用它来获取 SharedPreferences，实现数据的持久化存储。这里主要是保存最高纪录使用。

## 2.5 DataBinding

- 首先，将四个Fragment的布局界面都转化为 DataBinding 界面，并绑定ViewModel。

<img src="https://www.helloimg.com/images/2023/06/27/o418kM.png">

```xml
    <data>

        <variable
            name="data"
            type="com.leihao.kotlinapp.calculation.viewmodel.MyViewModel" />
    </data>
```
### 2.5.1 TitleFragment 页面

- 只有最高分需要用到 DataBinding，在布局文件对应最高分 TextView 控件中添加属性：
```xml
<TextView 
        android:text="@{@string/high_score(data.highScore)}"
/>
```
其中，@string/high_score(data.highScore)的写法是为了和 strings.xml 中的：
```xml
<string name="high_score">历史最高分：%d</string>
```
进行一个匹配，这样就无需关注字符串的拼接问题了，后面类似的地方也是这样处理的。

### 2.5.2 Question 页面

- 当前得分的 TextView 添加属性：
```xml
<TextView 
        android:text="@{@string/current_score(data.currentScore)}"
/>
```
右边数字，运算符，右边数字对应的 TextView 分别添加下面三项：
```xml
<TextView
        android:text="@{String.valueOf(data.leftNumber)}"
/>
```
```xml
<TextView
        android:text="@{data.operator}"
/>
```
```xml
<TextView
        android:text="@{String.valueOf(data.rightNumber)}"
/>
```

### 2.5.3 WinFragment 和 LoseFragment 页面

- 在分数对应的 TextView 中分别添加属性：
```xml
<TextView
        android:text="@{@string/win_score_message(data.currentScore)}"
/>
```
```xml
<TextView
        android:text="@{@string/lose_score_message(data.currentScore)}"
/>
```
- 大功告成！现在就已经将界面和视图进行了一个绑定，当数据变化时，界面会自动观察到数据变化并做相应的更新。

## 2.6 功能完善

- 接下来将各个页面的剩余功能完成。
- TitleFragment：
```kotlin
class TitleFragment : Fragment() {
    private lateinit var binding: FragmentTitleBinding

    //快速绑定ViewModel，如果是Fragment需要绑定activity级别的viewModel，避免使用了不同的viewModel导致数据不一致
    private val myViewModel: MyViewModel by activityViewModels()


    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        binding = FragmentTitleBinding.inflate(inflater, container, false)
        //将 ViewModel 中的数据与布局文件进行绑定
        binding.data = myViewModel
        //设置 binding.lifecycleOwner，这样当 myViewModel 中的数据改变时就能及时更新到视图上。
        binding.lifecycleOwner = this
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //去答题界面
        binding.btnBegin.setOnClickListener {
            val navController = Navigation.findNavController(view)
            navController.navigate(R.id.action_titleFragment_to_questionFragment)
        }
    }
}
```

- QuestionFragment：
```kotlin
class QuestionFragment : Fragment() {
    private lateinit var binding: FragmentQuestionBinding

    //快速绑定ViewModel，如果是Fragment需要绑定activity级别的viewModel，避免使用了不同的viewModel导致数据不一致
    private val myViewModel: MyViewModel by activityViewModels()


    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        binding = FragmentQuestionBinding.inflate(inflater, container, false)

        //将 ViewModel 中的数据与布局文件进行绑定
        binding.data = myViewModel
        //设置 binding.lifecycleOwner，这样当 myViewModel 中的数据改变时就能及时更新到视图上。
        binding.lifecycleOwner = this

        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        //生成题目
        myViewModel.generator()

        //当前分数置0
        myViewModel.getCurrentScore().value = 0

        //显示输入的内容或者提示信息
        val builder = StringBuilder()

        val listener = View.OnClickListener {
            when (it.id) {
                R.id.button0 -> builder.append("0")
                R.id.button1 -> builder.append("1")
                R.id.button2 -> builder.append("2")
                R.id.button3 -> builder.append("3")
                R.id.button4 -> builder.append("4")
                R.id.button5 -> builder.append("5")
                R.id.button6 -> builder.append("6")
                R.id.button7 -> builder.append("7")
                R.id.button8 -> builder.append("8")
                R.id.button9 -> builder.append("9")
                R.id.button_clear -> builder.setLength(0)
            }

            if (builder.isEmpty()) {
                binding.textView10.text = getString(R.string.input_indicator)
            } else {
                binding.textView10.text = builder.toString()
            }
        }
        //提交答案
        binding.buttonSubmit.setOnClickListener {
            if (builder.isNotEmpty()) {
                //计算正确
                if (Integer.valueOf(builder.toString()).equals(myViewModel.getAnswer().value)) {
                    myViewModel.answerCorrect()
                    builder.setLength(0)
                    binding.textView10.text = getString(R.string.answer_correct)
                } else {//计算错误
                    val navController = Navigation.findNavController(it)
                    if (myViewModel.winFlag) {
                        navController.navigate(R.id.action_questionFragment_to_winFragment)
                        myViewModel.winFlag = false
                        myViewModel.save()
                    } else {
                        navController.navigate(R.id.action_questionFragment_to_loseFragment)
                    }
                }
            }
        }
        binding.button0.setOnClickListener(listener)
        binding.button1.setOnClickListener(listener)
        binding.button2.setOnClickListener(listener)
        binding.button3.setOnClickListener(listener)
        binding.button4.setOnClickListener(listener)
        binding.button5.setOnClickListener(listener)
        binding.button6.setOnClickListener(listener)
        binding.button7.setOnClickListener(listener)
        binding.button8.setOnClickListener(listener)
        binding.button9.setOnClickListener(listener)
        binding.buttonClear.setOnClickListener(listener)
    }
}
```

- WinFragment 和 LoseFragment，二者几乎完全一致，所以只展示 WinFragment：
```kotlin
class WinFragment : Fragment() {

    private lateinit var binding: FragmentWinBinding

    //快速绑定ViewModel，如果是Fragment需要绑定activity级别的viewModel，避免使用了不同的viewModel导致数据不一致
    private val myViewModel: MyViewModel by activityViewModels()


    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        binding = FragmentWinBinding.inflate(inflater, container, false)

        //将 ViewModel 中的数据与布局文件进行绑定
        binding.data = myViewModel
        //设置 binding.lifecycleOwner，这样当 myViewModel 中的数据改变时就能及时更新到视图上。
        binding.lifecycleOwner = this

        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.btnBack.setOnClickListener {
            val navController = Navigation.findNavController(it)
            navController.navigate(R.id.action_winFragment_to_titleFragment)
        }
    }
}
```

# 三、总结

- 通过使用 ViewModel 帮我们管理数据，Navigation 帮我们管理页面，DataBinding 帮我们绑定视图和数据，我们很轻松的就实现了一个较为符合开发规范的应用。
- 在 NavGraph 中，我们一眼就能看出整个应用的骨架，这对 Debug 或者新人熟悉项目都有非常大的益处！
