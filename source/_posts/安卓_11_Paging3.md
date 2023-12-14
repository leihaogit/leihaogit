---
title: '使用 Paging3 实现按需加载功能'
date: '2023-12-14'
description: '使用 Paging3 + Retrofit + Coroutines + Flow 实现按需加载功能'
cover: 'https://s11.ax1x.com/2023/12/14/pifxxQs.jpg'
categories:

- 编程开发

tags:

- Kotlin
- Android 
- Jetpack

--- 

# 一、概述

## 1.1 前言

- 在上一篇文章[使用 Paging2 实现按需加载功能](https://www.leihao168.top/2023/11/25/%E5%AE%89%E5%8D%93_10_Paging2/)中，介绍了如何使用 paging2 + Room 从本地获取数据，今天就是用最新的 Paging3 来进行一个网络数据的分页加载。
- Demo 使用到的技术包含 Paging3、Retrofit、Coroutines、Flow 等，有不熟悉的同学可以先去了解一下，这里不再对其他几个库做详细说明。

## 1.2 与 Paging2 的区别

- Paging3 对比 Paging2 几乎可以称为完全不兼容的跨代升级，前面说过 Paging2 有几个非常重要的类：`PageListAdapter`、`PagedList`、`DataSource`，分别用于`绑定和展示分页数据`、`持有分页数据`以及`提供分页数据`。而在 Paging3 中，变成了下面三者： 
  1. `PagingDataAdapter`：取代了 PagedList，支持增量更新，可以根据新加载的数据自动更新列表，并提供了更好的性能和动画效果。
  2. `PagingData`：取代了 PagedList，是一个流（Flow）类型，与 PagedList 不同，PagingData 可以直接与 Kotlin 流 API 进行集成，在数据变化时提供更强大的操作和转换功能。
  3. `PagingSource`：取代了 DataSource，用于定义如何从数据源加载分页数据，且不再区分 ItemKeyedDataSource、PageKeyedDataSource 或 PositionalDataSource，而是提供了一种通用的方式来加载数据。这也是为什么上篇文章我没有使用 Paging2 来加载网络中的数据，毕竟已经过时了嘛，所以认识一下就好不需要深入了解。

# 二、使用

- 这次的 Demo 是使用 Github 官方提供的 api 获取 Android 相关的开源库，按 Star 由多到少排名，且加载为分页加载。
- Demo 是参考郭霖大神的文章[Jetpack新成员，Paging3从吐槽到真香](https://blog.csdn.net/guolin_blog/article/details/114707250)后自己进行实践搭建的。

## 2.1 引入依赖

```groovy

plugins {
  id("com.android.application")
  id("org.jetbrains.kotlin.android")
  id("kotlin-kapt")
}

dependencies {

  implementation("androidx.room:room-paging:2.5.1") // Room 的分页支持依赖
  kapt("androidx.room:room-compiler:2.5.1") // Room 的注解处理器，用于生成部分代码
  implementation("androidx.room:room-runtime:2.5.1") // Room 运行时库
  implementation("androidx.paging:paging-runtime-ktx:3.1.1") // 分页库，提供了对分页数据加载的支持
  
    /** ... **/
}
```

## 2.2 结构

<img src="https://s11.ax1x.com/2023/12/14/pih97cQ.png">

## 2.3 编码

### 2.3.1 BaseResp

- 基本的响应类，用于包装从服务器端获取的开源库数据

```kotlin
class BaseResp {
    @SerializedName("items")
    val items: List<GitRepo> = emptyList()
}
```

### 2.3.2 GitRepo

- 每一个开源库数据对应的实体类，为了简单易读，去掉了很多不需要的字段

```kotlin
data class GitRepo(
    @SerializedName("id") val id: Int,//开源库的id
    @SerializedName("name") val name: String,//开源库的名称
    @SerializedName("description") val description: String?,//开源库的描述
    @SerializedName("stargazers_count") val starCount: Int//开源库的star数量
)
```
### 2.3.3 Retrofit

- 网络请求相关的工具类和接口配置

```kotlin
object RetrofitClient {

    private const val BASE_URL = "https://api.github.com/"

    // 创建自定义的 OkHttpClient 实例
    private val customOkHttpClient =
        OkHttpClient.Builder().connectTimeout(10, TimeUnit.SECONDS) // 设置连接超时时间为 10 秒
            .readTimeout(30, TimeUnit.SECONDS) // 设置读取超时时间为 30 秒
            .writeTimeout(30, TimeUnit.SECONDS) // 设置写入超时时间为 30 秒
            .build()


    // 创建 Retrofit 实例
    private val retrofit: Retrofit by lazy {
        Retrofit.Builder().baseUrl(BASE_URL) // 设置基础 URL
            .client(customOkHttpClient) // 使用自定义的 OkHttpClient
            .addConverterFactory(GsonConverterFactory.create()) // 添加 Gson 转换器工厂
            .build()
    }

    /**
     * 创建指定类型的服务接口实例
     *
     * @param serviceClass 服务接口的类对象
     * @return 服务接口实例
     */
    fun <T> getService(serviceClass: Class<T>): T {
        return retrofit.create(serviceClass)
    }

}

interface ServicesConfig {
    @GET("search/repositories?sort=stars&q=Android")
    suspend fun getGitRepos(
        @Query("page") page: Int, @Query("per_page") perPage: Int
    ): BaseResp
}
```

### 2.3.4 GitRepoPagingSource

- 负责从数据源加载分页数据，继承自 PagingSource<Key, T>，实现 load() 方法，该方法根据传入的 LoadParams 参数来加载分页数据，并返回`LoadResult<Key, T>`对象。LoadResult 是一个包含加载结果和状态的 sealed class，它可以是 LoadResult.Page（成功加载一页数据）、LoadResult.Error（加载错误）或 LoadResult.EndOfPagination（已加载完所有数据）。
- `LoadParams`包含了请求的页数、加载大小以及初始加载或刷新操作的信息，根据 LoadParams 提供的信息，可以方便地进行网络请求，获取相应页码的数据，并根据加载情况返回对应的 LoadResult。

```kotlin
class GitRepoPagingSource(private val servicesConfig: ServicesConfig) :
    PagingSource<Int, GitRepo>() {//PagingSource<Int, GitRepo>()接收两个泛型参数，前一个表示页数的数据类型，后一个表示每一项数据的类型

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, GitRepo> {
        return try {
            // 获取当前页码，如果为空则默认为1
            val page = params.key ?: 1
            // 获取每页加载的数据大小
            val pageSize = params.loadSize
            // 调用服务配置中的getGitRepos方法获取Git仓库数据
            val repoResponse = servicesConfig.getGitRepos(page, pageSize)
            // 取出其中的List<GitRepo>对象
            val repoItems = repoResponse.items
            // 如果当前页码大于1，则上一页的页码为当前页码减1，否则为null
            val prevKey = if (page > 1) page - 1 else null
            // 如果获取到的仓库数据不为空，则下一页的页码为当前页码加1，否则为null
            val nextKey = if (repoItems.isNotEmpty()) page + 1 else null
            // 将仓库数据、上一页页码和下一页页码封装成LoadResult.Page对象并返回
            LoadResult.Page(repoItems, prevKey, nextKey)
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    // 方法用于获取刷新数据时所需的键值。这里返回 null，表示不支持刷新操作。
    override fun getRefreshKey(state: PagingState<Int, GitRepo>): Int? = null

}
```

### 2.3.5 adapter

- GitRepoAdapter 负责加载和展示分页数据，并通过监听滚动事件和 LoadState 实现自动加载更多的功能。
- GitRepoFooterAdapter 即底部的加载提示，可以随意自定义样式。

```kotlin
class GitRepoAdapter : PagingDataAdapter<GitRepo, GitRepoAdapter.MyViewHolder>(diffCallback) {

    companion object {
        private val diffCallback = object : DiffUtil.ItemCallback<GitRepo>() {
            override fun areItemsTheSame(oldItem: GitRepo, newItem: GitRepo): Boolean {
                return oldItem.id == newItem.id
            }

            override fun areContentsTheSame(oldItem: GitRepo, newItem: GitRepo): Boolean {
                return oldItem == newItem
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val binding =
            CellGitRepoItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return MyViewHolder(binding)
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val gitRepo = getItem(position)

        when (val binding = holder.binding) {
            is CellGitRepoItemBinding -> {
                if (gitRepo != null) {
                    binding.textViewName.text = gitRepo.name
                    binding.textViewDescription.text = gitRepo.description
                    binding.textViewStarCount.text = gitRepo.starCount.toString()
                }
            }
        }
    }

    class MyViewHolder(val binding: ViewBinding) : RecyclerView.ViewHolder(binding.root)

}

class GitRepoFooterAdapter(val retry: () -> Unit) :
    LoadStateAdapter<GitRepoFooterAdapter.MyViewHolder>() {

    class MyViewHolder(val binding: ViewBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(
        parent: ViewGroup, loadState: LoadState
    ): MyViewHolder {
        val binding =
            CellFooterGitRepoItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        binding.buttonRetry.setOnClickListener { retry() }
        return MyViewHolder(binding)
    }

    override fun onBindViewHolder(holder: MyViewHolder, loadState: LoadState) {

        when (val binding = holder.binding) {
            is CellFooterGitRepoItemBinding -> {
                when (loadState) {
                    is LoadState.Loading -> {
                        binding.buttonRetry.isVisible = false
                        binding.progressBar.isVisible = true
                    }

                    else -> {
                        binding.buttonRetry.isVisible = true
                        binding.progressBar.isVisible = false
                    }
                }
            }
        }
    }
}
```

### 2.3.6 GitRepository

- 值得注意的类有两个：
  1. `Flow`：Kotlin 中提供的协程流（Flow）API，可以用来处理异步数据流。
  2. `Pager`：用于创建分页数据源的工具类，接收两个参数，config：一个 PagingConfig 对象，表示分页配置信息，如每页加载数量等；pagingSourceFactory：一个函数，用于创建分页数据源。Pager 还实例提供了一个 flow 属性，它返回一个 Flow<PagingData<T>> 对象，表示一个包含分页数据的异步数据流。
- Flow 的话，它是 Kotlin 协程库中提供的一种用于处理异步数据流的工具，可以用于在异步场景下处理连续的、可能无限的数据集合。简单来说，Flow 可以看作是一种类似于集合的数据结构，但不同于集合的是，它不会一次性返回所有的数据，而是以异步的方式逐个或批量地`发射`数据，我们只需要在对应的地方接收（collect）即可。
- 最后，通过 getGitRepoPagingData() 方法，返回一个 Flow<PagingData<T>> 对象。

```kotlin
class GitRepository {

    companion object {
        private const val PAGE_SIZE = 20
    }

    /**
     * 获取开源库的PagingData
     * @return Flow<PagingData<GitRepo>>，包含分页加载开源库数据的相关配置信息和数据源
     */
    fun getGitRepoPagingData(): Flow<PagingData<GitRepo>> {
        return Pager(config = PagingConfig(PAGE_SIZE),//每页数据的大小
            pagingSourceFactory = {
                GitRepoPagingSource(RetrofitClient.getService(ServicesConfig::class.java))
            })//指定了分页数据的数据源
            .flow
    }

}
```

### 2.3.7 GitRepoViewModel

- 连接界面和数据的中间层，负责处理业务逻辑和分页数据的管理

```kotlin
class GitRepoViewModel : ViewModel() {

    private val gitRepository by lazy { GitRepository() }

    fun getGitRepoPagingData(): Flow<PagingData<GitRepo>> {
        return gitRepository.getGitRepoPagingData().cachedIn(viewModelScope)
    }

}
```

### 2.3.8 Paging3Activity

- 解释几个点：
  1. `addLoadStateListener`：PagingDataAdapter 提供的函数，用于监听加载状态的变化，我们可以在这里判断数据的获取状态，进而进行 UI 方面的更新提示操作。
  2. `refresh()`：也是 PagingDataAdapter 提供的函数，用于刷新数据。
  3. `withLoadStateFooter()`：PagingDataAdapter 的一个扩展函数，用于为适配器添加加载状态的底部视图，这里我们将自己定义的 GitRepoFooterAdapter 放进去。
  4. `collect`：前面提到 Flow 的时候说过接收数据怎么接收，就是使用这里的`collect`函数，实现从数据流中收集并处理分页数据。还要注意 collect 是一个`挂起函数`，它会暂停当前协程，并等待数据流的发射，所以在调用 collect 之前，需要确保代码运行在协程作用域内，比如这里就使用了`lifecycleScope.launch { ... }`来开启一个协程。

```kotlin
class Paging3Activity : AppCompatActivity() {

    private val binding: ActivityPaging3Binding by lazy {
        ActivityPaging3Binding.inflate(layoutInflater)
    }

    private val gitRepoViewModel: GitRepoViewModel by viewModels()

    private lateinit var context: Context

    private lateinit var gitRepoAdapter: GitRepoAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        context = this

        initData()

        initEvent()

    }

    private fun initEvent() {
        gitRepoAdapter.addLoadStateListener {
            when (it.refresh) {
                //加载完毕
                is LoadState.NotLoading -> {
                    Log.e("halo", "LoadState.NotLoading")
                    binding.swipeRefreshLayout.isRefreshing = false
                    binding.lottieAnimationViewLoading.visibility = View.GONE
                }

                //加载中
                is LoadState.Loading -> {
                    Log.e("halo", "LoadState.Loading")
                }

                //加载失败
                is LoadState.Error -> {
                    binding.swipeRefreshLayout.isRefreshing = false
                    Log.e("halo", "LoadState.Error")
                    Toast.makeText(context, "加载失败", Toast.LENGTH_SHORT).show()
                }
            }
        }

        //下拉刷新
        binding.swipeRefreshLayout.setOnRefreshListener {
            gitRepoAdapter.refresh()
        }

    }

    private fun initData() {

        gitRepoAdapter = GitRepoAdapter()

        binding.recyclerView.layoutManager = LinearLayoutManager(context)
        //添加一个分隔符
        binding.recyclerView.addItemDecoration(
            DividerItemDecoration(
                context, DividerItemDecoration.VERTICAL
            )
        )

        binding.recyclerView.adapter =
            gitRepoAdapter.withLoadStateFooter(GitRepoFooterAdapter { gitRepoAdapter.retry() })

        lifecycleScope.launch {
            gitRepoViewModel.getGitRepoPagingData().collect { pagingData ->
                gitRepoAdapter.submitData(pagingData)
            }
        }

    }

}
```

# 四、总结

- 这里就不再做什么总结了，相关的内容都在代码中有所解释。
- 再看一下整个工作流程：

<img src="https://s11.ax1x.com/2023/12/14/pifxxQs.jpg">

- 另外，上面没有提到本地数据的加载，其实非常简单，因为不需要自己编写 PagingSource 了，我们可以直接在 Dao 中定义获取数据的方法返回类型为`PagingData<Int, T>`，后续操作就和上面获取到网络数据一样了。说白了，数据无论从哪儿来，只要获取到了，就被视为数据源，后续都做同样的处理。
- 个人的使用感受还是挺不错的，对于官方所说的功能强大且易于使用，我觉得功能强大确实是强大，易于使用的话只能说还得多练才行呐！而且还有一些高级的用法并没有提及到，比如 RemoteMediator、getRefreshKey()等，这些等到后面体验了之后再来分享。
- 有任何问题都可以留言告诉我，感谢！