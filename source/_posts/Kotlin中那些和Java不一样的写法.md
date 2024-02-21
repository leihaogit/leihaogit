---
title: 'Kotlin 中那些和 Java 不一样的写法'
date: '2023-07-24'
description: '总结一下 Kotlin 和 Java 在语法、集合数组、流程控制等方面的差异。'
cover: 'https://z1.ax1x.com/2023/09/25/pP7y8sK.png'
categories:

- 编程开发

tags:

- Kotlin
- 面向对象编程
- 函数式编程

---

# 一、语法和类型系统

## 1.1 可空性

- Kotlin 引入了可空性的概念，通过在类型后面添加`?`来表示一个可空的引用类型，使得编译器能够在编译时检查空指针异常。

```kotlin
    var name: String? = null // 可空的字符串
```

- Java 中没有默认的机制来处理空引用，因此更容易出现空指针异常。

```java
    String name = null; // 普通字符串，可能为 null
```

## 1.2 函数声明

- Kotlin 使用`fun`关键字来声明函数，将函数参数的类型放在参数名称后面，而不是像 Java 那样将类型放在参数列表前面。

```kotlin
fun greet(name: String) {
    println("Hello, $name!")
}
```

- Java 中的函数声明方式：

```java
void greet(String name) {
    System.out.println("Hello, " + name + "!");
}
```

## 1.3 数据类

-  Kotlin 提供了数据类的概念，通过简单地声明一个类为 data class，编译器会自动生成一些标准方法，如 equals()、hashCode()、toString() 等。在 Java 中，需要手动实现这些方法。

```kotlin
data class User(val name: String, val age: Int)
```

## 1.4 字符串插值

- Kotlin 使用 $ 加变量名的方式进行字符串插值，而不是 Java 中的字符串拼接或者使用 String.format()。

```kotlin
kotlin
val name = "Alice"
val message = "Hello, $name!"
```

## 1.5 Lambda 表达式

- Kotlin 提供了更简洁的语法来定义 Lambda 表达式，使得函数式编程更加方便。

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val squared = numbers.map { it * it }
```

- Java 中函数式编程需要通过匿名内部类来实现。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> squared = numbers.stream().map(x -> x * x).collect(Collectors.toList());
```

## 1.6 类型推断

- Kotlin 具有更强大的类型推断能力，可以根据上下文自动推断变量的类型，减少了代码中的类型声明。

```kotlin
val number = 42 // 推断为 Int 类型
val doubleNumber = 3.14 // 推断为 Double 类型
```

## 1.7 构造器

- Kotlin 中构造器统一使用 constructor 进行声明，而 Java 中构造器和类同名。

```kotlin
class User {
    val id: Int
    val name: String
    
    constructor(id: Int, name: String) {
        this.id = id
        this.name = name
    }
}
```

```java
public class User {
    int id;
    String name;
    
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

- 还可以注意到：Kotlin 中构造器没有 public 修饰，因为默认可见性就是 public。

## 1.8 init 代码块 

- Kotlin 中的 init 代码块必须加上 init 关键字修饰，Java 中不需要加。

```kotlin
class User {
    
    init {
        // 初始化代码块，先于构造器执行
    }
    
    constructor() {
    }
}
```

```java
public class User {
   
    {
        // 初始化代码块，先于构造器执行
    }
    
    public User() {
    }
}
```

## 1.9 final 和 val

- 大多数情况下，Kotlin 中的 val 可以直接看做 Java 中的final，都表示变量是不可修改的。Kotlin 中 val 有一种比较特殊的用法是可以通过自定义 getter 让变量每次被访问时，返回动态获取的值。

```kotlin
val size: Int
    get() { // 每次获取 size 值时都会执行 items.size
        return items.size
    }
```

## 1.10 static 和 companion object

- kotlin 中将静态变量和静态方法这两个概念完全去除了，取而代之的是`伴生对象（Companion Object）`，在 Kotlin 中，每个类都可以有一个伴生对象，通过 companion object 关键字声明。伴生对象中的属性和函数可以被视为类的静态属性和静态方法，可以通过类名直接访问。

```kotlin
class Sample {
    companion object {
        //str 可以直接通过类名进行访问
        val str = "Hello word"
    }
}
```

- 静态初始化。由于 Java 中的静态变量和方法，在 Kotlin 中都放在了 companion object 中。因此 Java 中的静态初始化在 Kotlin 中自然也是放在 companion object 中的，像类的初始化代码一样，由 init 和一对大括号表示：

```kotlin
class Sample {
       
    companion object {
         
        init {
            //初始化工作
        }
    }
}
```

## 1.11 Object 和 Any

- 在 Java 中，我们都知道 Object 类是所有类的超类，即所有类都继承自 Object，而在 Kotlin 中，这个类变为了`Any`，它定义了一些通用的方法，例如 equals()、hashCode() 和 toString() 等。

```kotlin
class MyClass {
    // ...
}

