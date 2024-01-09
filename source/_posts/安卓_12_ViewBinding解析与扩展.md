---
title: 'ViewBinding解析与扩展'
date: '2024-01-09'
description: 'ViewBinding进阶学习，自定义 by viewBinding() 扩展实现'
cover: 'https://s11.ax1x.com/2024/01/09/pFpnpV0.png'
categories:

- 编程开发

tags:

- Kotlin
- Android

--- 

> 尽管 Jetpack Compose 的声明式 UI 开发框架正在快速发展并引起越来越多的关注，但目前市面上主流开发依然使用的是传统的 XML 布局，视图绑定依然是安卓开发们绕不开的话题。

# 一、介绍与使用

## 1.1 概述

`视图绑定（ViewBinding）`是一种在安卓开发中用于访问布局文件中视图的机制，使用非常简单，只需要几行代码就可以了。如果只是单纯介绍 ViewBinding 的使用，是完全没必要写一篇博客专门来介绍的，所以这篇文章会更深入的剖析一下 ViewBinding 实现原理，强调一些使用中需要注意的地方，以及自己实现一个`by viewModels()`类似的视图绑定扩展`by viewBinding()`。

已经会使用的同学可以直接跳至第二章。

## 1.2 使用

在 Android Studio 3.6 Canary 11 及更高版本中，使用 ViewBinding 无需引入任何依赖，直接在 app 的 build.gradle 文件中开启即可：

```kotlin
android {
    /* ... */

    buildFeatures {
        viewBinding = true
    }
    
    /* ... */
}
```

重新构建完成，接下来，你就可以在 Activity、Fragment 或者 Adapter 中使用 ViewBinding 自动帮我们生成的`绑定类`了。例如你有一个 activity_main.xml 布局文件，那么在 MainActivity 中，你可以这样进行视图绑定：

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding:ActivityMainBinding;

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.textView.text = "测试"
    }
}
```

为某个模块启用视图绑定功能后，系统会为该模块中包含的每个 XML 布局文件生成一个绑定类。每个绑定类均包含对根视图以及具有 ID 的所有视图的引用。系统会通过以下方式生成绑定类的名称：将 XML 文件的名称转换为驼峰式大小写，并在末尾添加“Binding”一词。
比如 activity_main.xml 布局文件，会生成一个对应的绑定类`ActivityMainBinding`，路径为：build/generated/data_binding_base_class_source_out/debug/out/com/xxx/yyy/databinding/ActivityMainBinding.java

## 1.3 注意事项

在 Fragment 中使用时，由于 Fragment 的存在时间比其视图长，所以`需要在 Fragment 的 onDestroyView() 方法中清除对绑定类实例的所有引用`。
这样做的目的是为了防止`内存泄漏`，具体而言，绑定类实例通常会持有对布局文件中各个视图的引用，以便我们可以直接访问它们。而在 Fragment 中，由于 Fragment 的生命周期与其视图的生命周期不完全一致，可能会出现视图被销毁但 Fragment 实例仍然存在的情况。所以我们必须要在销毁视图的同时，清除绑定类实例对视图的引用，确保 Fragment 的视图和绑定类实例的生命周期保持一致。

示例（使用 inflate 方法）：

```kotlin
    private var _binding: ResultProfileBinding? = null

    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        val view = binding.root
        return view
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
    
```

- 示例（使用 bind 方法）：

```kotlin
    private var fragmentBlankBinding: FragmentBlankBinding? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val binding = FragmentBlankBinding.bind(view)
        fragmentBlankBinding = binding
        binding.textViewFragment.text = getString(string.hello_from_vb_bindfragment)
    }

    override fun onDestroyView() {
        fragmentBlankBinding = null
        super.onDestroyView()
    }
```

## 1.4 优缺点

- 与 findViewById 相比：
    1. **Null 安全**：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null 指针异常的风险。
    2. **类型安全**：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型。这意味着不存在发生类转换异常的风险。

- 与 DataBinding 相比：
    1. **更快的编译速度**：视图绑定不需要处理注释，因此编译时间更短。
    2. **更易于使用**：视图绑定不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用视图绑定后，它会自动应用于该模块的所有布局。
    3. **不支持**布局变量或布局表达式，**不支持**双向数据绑定。

# 二、原理解析

我们首先在 activity_main.xml 中添加一个 TextView 和 ImageView，根布局使用默认的 ConstraintLayout 即可，方便后续观察。 
使用快捷键访问 ActivityMainBinding 类会进入到其对应的 xml 布局中，因此我们直接到上面提到的路径那里去查看绑定类即可：

```java
public final class ActivityMainBinding implements ViewBinding {
    @NonNull
    private final ConstraintLayout rootView;

