---
title: 'Room 组件使用示例'
date: '2023-07-22'
description: 'Room 是 Android 中的一个持久性库，用于简化数据库的访问和管理。'
cover: 'https://www.helloimg.com/images/2023/07/22/oAQU7o.png'
categories:

- 编程开发

tags:

- Kotlin
- Android
- Jetpack
- 数据库

---

> Room 是 Jetpack 库中的组件之一，用于创建、存储和管理由 SQLite 数据库支持的持久性数据。

# 一、概念及组成部分

## 1.1 概念

- 为了更简单的管理 SQLite 数据库，网上出现了很多持久层框架，比如最著名的 GreenDao；还有我前面文章讲过的 Ormlite，还有 SugarORM、ActiveAndroid 等等。
- 今天要讲的这个 Room，目前安卓开发群体中用得相对较少，原因是他要和其他 Jetpack 组件一起使用才能发挥其最大优势，而现在 Jetpack 并未完全普及开来，也就导致了 Room 并未变得十分流行。
- 简而言之，Room 提供了一个抽象层，使得使用 SQLite 数据库变得更简单、更可靠，并且提供了与其他 Jetpack 组件（如 LiveData 和 ViewModel）的集成。项目如果使用了大量的 Jetpack 组件进行开发，那么数据库管理这一块儿，使用 Room 再好不过。

## 1.2 组成部分

   1. `实体（Entity）`：实体是数据库表的映射对象。通过在类上添加 @Entity 注解，可以将类声明为一个实体，并指定表名、主键和列等信息。
   2. `数据访问对象（DAO）`：数据访问对象是用于定义数据库操作方法的接口或抽象类。通过在方法上添加注解（如 @Insert、@Delete、@Query 等），可以指定对数据库的插入、删除和查询等操作。
   3. `数据库对象（Database）`：数据库对象是 Room 的核心组件，用于管理应用程序的整个数据库。通过创建一个继承自 RoomDatabase 的抽象类，并指定其中的实体和版本等信息，可以定义一个数据库对象。数据库对象还通过提供 DAO 的抽象方法，允许应用程序对数据库执行各种操作。
   4. `仓库（Repository）`：处理数据源和业务逻辑，Repository 是连接数据访问对象（DAO）和视图模型/用户界面之间的中间层，负责从数据库获取数据并向上层提供数据。
   5. `视图模型（ViewModel）`：用于管理应用程序的数据和业务逻辑。它通常与 LiveData 一起使用，可以将业务逻辑和 UI 组件进行解耦，使得数据持久性和业务逻辑不受 UI 生命周期的影响。

# 二、基本使用

## 2.1 导入依赖

- 在项目的 build.gradle 文件中添加 Room 的依赖项。并确保你已经在 repositories 部分添加了 Google 仓库。

```groovy
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    //添加 kotlin-kapt 插件
    id 'kotlin-kapt'
}
dependencies {
    //添加主依赖
    implementation 'androidx.room:room-runtime:2.4.2'
    //添加处理注解的依赖
    kapt 'androidx.room:room-compiler:2.4.2'
}
```
- `注：`kotlin-kapt 是 Kotlin 语言中的一个插件，它用于处理 Kotlin 注解处理器（KAPT）。KAPT 允许在编译时生成额外的代码，通常用于自动生成代码、实现依赖注入或数据库操作等。

## 2.2 创建实体类（Entity）

```kotlin
@Entity
class Word {
    
    @PrimaryKey(autoGenerate = true)
    var id = 0

    @ColumnInfo(name = "english_word")
    var word: String
    
    @ColumnInfo(name = "chinese_meaning")
    var chineseMeaning: String

    //根据自己的需求添加构造器
    constructor(word: String, chineseMeaning: String) {
        this.word = word
        this.chineseMeaning = chineseMeaning
    }

}
```

- `注：`使用 @Entity 注解创建一个用于映射数据库表的实体类。如果使用 Kotlin 编写不需要写 getter 和 setter 方法。@ColumnInfo 直接使用，这样数据库里面的字段名称就是你定义的变量名称。

## 2.3 创建数据访问对象（DAO）

```kotlin
@Dao
interface WordDao {

    @Insert //vararg  word: Word 这里表示参数数量不定
    fun insertWords(vararg word: Word)

    @Update
    fun updateWords(vararg word: Word)

    @Delete
    fun deleteWords(vararg word: Word)

    @Query("delete from word")
    fun deleteAllWords()

    @Query("select * from word order by id desc")
    fun getAllWordsLive(): LiveData<List<Word>>

}
```

- `注：`@Insert、@Update、@Delete 分别用于插入、更新和删除数据库中的数据，这个看名字就能看出来；@Query 用于执行自定义查询语句，用的比较多。

## 2.4 创建数据库对象（Database）

```kotlin
@Database(entities = [Word::class], version = 1, exportSchema = false)
abstract class WordDatabase : RoomDatabase() {

    //伴生对象，里面的方法可以视为Java中的静态方法，属性可以视为静态属性
    companion object {
        // 单例实例
        @Volatile
        private var INSTANCE: WordDatabase? = null

        // 获取单例实例的方法
        fun getInstance(context: Context): WordDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext, WordDatabase::class.java, "word_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }

    abstract fun getWordDao(): WordDao
    
}
```

- `注：`这里采用了单例模式，保证全局只有一个 WordDatabase 对象，避免数据库资源的浪费和多线程并发访问的问题。如果有多个 Dao 对象，在 @Database 注解的 entities 数组中添加即可。

## 2.5 创建仓库（Repository）

