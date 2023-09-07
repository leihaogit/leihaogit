---
title: 'OrmLite 框架的简单使用'
date: '2023-04-19'
description: '介绍如何使用OrmLite框架对SQLite数据库进行管理'
cover: 'https://www.helloimg.com/images/2023/07/19/oA2z41.webp'
categories:

- 数据库

tags:

- Android
- Java
- 数据库
- SQLite

---

# 一、OrmLite和SQLite简介

## 1.1 ORM

- 首先我们需要了解一个概念，什么是 ORM（Object Relational Mapping，对象关系映射）？
  一种编程技术或工具，它可以将面向对象的编程语言中的对象和关系型数据库中的数据表之间的映射关系定义为元数据（XML、注解等形式），并且能够在程序运行时自动地将对象转化为关系型数据，或者将关系型数据转换为对象，以此来实现程序员所需的数据访问。
    - 有过 MyBatis 经验的小伙伴对"将映射关系定义为元数据（XML、注解等形式"这种表述肯定不会陌生，在 MyBatis 中就是通过XML映射文件或注解的方式将 SQL 语句和 Java 对象进行映射。
    - 没有相关开发经验也没有关系，通俗的说，ORM 就是一种工具或框架，它将数据库中的表和数据都映射成我们代码中的对象和属性，从而使我们能够像操作实例对象一样去操作数据库。

  举个例子：
    - 假设我们有一个 User 表，其中包括 id、name 和 age 三个字段。如果不使用 ORM，我们需要先使用 SQL 语句查询数据库，然后将查询结果手动转换成 Java 对象，再进行业务操作。这样做的话，我们需要写很多繁琐的
      SQL 语句，而且还需要手动将查询结果转换成 Java 对象，挺麻烦的。
    - 使用 ORM 后，我们只需要定义一个 User 类，通过一些注解或者配置文件告诉 ORM 框架 User 类与数据库中的哪个表相对应，以及每个属性对应表中的哪个字段，ORM 框架就可以自动的将 User 对象和数据库中的
      User 表进行映射。这样我们就可以像操作普通的 Java 对象一样去操作数据库了，不需要写复杂的 SQL 语句，也不需要手动进行数据转换。

<img src="https://www.helloimg.com/images/2023/07/20/oAWVvE.gif" width="30%">

## 1.2 OrmLite

Lite 我们都知道，是精简、轻量的意思，像手机或者软件就会有 lite 版，表示青春版或者精简版的意思。所以 OrmLite 这个直接看名字就能知道，是一个轻量级的 ORM 框架。
再比如本章后面要说的 SQLite，看名字就能知道就是一个轻量级的数据库。

- 具体来说，OrmLite 是一个基于 Java 的轻量级 ORM 框架。它提供了对 SQLite、MySQL、PostgreSQL、SQLServer 等数据库的访问支持。
- 由于轻量级的设计，他非常适合嵌入式设备和移动应用程序等场景。
- OrmLite 提供了一个简单易用的 API，以及一些可以方便地配置、自定义和扩展的工具类和接口。
- OrmLite 的查询性能很高，在大量数据查询时具有优势。此外，它还支持事务处理和存储过程，可以在数据库中执行复杂操作。

## 1.3 SQLite

- SQLite 是一种轻量级的嵌入式关系型数据库，被广泛应用于各种平台和应用程序中，包括移动设备、桌面应用、Web 应用等。
- SQLite 的代码量非常小，可靠性高，而且它不需要一个单独的服务器进程或操作系统访问数据库，因此它非常适合于嵌入式设备、移动设备以及桌面应用。
- 无类型、支持多数标准数据类型：SQLite 采用无类型的数据模型，因此它可以支持多种标准数据类型，包括 INTEGER、REAL、BLOB 和 TEXT 类型数据。
- 支持 ACID 事务：SQLite 支持 ACID（原子性、一致性、隔离性和持久性）事务特性，使开发人员可以方便地编写安全可靠的应用程序。
- SQLite 可以在多种平台上运行，包括 Windows、Linux、Mac OS X、Android 等，我们这里就着重讲在 Android 平台的运用。
- SQLite 非常易于学习和使用，它提供了非常简单的 SQL 语法，同时还提供了大量的 API 接口，可以轻松地进行各种数据操作。

