---
title: 'Kotlin 中的空安全设计详解'
date: '2023-07-19'
description: '之前对 Kotlin 的基本语法、变量声明、空安全设计和兼容性做了笼统的介绍，今天对其中的空安全设计做一个详细的剖析。'
cover: 'https://z1.ax1x.com/2023/09/25/pP7y0zt.jpg'
categories:

- 编程开发

tags:

- Kotlin
- 面向对象编程
- 函数式编程

---

# 一、变量

- 在讲空安全设计之前，我们最好是回忆一下 Java 和 Kotlin 中`变量声明和赋值`方面的异同。
- 首先，在 Java 中，声明一个变量的格式是：`<数据类型> <变量名>;`缺一不可，比如：

```java
class Sample {
    int age; // 声明一个整型变量 age
    String name; // 声明一个字符串型变量 name
}
```

- 除了简单声明变量，还可以同时进行初始化，即：`<数据类型> <变量名> = <初始值>;`如：

```java
class Sample {
    int age = 25; // 声明一个整型变量 age，并初始化为 25
    String name = "John"; // 声明一个字符串型变量 name，并初始化为 "John"
}
```

- 然后，在 Kotlin 中，声明一个变量的格式是：`var <变量名>: <数据类型>`，像这样：

```kotlin
    var name: String
```

- 一眼看去和 Java 有几处不同：
   1. 有一个`var`关键字
   2. 类型和变量名位置互换了
   3. 中间是用冒号分隔的
   4. 结尾没有分号（Kotlin 里面不需要分号）

- 虽然看上去只是语法格式有些不同，但如果真这么写，IDE 会报错：`Property must be initialized or be abstract`。什么意思呢，就是说属性需要在声明的同时初始化，除非你把它声明成抽象的。（没错，Kotlin 中属性也可以是抽象的，这点和 Java 也不相同，这里先不理会）
- 彳亍！那就初始化！欸等等，为什么一定先要初始化啊？Java 里面声明变量的时候都不需要初始化啊，是不是你 Kotlin 故意找茬啊？

- 不不不！其实是有原因的，在 Kotlin 中，变量是`没有默认值`的，这点不像 Java，Java 的 field 有默认值（局部变量没有），比如：

```java
class Sample {
    String name; // 默认值是 null
    int count; // 默认值是 0
}
```

- 好吧，你没有默认值那我给你一个吧，我这样写：

```kotlin
    var name: String = null
```

- 哎呀又报错了，IDE 告诉你说：`Null can not be a value of a non-null type View`，也就是说，需要赋一个非空的值给它才行。那怎么办？我们下一节说。

# 二、空安全设计

## 2.1 NullPointerException

- 先介绍一个异常：`NullPointerException` - 空指针异常，这个异常可谓是大家的老熟人了，只要干过开发的一定见过它。如果没见过也没关系，在 Java 中你可以通过下面几行代码轻易复现这个异常：

```java
public class Main {
    public static void main(String[] args) {
        String str = null;
        System.out.println(str.length()); // 触发 NullPointerException
    }
}
```

- 具体来说，这里的 str 也就是 null 表示一个空引用，它并没有指向实际的对象，当我们试图在 str 上调用 length() 方法来获取字符串的长度时，没有实际的字符串对象可供调用方法，所以会抛出 NullPointerException。

## 2.2 空安全设计

- 好的，进入正题。什么是 Kotlin 的空安全设计？它的存在有什么意义？
- 简单来说，空安全设计就是通过 IDE 的提示来避免调用 null 对象，从而`避免 NullPointerException`。
- 可别小看这一点，单单`避免 NullPointerException`这一句话，就含金量十足，要知道 NullPointerException 这个异常不仅常见，并且是致命的，一旦出现并且没有捕获处理那么程序就会直接崩溃。
- 空安全检测其实在 androidx 里就有支持的，用一个注解就可以标记变量是否可能为空，然后 IDE 会帮助检测和提示，我们来看下面这段 Java 代码：