```kotlin
class WordRepository(context: Context) {

    // 获取数据库实例
    private val wordDatabase = WordDatabase.getInstance(context)

    private val wordDao = wordDatabase.getWordDao()

    val allWordsLive = wordDao.getAllWordsLive()

    fun insertWords(vararg word: Word) {
        InsertAsyncTask(wordDao).execute(*word)
    }

    fun updateWords(vararg word: Word) {
        UpdateAsyncTask(wordDao).execute(*word)
    }


    fun deleteWords(vararg word: Word) {
        DeleteAsyncTask(wordDao).execute(*word)
    }

    fun deleteAllWords() {
        DeleteAllAsyncTask(wordDao).execute()
    }


    //插入
    class InsertAsyncTask(private val wordDao: WordDao) : AsyncTask<Word, Unit, Unit>() {

        override fun doInBackground(vararg words: Word): Unit? {
            wordDao.insertWords(*words)
            return null
        }

    }

    //清空
    class DeleteAllAsyncTask(private val wordDao: WordDao) : AsyncTask<Unit, Unit, Unit>() {

        override fun doInBackground(vararg unit: Unit): Unit? {
            wordDao.deleteAllWords()
            return null
        }

    }

    //删除
    class DeleteAsyncTask(private val wordDao: WordDao) : AsyncTask<Word, Unit, Unit>() {

        override fun doInBackground(vararg words: Word): Unit? {
            wordDao.deleteWords(*words)
            return null
        }

    }

    //更新
    class UpdateAsyncTask(private val wordDao: WordDao) : AsyncTask<Word, Unit, Unit>() {

        override fun doInBackground(vararg words: Word): Unit? {
            wordDao.updateWords(*words)
            return null
        }

    }

}
```

- `注：`上面对于数据库的操作全部放在了异步子线程中，这是 Room 默认的规定，如果直接在主线程进行数据库操作会直接异常。但如果你确实想直接在主线程中进行操作，需要在创建 Database 的 databaseBuilder 处添加`.allowMainThreadQueries()`，即允许在主线程进行数据库操作。

## 2.6 创建视图模型（ViewModel）

```kotlin
class WordViewModel(application: Application) : AndroidViewModel(application) {

    private val wordRepository = WordRepository(application)

    fun getAllWordsLive():LiveData<List<Word>> {
        return wordRepository.allWordsLive
    }

    fun insertWords(vararg word: Word) {
        wordRepository.insertWords(*word)
    }

    fun updateWords(vararg word: Word) {
        wordRepository.updateWords(*word)
    }

    fun deleteWords(vararg word: Word) {
        wordRepository.deleteWords(*word)
    }

    fun deleteAllWords() {
        wordRepository.deleteAllWords()
    }

}
```

- `注：`这里选择继承`AndroidViewModel`而不是直接继承`ViewModel`是因为我们需要用到上下文对象 application，同时，AndroidViewModel 与 Activity 或 Fragment 的生命周期无关，可以避免内存泄漏。

## 2.7 测试

- 在 Activity 或者 Fragment 中，我们直接调用 ViewModel 中的相关方法即可，比如我要插入几条数据：

```kotlin
        //插入
        binding.btnInsert.setOnClickListener {
            val english = arrayOf("hello", "world", "google", "pear", "apple")
            val chinese = arrayOf("你好", "世界", "谷歌", "梨", "苹果")
            for (i in english.indices) {
                wordViewModel.insertWords(Word(english[i], chinese[i]))
            }
        }

```

- 这样数据库中就插入了 5 条数据。

# 三、版本迁移

- 由于数据库的结构变化，导致需要进行数据库版本升级和数据迁移，而这两件事一直以来都是比较麻烦的。这里我们演示一下在 Room 如何进行一个简单的数据库升级操作。
- 假如我们的数据库 word 表需要新增一个字段`bar_data`，我们首先需要做的是在 Word 实体类中增加一个属性

```kotlin
    @ColumnInfo(name = "bar_data")
    var barData: Boolean = false
```

- 然后修改我们的 WordDatabase：

```kotlin
@Database(entities = [Word::class], version = 2, exportSchema = false)
abstract class WordDatabase : RoomDatabase() {

    //伴生对象，里面的方法可以视为Java中的静态方法，属性可以视为静态属性
    companion object {
        // 单例实例
        @Volatile
        private var INSTANCE: WordDatabase? = null

        // 获取单例实例的方法
        //.fallbackToDestructiveMigration()表示破坏式迁移，不会保留以前的数据，更改数据库结构及版本号后添加这个可以实现版本升级
        fun getInstance(context: Context): WordDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext, WordDatabase::class.java, "word_database"
                )//.fallbackToDestructiveMigration()
                    .addMigrations(Migration_1_2).build()
                INSTANCE = instance
                instance
            }
        }

        //自定义迁移逻辑
        private val Migration_1_2 = object : Migration(1, 2) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("Alter table word ADD COLUMN bar_data INTEGER NOT null Default 0")
            }
        }
    }

    abstract fun getWordDao(): WordDao
    
}
```

- 主要注意几点：
  1. 更改`version = 2`，如果实体类出现变化，必须更改数据库版本号，否则会出现异常。
  2. 创建一个 Migration_1_2 对象，也就是我们的自定义迁移逻辑，在里面执行我们的数据库相关操作。
  3. 将我们创建好的迁移逻辑通过`.addMigrations()`方法添加进 databaseBuilder。
- 上面就实现了一个最简单的数据库升级方法，可以不销毁目前用户数据的情况下进行数据库版本升级，可以看到代码中还有另一种方式，即`破坏式迁移`，这种迁移方式并不推荐，知道就可以了。

# 四、总结

- 总之，Room 数据库是一个优秀的工具，帮助开发人员有效地管理和操作本地数据，从而提升应用程序的性能和用户体验。无论是小型应用还是大型项目，使用 Room 数据库都是一个值得考虑的选择。