    @NonNull
    public final ImageView imageView;

    @NonNull
    public final TextView textView;

    private ActivityMainBinding(@NonNull ConstraintLayout rootView, @NonNull ImageView imageView,
                                @NonNull TextView textView) {
        this.rootView = rootView;
        this.imageView = imageView;
        this.textView = textView;
    }

    @Override
    @NonNull
    public ConstraintLayout getRoot() {
        return rootView;
    }

    @NonNull
    public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater) {
        return inflate(inflater, null, false);
    }

    @NonNull
    public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
                                              @Nullable ViewGroup parent, boolean attachToParent) {
        View root = inflater.inflate(R.layout.activity_main, parent, false);
        if (attachToParent) {
            parent.addView(root);
        }
        return bind(root);
    }

    @NonNull
    public static ActivityMainBinding bind(@NonNull View rootView) {
        // The body of this method is generated in a way you would not otherwise write.
        // This is done to optimize the compiled bytecode for size and performance.
        int id;
        missingId: {
            id = R.id.imageView;
            ImageView imageView = ViewBindings.findChildViewById(rootView, id);
            if (imageView == null) {
                break missingId;
            }

            id = R.id.textView;
            TextView textView = ViewBindings.findChildViewById(rootView, id);
            if (textView == null) {
                break missingId;
            }

            return new ActivityMainBinding((ConstraintLayout) rootView, imageView, textView);
        }
        String missingId = rootView.getResources().getResourceName(id);
        throw new NullPointerException("Missing required view with ID: ".concat(missingId));
    }
}
```

其中 ViewBinding 是一个接口，且只有一个方法 getRoot():

```java
/** A type which binds the views in a layout XML to fields. */
public interface ViewBinding {
    /**
     * Returns the outermost {@link View} in the associated layout file. If this binding is for a
     * {@code <merge>} layout, this will return the first view inside of the merge tag.
     */
    @NonNull
    View getRoot();
}
```

- 首先，构造函数是`私有`的，也就是不允许外部创建类实例。
- 然后是提供了几个静态方法，用于创建类实例。其中
  1. inflate() 方法用于通过传入一个 LayoutInflater 对象来实例化 ActivityMainBinding。该方法内部会使用 LayoutInflater 对象加载指定的布局文件，并返回一个 View 对象。然后，通过调用`bind()`方法，将该 View 对象传入并创建 ActivityMainBinding 实例。
  2. bind() 方法用于传入一个 View 对象来创建 ActivityMainBinding 实例。在方法内部，会将该 View 对象强制转换为 ConstraintLayout 类型（因为根布局是 ConstraintLayout），并将其作为参数传入私有构造函数，创建 ActivityMainBinding 实例。
- 在`bind()`方法中会执行最后的视图查找和绑定工作，能清楚的看到使用的方法为`ViewBindings.findChildViewById(rootView, id)`，可以大胆猜测其内部一定是通过`findViewById()`进一步实现view的查找和绑定，点进去看，果然：

```java
public class ViewBindings {

    private ViewBindings() {
    }

