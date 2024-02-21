---
title: '使用 Paging2 实现按需加载功能'
date: '2023-11-25'
description: 'Paging2 虽然已经被 Paging3 替代，但我们依然可以通过它来了解谷歌对于滚动列表的性能和用户体验方面的思考'
cover: 'https://z1.ax1x.com/2023/11/25/piwjyXq.jpg'
categories:

- 编程开发

tags:

- Kotlin
- Android 
- Jetpack

--- 

# 一、概述

## 1.1 Paging 库的作用

- 我们在日常开发需求中肯定有过加载更多的需求吧，一次性加载太多数据会带来很多问题，比如更长的加载时间、更大的流量消耗、更重的视觉疲劳感，严重的甚至会出现内存溢出等性能问题。
- Paging 就是谷歌为我们提供的专门解决`按需加载`功能的库，中文也翻译为分页库。在 Paging 出现之前，我们对于不同的接口返回格式，会通过多种对应的方式实现按需加载功能，随着 JetPack 的普及，Paging 已经有越来越多的人使用，我们有必要掌握这一套规范的按需加载设计。
- 但就我查到的资料和结合自己的见闻来看，JetPack 中两个最叫好不叫座的组件就包含 Paging，另一个是 WorkManager。他们属于呼声高但浪花小，虽然都提供了在特定场景下非常有价值的解决方案，但是对于初学者来说，学习成本有点偏高了，远没有 ViewModel、Navigation、Room 等这些热门组件使用频率高。

## 1.2 Paging2

- Paging2 在很早就已经被 Paging3 替代，但是其设计思想我们还是可以学习一下的，也算是为学习 Paging3 铺铺路吧。为了简单易懂，并且本身 Paging2 的很多 API 已经标记为废弃了，所以这里就使用 Room + Paging2 实现一下本地数据的按需加载，网络数据的加载在后面 Paging3 时再做详细的介绍。
- 下面介绍用到：ViewModel + Room + Paging2 + Coroutines，实现一个最简单的加载更多 Demo。

# 二、使用

## 2.1 引入依赖

- 引入 Paging2 以及其他几个组件的相关依赖：

```groovy

plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt' //启用 Kotlin 的注解处理器功能
}

dependencies {

    implementation 'androidx.paging:paging-runtime-ktx:2.1.2'

    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.1'
    
    implementation 'androidx.room:room-runtime:2.4.2'

    // Room 库的编译时注解处理器（Java中使用 annotationProcessor）
    kapt 'androidx.room:room-compiler:2.4.2'
    
    /** ... **/
}
```

## 2.2 整理结构

- 麻雀虽小，五脏俱全。尽管是一个最简单的 Demo，我们也要将项目结构力所能及的规范化，各个类之间要分工明确，我们要整理清楚自己需要哪些类，下面是大致的结构：

```text
app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── com.example.demo/
│   │   │   │   ├── adapter/
│   │   │   │   │   └── StudentPagedAdapter.kt      // RecyclerView 适配器类
│   │   │   │   ├── dao/
│   │   │   │   │   └── StudentDao.kt               // 数据访问对象接口
│   │   │   │   ├── database/
│   │   │   │   │   └── StudentDatabase.kt          // 数据库类
│   │   │   │   ├── entity/
│   │   │   │   │   └── Student.kt                  // 数据实体类
│   │   │   │   ├── repository/
│   │   │   │   │   └── StudentRepository.kt        // 数据仓库类
│   │   │   │   ├── viewmodel/
│   │   │   │   │   └── StudentViewModel.kt         // ViewModel 类
│   │   │   │   ├── ui/
│   │   │   │   │   └── PagingActivity.kt           // 分页显示数据的 Activity 类
│   │   └── res/
│   └── test/
└── build.gradle
```

- 项目最终的结构如图：

<img src="https://z1.ax1x.com/2023/11/25/piwvRKI.png" width="30%">

## 2.3 开始编码

### 2.3.1 Student

- 首先，是我们最简单的实体类对象：Student，简单起见学生名字都不需要了，就一个学号即可：