val myObj = MyClass()
if (myObj is Any) {
    println("myObj is an instance of Any")
}

val hashCode = myObj.hashCode() // 调用 Any 类的方法

```

- object（首字母小写）在 Kotlin 中变为了一个关键字，功能类似于`class`，用于声明一个匿名对象或单例对象。使用 object 关键字创建的对象是唯一的，可以直接访问该对象中定义的属性和方法。

```kotlin
object Singleton {
    val name = "Singleton"
    fun greet() {
        println("Hello, $name!")
    }
}

val singletonName = Singleton.name // 访问对象的属性
Singleton.greet() // 调用对象的方法

```

- 所以在 Kotlin 中创建单例不用像 Java 中那么复杂，只需要把 class 换成 object 就可以了。举个例子：

```java
public class A {
    private static A sInstance;
    
    public static A getInstance() {
        if (sInstance == null) {
            sInstance = new A();
        }
        return sInstance;
    }
}
```

- 上面是 Java 实现一个单例类的方法（非线程安全），而在 Kotlin 中，只需要将 class 替换为 object：

```kotlin
object A {
    val number: Int = 1
    fun method() {
        println("A.method()")
    }
}   
```

- 另外，通过 object 实现的单例是一个`饿汉式`的单例，并且实现了`线程安全`。
- 单例对象说完了，还有个匿名类写法差异：

```java
ViewPager.SimpleOnPageChangeListener listener = new ViewPager.SimpleOnPageChangeListener() {
    @Override 
    public void onPageSelected(int position) {
        // override
    }
};
```

- Kotlin 和 Java 创建匿名类的方式很相似，只不过把 new 换成了 object：

```kotlin
val listener = object: ViewPager.SimpleOnPageChangeListener() {
    override fun onPageSelected(position: Int) {
        // override
    }
}   
```

## 1.12 常量

- 在 Java 中，一般这样声明一个常量：

```java
public class Sample {
    public static final int CONST_NUMBER = 1;
}
```

- 可以注意到使用了`static final`进行修饰，主要目的是确保常量的唯一性、不可修改性和全局访问性。而在 Kotlin 中，自然应该写在`companion object`伴生类中才能实现这个功能：

```kotlin
class Sample {
    companion object {
        const val CONST_NUMBER = 1
    }
}

```

- 可以看出，Kotlin 还新增了修饰常量的 const 关键字。其实除此之外， Kotlin 还有个特殊机制：`top-level 顶层`，指的是不属于任何类或对象的代码块或声明。它是指在文件级别上直接编写的代码，而不是嵌套在类、函数或对象内部。
- 例如，以下代码片段中的 CONST_NUMBER 常量就是一个在顶层声明的常量：

```kotlin
const val CONST_NUMBER = 1

fun main() {
    println(CONST_NUMBER)
}
```

- 在这个示例中，CONST_NUMBER 常量被声明在 main() 函数之外的顶层位置，所以它是一个顶层常量。它可以在该文件的任何地方被访问和使用，包括 main() 函数内部。
- 学过 C++ 的同学可能觉得这个和 C++ 中的 #define 有一点类似，但是其实他们是有很大区别的：
    1. #define 是一个预处理指令，用于定义常量或宏。它是在编译之前进行文本替换的，将标识符替换为预定义的文本。这种方式并不会创建一个真正的符号，而只是简单的文本替换。
    2. 在 Kotlin 中，「top-level 顶层」是一种语言特性，用于在文件级别上直接声明常量、函数和其他类型的声明。这些声明在编译时会被编译器解析，并生成相应的符号。它们是真正的语言成分，可以在整个文件范围内被访问和使用。

## 1.13 可见性修饰符

- Java 中的可见性修饰符有：
  1. public：公共的，作用范围是整个项目。被public修饰的类、方法、属性可以被任何代码访问。
  2. protected：受保护的，作用范围是同一包内以及该类的子类。被protected修饰的方法和属性可以在同一包内的其他类中访问，也可以在该类的子类中访问。
  3. default（默认修饰符）：默认的，作用范围是同一包内。`如果没有明确指定访问修饰符，即采用默认修饰符，则只能在同一包内访问。`
  4. private：私有的，作用范围是仅限于该类内部。被private修饰的方法和属性只能在该类内部访问。

- kotlin 中的可见性修饰符有：
  1. public：公共的，作用范围是整个模块（module）。`默认情况下，所有声明都具有public可见性，可以被任何代码访问`。
  2. internal：内部的，作用范围是同一模块内。被internal修饰的声明可以在同一模块内的任何位置被访问。
  3. protected：受保护的，作用范围是同一类内或者子类。在Kotlin中，protected仅适用于类成员，不适用于顶级声明。
  4. private：私有的，作用范围是仅限于该类或文件内部。被private修饰的声明只能在声明它的类或文件内部访问。

- 可以看出，Kotlin 相比 Java 少了一个 default 「包内可见」修饰符，多了一个 internal「module 内可见」修饰符。
- 特别的，需要说明一下两者 private 修饰符的区别：
  1. Java 中的 private 表示`类中可见`，作为内部类时对外部类`可见`。
  2. Kotlin 中的 private 表示`类中或所在文件内可见`，作为内部类时对外部类`不可见`。

# 二、数组和集合

## 2.1 数组

- 在 Java 中声明一个 String 数组：

```java
    String[] strs = {"a", "b", "c"};
