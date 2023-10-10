---
title: 'Kotlin 扩展函数'
date: '2023-10-09'
description: '简洁优雅的语法糖 - 扩展函数（Extension Functions）'
cover: 'https://z1.ax1x.com/2023/09/26/pP7jdYj.jpg'
categories:

- 编程开发

tags:

- Kotlin
- 面向对象编程
- 函数式编程

---

# 一、概念及使用

## 1.1 概念

- Kotlin 扩展函数是一种特殊类型的函数，它允许我们在`已有的类`上添加新的函数，而`无需修改这些类的源代码`。
- 通过扩展函数，我们可以向现有的类添加更多的行为和功能，使代码更加模块化和可读性更高。

## 1.2 定义和使用

- 在 Kotlin 中，定义扩展函数的语法非常简洁。我们可以`使用 fun 关键字，后跟接收者类型和函数名，然后在函数体内编写逻辑即可`。其中，接收者类型指定了我们要对哪个类进行扩展。
- 下面是一个示例，演示如何在字符串类上定义一个扩展函数 isPalindrome：

```kotlin
fun String.isPalindrome(): Boolean {
    val reversed = this.reversed()
    return this == reversed
}
```

- 上述代码定义了一个名为 isPalindrome 的扩展函数，它接收一个字符串作为接收者，并返回一个布尔值。函数内部使用 reversed() 函数将字符串进行反转，并将反转后的字符串与原始字符串进行比较，判断是否是回文。
- 我们可以在任何字符串对象上调用 isPalindrome 函数，就像调用该类的普通函数一样：

```kotlin
val str1 = "madam"
val str2 = "hello"

println(str1.isPalindrome()) // 输出 true
println(str2.isPalindrome()) // 输出 false
```

- 通过扩展函数，我们为字符串类添加了一个检查回文的功能，且未对 String 类的源代码进行操作，当然我们本身也不能修改标准库的源码。

## 1.3 不同的写法

- 扩展函数写在哪都可以，但写的位置不同，作用域就也不同。所谓作用域就是说你能在哪些地方调用到它。

1. `Top Level Function`
- 最简单的写法就是把它写成 Top Level 也就是顶层的，让它不属于任何类，这样你就能在任何类里使用它。这也和成员函数的作用域很像——哪里能用到这个类，哪里就能用到类里的这个函数：

```kotlin
package com.example

fun String.method1(i: Int) {
  ...
}

...

"hello".method1(1)
```

- 有一点要注意了：这个函数属于谁？属于函数名左边的类吗？并不是的，它是个 Top-level Function，它谁也不属于，或者说它只属于它所在的 package。
- 那它为什么可以被这个类的对象调用呢？——因为它在函数名的左边呀！在 Kotlin 里，当你给声明的函数名左边加上一个类名的时候，表示你要给这个函数限定一个 Receiver——直译的话叫接收者，其实也就是哪个类的对象可以调用这个函数。虽然说你是个 Top-level Function，不属于任何类——确切地说是，不是任何一个类的成员函数——但我要限制只有通过某个类的对象才能调用你。这就是扩展函数的本质。

2. `成员扩展函数`
- 除了写成 Top Level 的，扩展函数也可以写在某个类里：

```kotlin
class Example {

  fun String.method2(i: Int) {
    ...
  }

}
```

- 然后你就可以在这个类里调用这个函数，但必须使用那个前缀类的对象来调用它：

```kotlin
class Example {

  fun String.method2(i: Int) {
    ...
  }

  ...

  "hello".method2(1) // 可以调用

}
```

- 这个函数这么写，它到底是属于谁的呀？属于外部的类还是左边前缀的类？
  属于谁？这个「属于谁」其实有点模糊的，我需要问再明确点：它是谁的成员函数？当然是外部的类的成员函数了，因为它写在它里面嘛，对吧？那函数名左边的是什么？刚才我刚说过，它是这个函数的 Receiver，对吧？也就是谁可以去调用它。所以它既是外部类的成员函数，又是前缀类的扩展函数。
- 这种既是成员函数、又是扩展函数的函数，它们的用法跟 Top Level 的扩展函数一样，只是由于它同时还是成员函数，所以只能在它所属的类里面被调用，到了外面就不能用了：

```kotlin
class Example {

  fun String.method2(i: Int) {
    ...
  }

  ...

  "hello".method2(1) // 可以调用

}

"hello".method2(1) // 类的外部不能调用
```

## 1.4 指向扩展函数的引用

- 我们知道，Kotlin 中函数是可以使用双冒号被指向的：

```kotlin
    Int::toFloat
```

- 需要知道指向的并不是函数本身，而是和函数等价的一个对象，这也是为什么你可以对这个引用调用 invoke()，却不能对函数本身调用：