    /**
     * Like `findViewById` but skips the view itself.
     *
     * @hide
     */
    @Nullable
    public static <T extends View> T findChildViewById(View rootView, @IdRes int id) {
        if (!(rootView instanceof ViewGroup)) {
            return null;
        }
        final ViewGroup rootViewGroup = (ViewGroup) rootView;
        final int childCount = rootViewGroup.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final T view = rootViewGroup.getChildAt(i).findViewById(id);
            if (view != null) {
                return view;
            }
        }
        return null;
    }
}
```

简而言之，ViewBinding 内部绑定视图就是通过`findViewById()`实现的，不过整个绑定过程系统都已经帮我们完成，我们只需要拿到对应的 ViewBinding 实例，就可以访问每一个拥有 id 的 view 了。

# 三、优化扩展

在第一节中提到了如何在 Activity 和 Fragment 中使用 ViewBinding，但是直接那样写会存在不少的重复代码，在不同的界面中创建的流程是一样的，仅仅是 ViewBinding 对象的区别。
我们有没有可能仿照`by viewModels()`的扩展那样自己写一个`by viewBinding()`的扩展呢？
注：`by viewModels()`是指在引入`activity-ktx`或者`fragment-ktx`依赖后，在 Activity 或者 Fragment 中直接使用：

```kotlin
private val viewModel: VM by viewModels()
```
从而快捷创建 viewModel 实例的扩展方法。

## 3.1 activity 扩展

```kotlin
@MainThread
inline fun <reified VB : ViewBinding> Activity.viewBinding() = object : Lazy<VB> {
    private var binding: VB? = null
    override val value: VB
        get() = binding ?: VB::class.java.getMethod(
            "inflate",
            LayoutInflater::class.java,
        ).invoke(null, layoutInflater).let {
            if (it is VB) {
                binding = it
                it
            } else {
                throw ClassCastException()
            }
        }

    override fun isInitialized(): Boolean = binding != null
}
```

说明：
1. `@MainThread`表示该扩展函数只能在主线程调用。
2. 该扩展函数返回了一个实现了`Lazy<T>`接口的匿名类，其中`T`是具体的`VB`类型。通过实现`Lazy<T>`接口，可以实现懒加载的效果，只有在真正需要使用 VB 实例时才会进行初始化。
3. `inline`关键字表示内联，它告诉编译器将函数的代码直接插入到调用它的位置，可以减少函数调用的开销。`reified`关键字用于修饰内联函数中的类型参数，使函数内部可以访问类型参数的具体类型。在这个例子中，VB 是一个内联函数的类型参数，并且可以在函数内部以具体类型的方式使用。
4. `getMethod()`方法是通过反射获取指定类中的指定方法。在这个例子中，通过`VB::class.java.getMethod("inflate", LayoutInflater::class.java)`获取了名为`"inflate"`的方法，并且该方法接受一个`LayoutInflater`参数。
5. `invoke()`方法是通过反射来调用方法。在这个例子中，通过`VB::class.java.getMethod("inflate", LayoutInflater::class.java).invoke(null, layoutInflater)`调用了获取到的`inflate()`方法，并传递了一个`LayoutInflater`对象作为参数（确保 binding 在 onCreate() 之后再使用，否则拿不到 LayoutInflater 对象）。由于`inflate()`方法通常是静态方法，因此传递了`null`作为调用者对象。

简而言之，这个扩展函数返回一个实现了`Lazy<T>`接口的匿名类，通过内联和反射调用指定类的方法，实现了懒加载效果。
具体到上面第一节的例子中，就是调用了下面这个方法：

```kotlin
    @NonNull
    public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater) {
        return inflate(inflater, null, false);
    }
```

接下来，在 Activity 中就可以直接使用 by viewBinding() 实现 ViewBinding 实例的创建。

```kotlin
class MainActivity : AppCompatActivity() {

    private val binding:ActivityMainBinding by viewBinding()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        binding.textView.text = "测试"
    }
}
```

## 3.2 Fragment 扩展

```kotlin
@MainThread
inline fun <reified VB : ViewBinding> Fragment.viewBinding() = object : Lazy<VB> {
    private var binding: VB? = null
    override val value: VB
        get() = binding ?: VB::class.java.getMethod(
            "inflate",
             LayoutInflater::class.java,
        ).invoke(null, requireActivity().layoutInflater).let {
            if (it is VB) {
                viewLifecycleOwner.lifecycle.addObserver(object : LifecycleEventObserver {
                    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                        if (event == Lifecycle.Event.ON_DESTROY) {
                            binding = null
                        }
                    }
                })
                binding = it
                it
            } else {
                throw ClassCastException()
            }
        }

    override fun isInitialized(): Boolean = binding != null
}
```

相同的地方就不再说明了，唯一的不同是添加了一个生命周期的监听，在适当的时候将 binding 置 null，避免内存泄露。
使用也是一样简单，例如我有一个 HomeFragment：

```kotlin
class HomeFragment : Fragment() {
    private val binding: FragmentHomeBinding by viewBinding()
  
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View {
        return binding.root
    }
}
```

短短几行就完成了视图绑定工作，大大减少了样板代码，让项目更加简洁优雅。