```

- Kotlin 中声明一个 String 数组：

```kotlin
val strs: Array<String> = arrayOf("a", "b", "c")
```

- Kotlin 中的数组是一个拥有泛型的类，创建函数也是泛型函数，和集合数据类型一样。将数组进行泛型化有什么好处？他可以让对数组的操作像集合一样功能更强大，由于泛型化，Kotlin 可以给数组增加很多有用的工具函数：
    1. get() / set()  获取/修改指定索引位置上的元素值。
    2. contains()  检查数组中是否包含指定元素。
    3. first()  获取数组的第一个元素。
    4. find()  根据给定的条件查找数组中的第一个匹配元素。

## 2.2 集合

- Java 中创建一个列表集合，需要一个个的添加元素：

```java
  List<String> strList = new ArrayList<>();
  strList.add("a");
  strList.add("b");
  strList.add("c");
```

- Kotlin 中创建一个列表集合有点像创建一个数组，代码非常简单：

```kotlin
    val strList = listOf("a", "b", "c")
```

- Kotlin 中创建相同的 Set：

```java
  Set<String> strSet = new HashSet<>();
  strSet.add("a");
  strSet.add("b");
  strSet.add("c");
```

- Kotlin 中创建相同的 Set 集合：

```kotlin
val strSet = setOf("a", "b", "c")
```

- Java 中创建一个 Map 集合：

```java
  Map<String, Integer> map = new HashMap<>();
  map.put("key1", 1);
  map.put("key2", 2);
  map.put("key3", 3);
  map.put("key4", 3);
```

- Kotlin 中创建一个同样的 Map 集合：

```kotlin
val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 3)
```

- 简洁、优雅。

# 三、流程控制

## 3.1 if else

- 就 if 语句来说，Java 和 Kotlin 的用法非常相似，都是使用关键字 if、else if 和 else 来进行条件判断。
- 但是，在 kotlin 中，if 语句还能是一个表达式，也就是说它可以返回一个值，Java 中的 if 语句就只是语句，不能作为表达式使用：

```kotlin
  val a = 5
  val b = if (a > 0) {
      10
  } else {
      -10
  }
```

## 3.2 for 循环

- Kotlin引入了区间迭代的概念。我们可以使用..操作符定义一个范围，并在for循环中使用这个范围进行迭代。

```kotlin
for (i in 1..5) {
    println(i)
}
```
- 同样的功能在 Java 中需要这么写：

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

- 特别的，我们说一下遍历 Map 的操作，在 Java 中，遍历 Map 一般会使用 entrySet() 方法获取键值对的集合，然后使用 for-each 循环进行迭代：

```java
  Map<String, Integer> map = new HashMap<>();
  map.put("A", 1);
  map.put("B", 2);
  for (Map.Entry<String, Integer> entry : map.entrySet()) {
      System.out.println(entry.getKey() + ": " + entry.getValue());
  }
```

- 而在 Kotlin 中，可以直接使用for循环来遍历Map，无需额外的方法调用：

```kotlin
  val map = mapOf("A" to 1, "B" to 2)
  for ((key, value) in map) {
      println("$key: $value")
  }
```

## 3.3 switch case

- Java 中，switch case 需要使用关键字 switch、case、break 和 default：

```java
int day = 3;
        String dayName;
        switch (day) {
        case 1:
        dayName = "Monday";
        break;
        case 2:
        dayName = "Tuesday";
        break;
        case 3:
        dayName = "Wednesday";
        break;
        default:
        dayName = "Invalid day";
        }
        System.out.println(dayName);
```

- 而在 Kotlin 中，则使用了 when else 关键字，使用箭头操作符（->）将条件和结果连接起来，并且不需要显式的 break 语句：

```kotlin
val day = 3
val dayName = when (day) {
    1 -> "Monday"
    2 -> "Tuesday"
    3 -> "Wednesday"
    else -> "Invalid day"
}
println(dayName)
```

- 还是那么的简洁、优雅。

# 四、总结

- 上面举出的只是一些 Java 和 Kotlin 常见的不同之处，实际的差异不是短短一篇文章能说完的。
- 反正总的来说，kotlin 相比 Java 语法更简洁，减少了很多样板代码，提供了更简单的语法糖和函数式编程特性，使代码更易读、更易写。