```kotlin
(Int::toFloat)(1) // 等价于 1.toFloat()
Int::toFloat.invoke(1) // 等价于 1.toFloat()
1.toFloat.invoke() // 报错
```

- 为了简单起见，我们通常可以把这个`指向和函数等价的对象的引用`称作是`指向这个函数的引用`。
- 普通函数可以被指向，扩展函数同样也是可以被指向的：

```kotlin
fun String.method1(i: Int) {

}

...

String::method1
```

- 不过如果这个扩展函数不是 Top-Level 的，也就是说如果它是某个类的成员函数，它就不能被引用了：
  
```kotlin
class Extensions {

  fun String.method1(i: Int) {
    ...
  }

  ...

  String::method1 // 报错
}
```

- 为什么呢？首先，一个普通成员函数是通过什么来引用？类名加双冒号加函数名，扩展函数呢？也是类名加双冒号加函数名，只不过这次是 Receiver 也就是限定的接受者的类名。那成员扩展函数是用类名加双冒号加函数名？谁的类名呢？这是存在歧义的，所以 Kotlin 就干脆不许我们引用既是成员函数又是扩展函数的函数了，一了百了。
- 同样，跟普通函数的引用一样，扩展函数的引用也可以被调用，直接调用或者用 invoke() 都可以，不过要记得把 Receiver 也就是接收者或者说调用者填成第一个参数：

```kotlin
(String::method1)("hello", 1)
String::method1.invoke("hello", 1)

// 以上两句都等价于：
"hello".method1(1)
```

# 二、扩展函数和 Java 动态代理的区别

## 2.1 Java 动态代理

- 扩展函数的作用：`不修改原始类或创建子类的情况下，为现有类添加新的方法或行为。`对 Java 比较熟悉的同学肯定会觉得这话特别耳熟，这和 Java 中动态代理功能的描述简直一摸一样啊？
- 动态代理又是啥呢？在学习 Spring 的底层原理时，必然会接触到动态代理的相关知识，动态代理可以`用于在方法调用前后执行额外的逻辑`，例如日志记录、性能监控、事务管理等。它通常用于面向切面编程（Aspect-Oriented Programming）和代理模式（Proxy Pattern）。
- 以下是一个简单的示例，演示如何使用动态代理为目标对象添加日志记录功能：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 目标接口
interface UserService {
    void save(String data);
}

// 目标类
class UserServiceImpl implements UserService {
    public void save(String data) {
        System.out.println("保存数据：" + data);
    }
}

// 日志记录的代理处理器
class LogInvocationHandler implements InvocationHandler {
    private final Object target; // 目标对象

    public LogInvocationHandler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("开始记录日志");
        Object result = method.invoke(target, args); // 调用目标对象的方法
        System.out.println("结束记录日志");
        return result;
    }
}

// 测试代码
public class Main {
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();
        LogInvocationHandler handler = new LogInvocationHandler(target);
        
        // 创建代理对象
        UserService proxy = (UserService) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            handler
        );
        
        // 使用代理对象调用方法
        proxy.save("Hello World");
    }
}
```

## 2.2 二者的区别

- 让我们分析一下 Kotlin 扩展函数和 Java 动态代理二者的区别：
- 相同点：
  1. 都是为了在不修改原始类的情况下为类添加新的功能或方法（动态代理不能增加为类增加方法，但是可以为某方法进行增强）。
  2. 都可以提高代码的可读性、模块化和重用性。
  3. 都可以通过代理对象或扩展函数来调用新增的方法。
- 不同点：
  1. 语法不同：Java 动态代理使用 `Proxy` 和 `InvocationHandler` 进行实现，而 Kotlin 扩展函数使用关键字 fun 在类外部定义。
  2. 支持类型不同：Java 动态代理主要用于接口或实现了接口的类，无法为 final 类（如 String）创建代理；Kotlin 扩展函数可以为任意类添加方法，包括 final 类。
  3. 调用方式不同：Java 动态代理通过代理对象调用新增方法，但要先判断方法名以确定调用哪个方法；Kotlin 扩展函数直接在原始类的实例上调用新增方法，就像调用该类的普通方法一样。
  4. 编译时检查不同：Java 动态代理在编译时无法检查代理的方法名和参数，只有在运行时才能确定是否调用正确；Kotlin 扩展函数在编译时可以进行类型检查，并确保只调用扩展函数已声明的方法。
  5. 实现方式不同：Java 动态代理基于反射机制实现，会带来一定的性能开销；Kotlin 扩展函数在编译时就将扩展函数与类关联起来，无需反射，在运行时性能更高。

- 可以看出，虽然二者最大的作用点是一样的，即`不修改原始类的情况下为类添加新的功能或方法`，但是在语法、支持类型、调用方式、编译时检查和实现方式等方面都有所不同，可以说二者只是功能相似，但本质完全不同。