```kotlin
@Entity(tableName = "student_table")
data class Student(
    @PrimaryKey(autoGenerate = true)
    var id: Int = 0,
    @ColumnInfo(name = "student_number")
    var studentNumber: Int = 0
)
```

### 2.3.2 StudentDao

- 接下来是学生的数据库访问对象 StudentDao，里面提供三个抽象方法：

```kotlin
@Dao
interface StudentDao {

    /**
     * 插入学生
     * @param student 学生对象
     */
    @Insert
    fun insertStudent(vararg student: Student)

    /**
     * 删除所有学生
     */
    @Query("DELETE FROM student_table")
    fun deleteAllStudent()

    /**
     * 获取所有学生
     * @return 学生数据源对象，用于分页加载数据
     */
    @Query("SELECT * FROM student_table ORDER BY id")
    fun getAllStudents(): DataSource.Factory<Int, Student> //Int表示主键类型，Student表示项目的数据模型，即从数据库中查询出来的每个学生对象

}
```

### 2.3.3 StudentDataBase

- 然后是数据库类，负责管理学生数据的持久化存储，提供一个 getInstance() 方法用于获取数据库连接，一个 getStudentDao() 方法获取与学生数据交互的数据访问对象：

```kotlin
@Database(entities = [Student::class], version = 1, exportSchema = false)
abstract class StudentDataBase : RoomDatabase() {

    //伴生对象，里面的方法可以视为Java中的静态方法，属性可以视为静态属性
    companion object {

        private var INSTANCE: StudentDataBase? = null

        // 获取单例实例的方法
        fun getInstance(context: Context): StudentDataBase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context, StudentDataBase::class.java, "student_db"
                ).build()
                INSTANCE = instance
                instance
            }
        }

    }

    abstract fun getStudentDao(): StudentDao

}
```

### 2.3.4 StudentRepository

- 接下来是数据仓库类：StudentRepository，用于协调本地数据源（如数据库）和远程数据源（如网络请求），换句话说就是执行具体的获取数据动作（绝大部分情况下都是耗时操作），并提供统一的数据操作接口给上层（ViewModel），它可能包含从本地数据库加载数据、从网络获取数据、缓存数据等功能。

```kotlin
class StudentRepository(context: Context) {

    private val studentDao by lazy { StudentDataBase.getInstance(context).getStudentDao() }

    suspend fun insertStudent(vararg student: Student) {
        withContext(Dispatchers.IO) {
            studentDao.insertStudent(*student)
        }
    }

    suspend fun deleteAllStudent() {
        withContext(Dispatchers.IO) {
            studentDao.deleteAllStudent()
        }
    }

    fun getAllStudents(): DataSource.Factory<Int, Student> {
        return studentDao.getAllStudents()
    }
}
```

### 2.3.5 StudentViewModel

- 接下来是 StudentViewModel，负责处理与学生数据相关的逻辑和交互：

```kotlin
class StudentViewModel(application: Application) : AndroidViewModel(application) {

    private val studentRepository = StudentRepository(application)

    fun insertStudent(vararg student: Student) {
        //使用协程执行插入动作
        viewModelScope.launch {
            studentRepository.insertStudent(*student)
        }
    }

    fun deleteAllStudent() {
        //使用协程执行删除动作
        viewModelScope.launch {
            studentRepository.deleteAllStudent()
        }
    }

    fun getAllStudents(): DataSource.Factory<Int, Student> {
        return studentRepository.getAllStudents()
    }

}
```

### 2.3.6 StudentPagedAdapter

- 适配器，非常重要，继承于 PagedListAdapter<T,VH>，实现加载更多的功能：