# 二、OrmLite的使用

概念说多了没有太大意义，我们直接进行实操，才能深入体会 ORM 这种思想和 OrmLite 在操作数据库方面带来的便利。

## 2.1 引入依赖

- 由于是安卓开发，我们这里使用 Gradle 作为构建工具。在 app 的 build.Gradle 中引入依赖包，然后重新同步一下依赖：

```groovy
dependencies {
    //...
    //ormlite
    implementation 'com.j256.ormlite:ormlite-core:4.48'
    implementation 'com.j256.ormlite:ormlite-android:4.48'
    //...
}
```

## 2.2 定义数据模型类

- 我们这里简单定义一个 User 类，表示用户，属性有唯一id、姓名、年龄和性别，

```java

@DatabaseTable(tableName = "user")
public class User {
    //唯一身份识别
    @DatabaseField(generatedId = true) // 自动生成的主键，主键可以省略canBeNull = false
    private int id;

    //姓名
    @DatabaseField(columnName = "name", canBeNull = false)
    private String name;

    //年龄
    @DatabaseField(columnName = "age", canBeNull = false)
    private int age;

    //性别
    @DatabaseField(columnName = "gender", canBeNull = false)
    private String gender;

    public User() {
    }

    //其他构造器、getter和setter方法、toString等
}
```

<font color="FF0000"><br>重点来了（敲黑板!!）</font>
首先解释一下这段代码：**使用 OrmLite 框架，定义一个 User 类，并映射为数据库中的一张名为 user 的表。**

1. 注解解释：
    1. @DatabaseTable(tableName = "user")
       该注解用于标记该类为数据库表，tableName 指定了表的名称，在该注解中可以设置一些属性，例如索引、外键等。
    2. @DatabaseField(generatedId = true)
       该注解用于标记主键字段，generatedId=true 表示该字段为自增长主键。如果不想使用自增主键，在对应的 @DatabaseField 注解中加入 id = true 即可。
    3. @DatabaseField(columnName = "name", canBeNull = false)
       该注解用于标记一个普通的字段，columnName 指定了该字段在表中的列名，canBeNull=false 表示该字段不能为空。
2. 注意事项：
    1. 无参构造必须存在
       有框架使用经验的小伙伴对这个应该不会陌生，<font color="FF0000">框架 = 反射 + 注解 + 设计模式</font>，这和必须存在无参构造有什么关系呢？  
       Java 反射机制会根据类的信息动态地创建对象，并给属性赋值，而这个过程是通过默认的**无参构造**方法实现的。如果一个类没有提供无参构造方法，那么反射机制就无法创建该类的实例，也就无法将其映射为数据库表中的一行数据。

## 2.3 创建工具类

### 2.3.1 数据库管理工具类

为了重用代码，提高代码的可复用性、可读性和可维护性。我们构建一个工具类专门管理数据库：

```java
public class DatabaseHelper extends OrmLiteSqliteOpenHelper {

    // 数据库名称
    private static final String DATABASE_NAME = "db_demo.db";

    // 数据库版本号
    private static final int DATABASE_VERSION = 1;

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db, ConnectionSource connectionSource) {
        try {
            // 在这里创建表格
            TableUtils.createTable(connectionSource, User.class);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, ConnectionSource connectionSource, int oldVersion, int newVersion) {
        try {
            // 在这里升级表格
            TableUtils.dropTable(connectionSource, User.class, true);
            onCreate(db, connectionSource);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

之后，为了操作我们的数据库，我们应该为每一个实体类都创建一个相应的 DAO 接口和实现类。但是 OrmLite 已经提供了大量的 API 来操作数据库，不需要我们再手动编写 DAO 接口和实现类。
通过 OrmLite 提供的 Dao 接口，我们可以非常方便地进行数据库的插入、更新、删除和查询等操作。
例如，以下代码演示了如何使用 Dao 类完成对 User 实体类的插入操作：

```java
        Dao<User, Integer> userDao=ormLiteUtils.getDao(User.class);
        User user=new User("雷皓",18,"男");
        userDao.create(user);