```java
class Sample {
    @NonNull
    View view = null;//IDE 会发出警告：'null' is assigned to a variable that is annotated with @NotNull
}
```

- 到了 Kotlin 这里，就有了语言级别的默认支持，而且提示的级别从 `warning` 变成了 `error`（拒绝编译）：

```kotlin
    var view: View = null //IDE 会提示错误，Null can not be a value of a non-null type View
```

- `在 Kotlin 里面，所有的变量默认都是不允许为空的`，如果你给它赋值 null，就会报错，像上面那样。
- Kotlin 这样做的目的其实是可以理解的，你声明了一个对象，不就是要使用它吗？既然要使用它，那它为空就没有意义了呀。Java 对这方面的限制很宽松，我们已经习惯，但是这并不代表 Java 这样做就是最合理的。
- 这个时候就存在一个问题了，很多时候变量的值真的无法保证空与否，比如你要从服务器取一个 JSON 数据，并把它解析成一个 User 对象：

```kotlin
class User {
    var name: String = null // 这样写会报错，但该变量无法保证空与否
}
```

- 这个时候报错了，但是空值就是有意义的！对于这些可以为空值的变量，你可以在类型右边加一个 ? 号，解除它的非空限制：

```kotlin
class User {
    var name: String? = null
}
```

- 加了问号之后，一个 Kotlin 变量就像 Java 变量一样没有非空的限制，自由自在了。你除了在初始化的时候可以给它赋值为空值，在代码里的任何地方也都可以。
- 这种类型之后加 ? 的写法，在 Kotlin 里叫`可空类型`。不过，当我们使用了可空类型的变量后，会有新的问题：

```kotlin
    var view: View? = null
    view.setBackgroundColor(Color.RED)
    // 这样写会报错，Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type View?
```

- 对于我们定义的`可能为空`的变量，Kotlin 不允许我们用。那怎么办？我们尝试用之前检查一下，但似乎 IDE 不接受这种做法：

```kotlin
    if (view != null) {
        view.setBackgroundColor(Color.RED)
        // 这样写也会报错，Smart cast to 'View' is impossible, because 'view' is a mutable property that could have been changed by this time
    } 
```

- 这个报错的意思是即使你检查了非空也不能保证下面调用的时候就是非空，因为在`多线程`情况下，其他线程可能把它再改成空的。
- 那怎么办？Kotlin 里是这么解决这个问题的呢？它用的不是` . `而是` ?.`：

```kotlin
    view?.setBackgroundColor(Color.RED)
```

- 这个写法同样会对变量做一次非空确认之后再调用方法，这是 Kotlin 的写法，并且它可以做到`线程安全`，因此这种写法叫做`safe call`。
- 另外还有一种双感叹号的用法：

```kotlin
    view!!.setBackgroundColor(Color.RED)
```

- 意思是告诉编译器，我保证这里的 view 一定是非空的，编译器你不要帮我做检查了，有什么后果我自己承担。这种「肯定不会为空」的断言式的调用叫做 `non-null asserted call`。一旦用了非空断言，实际上和 Java 就没什么两样了，但也就享受不到 Kotlin 的空安全设计带来的好处（在编译时做检查，而不是运行时抛异常）了。
- 其实上述内容就是 Kotlin 的空安全设计了，很多人在上手的时候都被变量声明搞懵，原因就是 Kotlin 的空安全设计所导致的这些报错：
  1. 变量需要手动初始化，如果不初始化的话会报错； 
  2. 变量默认非空，所以初始化赋值 null 的话报错，之后再次赋值为 null 也会报错； 
  3. 变量用 ? 设置为可空的时候，使用的时候因为`可能为空`又报错。

- 关于空安全，最重要的是记住一点：所谓`可空不可空`，关注的全都是`使用的时候`，即`这个变量在使用时是否可能为空`。

- Elvis 操作符，它是 Kotlin 中的一种特殊运算符，用于处理可能为空的表达式，并为其提供一个备选的非空值作为默认值。写法：