```kotlin
class StudentPagedAdapter :
    PagedListAdapter<Student, StudentPagedAdapter.MyViewHolder>(diffCallback) {
    companion object {
        private val diffCallback = object : DiffUtil.ItemCallback<Student>() {
            override fun areItemsTheSame(oldItem: Student, newItem: Student): Boolean {
                return oldItem.id == newItem.id
            }

            override fun areContentsTheSame(oldItem: Student, newItem: Student): Boolean {
                return oldItem.studentNumber == newItem.studentNumber
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val binding =
            CellPagingItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return MyViewHolder(binding)
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {

        val student = getItem(position)

        when (val binding = holder.binding) {
            is CellPagingItemBinding -> {
                if (student == null) {
                    //还没有数据时显示为loading
                    binding.textView.text = "loading"
                } else {
                    binding.textView.text = student.studentNumber.toString()
                }
            }
        }

    }

    class MyViewHolder(val binding: ViewBinding) : RecyclerView.ViewHolder(binding.root)

}
```

### 2.3.7 PagingActivity

- 最后就是显示数据，测试功能了：

```kotlin
class PagingActivity : AppCompatActivity() {

    private val binding: ActivityPagingBinding by lazy {
        ActivityPagingBinding.inflate(layoutInflater)
    }

    private val studentViewModel: StudentViewModel by viewModels()

    private lateinit var context: Context

    private lateinit var studentPagedAdapter: StudentPagedAdapter

    private lateinit var studentDataBase: StudentDataBase

    private lateinit var studentDao: StudentDao

    private lateinit var allStudentsLivePaged: LiveData<PagedList<Student>>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        context = this

        initData()

        initEvent()

    }

    private fun initEvent() {
        //添加数据
        binding.buttonPopulate.setOnClickListener {

            //添加1000个学生
            val students = Array(1000) { Student() }
            for (i in 0 until 1000) {
                val student = Student().apply {
                    studentNumber = i + 1
                }
                students[i] = student
            }
            studentViewModel.insertStudent(*students)

        }

        //清除数据
        binding.buttonClear.setOnClickListener {

            studentViewModel.deleteAllStudent()

        }

    }

    private fun initData() {

        studentDataBase = StudentDataBase.getInstance(context)

        studentDao = studentDataBase.getStudentDao()

        studentPagedAdapter = StudentPagedAdapter()

        binding.recyclerView.layoutManager = LinearLayoutManager(context)
        //添加一个分隔符
        binding.recyclerView.addItemDecoration(
            DividerItemDecoration(
                context, DividerItemDecoration.VERTICAL
            )
        )
        binding.recyclerView.adapter = studentPagedAdapter

        //每次加载的数据为两个
        allStudentsLivePaged = LivePagedListBuilder(studentViewModel.getAllStudents(), 2).build()

        allStudentsLivePaged.observe(this) {
            studentPagedAdapter.submitList(it)
        }

    }

}
```

# 三、演示

## 3.1 效果

```kotlin
        //每次加载的数据为两个
        allStudentsLivePaged = LivePagedListBuilder(studentViewModel.getAllStudents(), 2).build()

        allStudentsLivePaged.observe(this) {
            studentPagedAdapter.submitList(it)
        }
```

- 在前面的代码中，设置了每次加载2条数据，为什么要设置这么小呢，因为如果设置太大了，在快速滑动时就很难观察到加载更多这个行为：

<img src="https://vip.helloimg.com/images/2023/11/25/o0SHq0.gif" width="20%">

- 可以明显的看出，并不是一下子把1000条数据全部加载出来的，而是每次都只加载了一部分，在快速滑动时会由于某些数据还没被加载，出现"loading"的提示。

## 3.2 验证

- 我们修改一下代码，直观的观察一下数据的加载过程。首先将每次加载的数据增加到20个，然后在 allStudentsLivePaged 观察数据变化时，为其 PagedList 添加一个弱引用的 Callback 对象：

```kotlin
//修改每次加载的数据为20个
allStudentsLivePaged = LivePagedListBuilder(studentViewModel.getAllStudents(), 20).build()

allStudentsLivePaged.observe(this) {
    studentPagedAdapter.submitList(it)

    Log.e("halo", "总数: ${it.size} , 初始加载个数: ${it.loadedCount}")

    //向 PagedList 中添加一个弱引用的 Callback 对象，观察数据变化
    it.addWeakCallback(null, object : PagedList.Callback() {
        override fun onChanged(position: Int, count: Int) {
            // 数据列表发生变化时的处理逻辑，每次加载更多数据的时候就会回调该方法
            Log.e("halo", "总数: ${it.size} , 当前加载个数: ${it.loadedCount}")
        }

        override fun onInserted(position: Int, count: Int) {
            // 数据插入时的处理逻辑
        }

        override fun onRemoved(position: Int, count: Int) {
            // 数据删除时的处理逻辑
        }
    })

}
```