```

在上述代码中，Dao 后面的两个泛型参数 User 和 Integer 分别表示需要操作的实体类和实体类主键的类型。

### 2.3.2 操作 DAO 对象的工具类

为了方便我们更容易的创建和获取 DAO 对象，我们创建一个工具类：

```java
public class OrmLiteUtils {

    // 单例模式静态变量
    private static OrmLiteUtils instance;
    // DatabaseHelper 对象
    private final DatabaseHelper databaseHelper;
    // UserDao 对象
    private Dao<User, Integer> userDao;

    // 私有构造方法
    private OrmLiteUtils(Context context) {
        // 创建 DatabaseHelper 对象
        databaseHelper = new DatabaseHelper(context);
    }

    // 获取实例对象（实现单例模式）
    public static synchronized OrmLiteUtils getInstance(Context context) {
        if (instance == null) instance = new OrmLiteUtils(context);

        return instance;
    }

    // 获取 UserDao 对象
    public synchronized Dao<User, Integer> getUserDao() throws SQLException, java.sql.SQLException {
        if (userDao == null)
            // 获取 Dao 对象
            userDao = databaseHelper.getDao(User.class);

        return userDao;
    }
}
```

该工具类提供了懒加载模式的 DAO 获取方式，即只有在第一次调用 getUserDao() 方法时才会去创建相应的 DAO 对象，并缓存在类的私有属性中。这种方式可以优化性能，提高应用启动速度。同时，为了线程安全，该类中的 DAO
获取方法都是同步方法，防止多线程获取时出现冲突。

在实际开发中，根据业务的需要，更改对应的数据库和实例类对象即可。

## 2.4 开始使用

好了，准备工作已经全部完成，接下来就会拥有非常愉快的数据库操作体验了！
接下来我们创建一个非常简单的应用，测试我们的上面的配置能否正常工作。具体的安卓代码就省略了，它不是今天的重点。
应用很简单，提供几个按钮，可以操作 user 表即可。

<img src="https://www.helloimg.com/images/2023/07/20/oAWhe9.jpg" width="50%">

### 2.4.1 获取数据库访问对象

```java
public class SQLiteActivity extends AppCompatActivity {
    //数据库访问对象
    private Dao<User, Integer> userDao;

    /*...*/
    private void initEvent() {
        /*...*/
        // 点击获取 Dao 对象
        OrmLiteUtils ormLiteUtils = OrmLiteUtils.getInstance(context);
        userDao = ormLiteUtils.getUserDao();
        /*...*/
    }
}
```

### 2.4.2 插入操作

```java
public class SQLiteActivity extends AppCompatActivity {
    private void initEvent() {
        //插入数据
        binding.btnInsert.setOnClickListener(view -> {
            /* 其他操作 */
            User user = new User(name, Integer.parseInt(age), gender);
            //插入用户
            userDao.create(user);
            /* 其他操作 */
        });
    }
}
```

### 2.4.3 删除操作

```java
public class SQLiteActivity extends AppCompatActivity {
    private void initEvent() {
        //删除数据
        binding.btnDeleteById.setOnClickListener(view -> {
            /* 其他操作 */
            User user = userDao.queryForId(Integer.valueOf(id));
            //删除用户
            userDao.delete(user);
            /* 其他操作 */
        });
    }
}
```

这里就演示这两个吧，因为其他操作也非常简单，几行代码就可以搞定，感兴趣的同学可以去探索一下。
接下来我们看看他给我们存到数据库里面的内容：

<img src="https://www.helloimg.com/images/2023/07/20/oAWjTX.png"  width="50%">

没有任何问题！

# 三、总结

- 在使用 OrmLite 时，需要先定义数据表对应的实体类，然后通过 Annotation 来配置表名、字段名、主键等信息。OrmLite 还可以帮助程序员自动生成表结构，同时支持手动管理版本升级。
- 除此之外，OrmLite 还提供了一些高级功能，例如事务处理、外键约束、查询器（QueryBuilder）等。查询器是 OrmLite 中用于查询数据的 API，支持链式调用、多条件查询等功能，可以帮助程序员快速提取所需数据。
- 总之，OrmLite 简单易用、灵活可扩展、性能优良，是 Android 数据库操作中非常值得探究的一种工具。