```kotlin
  expression ?: defaultValue
```

- 如果 expression 不为 null，则 Elvis 操作符的结果为 expression 的值；如果 expression 为 null，则结果为 defaultValue 的值。
- 一些注意事项：
  1. defaultValue 必须与 expression 具有相同的类型或兼容的类型。否则，在编译时就会发生类型不匹配的错误。
  2. defaultValue 可以是一个表达式，可以是常量、变量、函数调用等。
  3. Elvis 操作符可以嵌套使用，形成链式调用。例如：a ?: b ?: c
  4. Elvis 操作符可以与安全调用运算符一起使用，以处理可能为空的对象的属性或方法。例如：object?.property ?: defaultValue
  5. Elvis 操作符是一种简洁的处理可为空变量的方式，但在使用时需要谨慎考虑默认值的选择，以确保逻辑正确性。

- 空安全讲了这么多，但是有些时候我们声明一个变量是不会让它为空的，比如 view，其实在实际场景中我们希望它一直是非空的，可空并没有业务上的实际意义，使用 ?. 影响代码可读性。
- 但如果你在 MainActivity 里这么写：

```kotlin
class MainActivity : AppCompatActivity() {
    var view: View = findViewById(R.id.tvContent)
}
```

- 编译器不会报错，但程序一旦运行起来就 crash 了，原因是 findViewById() 是在 onCreate 之后才能调用（或者说是在 setContentView() 调用后）。
- 那怎么办呢？其实我们很想告诉编译器`我很确定我用的时候绝对不为空，但第一时间我没法给它赋值`。
- Kotlin 给我们提供了一个选项：`延迟初始化`。

## 2.3 延迟初始化

- 为了能声明第一时间没法赋初始值的变量，Kotlin 给我们提供了延迟初始化方式声明变量，具体是这么写的：

```kotlin
    lateinit var view: View
```

- lateinit 的意思是：告诉编译器我没法第一时间就初始化，但我肯定会在使用它之前完成初始化的。
- 它的作用就是让 IDE 不要对这个变量检查初始化和报错。换句话说，加了这个 lateinit 关键字，这个变量的初始化就全靠你自己了，编译器不帮你检查了。
- 然后我们就可以在 onCreate 中进行初始化了：

```kotlin
  lateinit var view: View
  override fun onCreate() {
      //...
      view = findViewById(R.id.tvContent)
  }
```

- 延迟初始化对变量的赋值次数没有限制，你仍然可以在初始化之后再赋其他的值给 view。

## 2.4 类型推断

- 空安全设计到此其实已经差不多讲完了，再补充一点其他内容。
- Kotlin 有个很方便的地方是，如果你在声明的时候就赋值，那不写变量类型也行：

```kotlin
   var name: String = "Mike"//可以直接写成 var name = "Mike"
```

- 这个特性叫做`类型推断`，它跟`动态类型`是不一样的。
  1. 类型推断（Type Inference）：类型推断是指编译器或解释器能够根据上下文推断出表达式的类型，而无需显式地指定类型。这种推断可以减少冗余代码，提高编码效率。
  2. 动态类型（Dynamic Typing）：动态类型是指在运行时确定变量的数据类型。在动态类型语言中，变量的类型是在运行时根据赋值语句来确定的，可以在程序中更改变量的类型。

## 2.5 val 和 var

- 除了前面提到的 var，我们还可以使用 val 来声明变量：

```kotlin
    val age = 18
```

- val 是 Kotlin 在 Java 的`变量`类型之外，又增加的一种变量类型：只读变量。它只能赋值一次，不能修改。而 var 是一种可读可写变量。
- val 和 Java 中的 final 类似，不过其实它们还是有些不一样的，总之直接进行重新赋值是不行的。

# 三、总结

- Kotlin 的空安全机制通过明确可为空和不可为空、编译时空值检查、安全调用运算符、Elvis 操作符和非空断言操作符等特点，提供了一套有效的工具来处理空指针异常，增加代码的稳定性和可靠性。