- 在 onChanged() 方法打印输出数据源的总数以及当前加载的实际个数：

```text
E  总数: 1000 , 初始加载个数: 60
E  总数: 1000 , 当前加载个数: 80
E  总数: 1000 , 当前加载个数: 100
E  总数: 1000 , 当前加载个数: 120
```

- 可以明显看到，在滑动的过程中，PagedList 会逐步从源数据 DataSource 中加载数据，并放置到我们的 PagedList 中，然后在页面中呈现，如果用户慢慢滑动，整个过程会是无感的，因为其在快要逼近当前饱和个数时，又会从源数据中加载更多数据。

## 3.3 配置

- 加载更多是实现了，也能很方便的更改每次加载的个数，但是感觉自由度不够高？比如初始加载多少条数据？还差多少条数据达底部时执行加载？这些其实都可以自定义配置，需要用到`PagedList.Config`类。
- LivePagedListBuilder(studentViewModel.getAllStudents(), 20) 方法第二个参数除了直接填写每次加载的个数，还可以填写 PagedList.Config 也就是我们的配置文件：

```kotlin
//自定义PagedList配置
val config = PagedList.Config.Builder()
    .setPageSize(20) //设置每页加载的数据量。
    .setPrefetchDistance(40) //设置从当前位置开始预取的距离，以优化滑动性能。默认值为 2 * pageSize。
    .setInitialLoadSizeHint(60) //设置初始加载的数量。默认值为 3 * pageSize。
    .build()

//每次加载的数据为两个
allStudentsLivePaged = LivePagedListBuilder(studentViewModel.getAllStudents(), config).build()
```

- setPageSize() 和 setInitialLoadSizeHint() 很好理解，setPrefetchDistance() 直观点说就是距离底部还有多少条数据时开始加载下一页数据。如果把它设置得比较小，哪怕我们 pageSize 是20，在极速滑动时任然会出现条目没来得及加载的情况，所以为了性能考虑，一般不会将其设置得太小。

# 四、总结

<img src="https://z1.ax1x.com/2023/11/25/piwjyXq.jpg">

- 上图就是整个 Paging2 库的工作流程，我们需要关注三个非常重要的类：PageListAdapter、PagedList、DataSource：
  1. `PageListAdapter` 是用于将分页加载的数据显示在 RecyclerView 中的适配器。它是 RecyclerView.Adapter 的子类，可以与 PagedListAdapter 结合使用，以便自动处理分页数据的更新和展示。PageListAdapter 能够检测数据的变化并根据需要进行局部刷新，从而提供更好的性能和用户体验。
  2. `PagedList` 是一个持有分页数据的类，它将数据分割成页面（page），并且在需要时异步地从 DataSource 中加载数据。PagedList 可以与 PagedListAdapter 结合使用，以便在 RecyclerView 中动态展示分页数据。
  3. `DataSource` 是用于提供分页数据的接口，它负责从数据源（如数据库、网络等）中加载数据，并将加载的数据提供给 PagedList。DataSource 可以是 ItemKeyedDataSource、PageKeyedDataSource 或 PositionalDataSource 的实现类，根据不同的数据源类型选择合适的实现类来加载数据。

- **再次说明**：这只是一个 Paging2 的 Demo 示例，里面使用的很多 API 都有了新的替换，并且只涉及到从本地加载数据，用于回顾 Paging 库的初始形态。后面我会再写一篇使用 Paging3 + Retrofit + Coroutines + Flow 来实现从网络上加载数据的文章，同时这套组合也是目前我所接触到最新的加载数据的方式。
- 有任何问题都可以留言告诉我，感谢！