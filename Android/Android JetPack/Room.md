# Room

> 原文地址：[Android Room with a view](https://codelabs.developers.google.com/codelabs/android-room-with-a-view/index.html?index=..%2F..index#0)


## Android推荐的架构组件


![image.png](https://upload-images.jianshu.io/upload_images/7819453-b5e960ab219970ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Entity:** 当使用架构组件时，Entity是描述数据库表的类，这个类通常使用注解。

**SQLite database:** SQLite是一个数据库，存储数据。为了简单起见，忽略其他的存储工具（如web服务器等）。Room持久性库用于创建和维护数据库。

**DAO:**即数据访问对象。之前使用 [`SQLite``OpenHelper`](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) 类定义这些内容。当使用DAO时，我们可以调用方法，room做其余的操作。

**Room database:**SQLite数据库之上的数据库层，负责处理以前使用SQLiteOpenHelper处理的普通任务。Room数据库使用DAO查询SQLite数据库。

**Repository:** Repository是用于管理多个数据资源，例如数据库，网络等。

**ViewModel****:**  为UI提供数据。ViewModel作为Repository和UI的通信中心。ViewModel在数据配置更改后仍然存在。

**LiveData****:**  LiveData是可以被观察到的数据持有类。它里面缓存或持有了最新的数据。当数据改变时会通知它的观察者。LiveData是可以感知生命周期的。UI组件只是观察相关数据，不会停止或恢复观察。 LiveData自动管理所有这些，因为它在观察时意识到相关的生命周期状态变化。

下面通过一个官方给的RoomWordSample例子讲解android架构组件中的Room和ViewModel是如何使用的。

## RoomWordSample概述
RoomWordSample 的功能很简单，在Room数据库中存储单词列表，并将其显示在RecyclerView中。MainActivity通过RecyclerView展示单词列表。NewWordActivity 用于添加一个单词到数据库中。



![image.png](https://upload-images.jianshu.io/upload_images/7819453-0d23c0751625d6c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




## 添加依赖

```java
	// Room components
    implementation "android.arch.persistence.room:runtime:$rootProject.roomVersion"
    annotationProcessor "android.arch.persistence.room:compiler:$rootProject.roomVersion"
    androidTestImplementation "android.arch.persistence.room:testing:$rootProject.roomVersion"

    // Lifecycle components
    implementation "android.arch.lifecycle:extensions:$rootProject.archLifecycleVersion"
    annotationProcessor "android.arch.lifecycle:compiler:$rootProject.archLifecycleVersion"
```



## 创建Entity

- `@Entity(tableName = `**"word_table"**`)`
  每个@Entity类表示一个数据库表中的实体。可以使用tableName指定表的名字。
- `@PrimaryKey`
 每个entity都需要一个主键。
- `@NonNull`
   表示参数，成员变量或者方法返回值从不为null
- `@ColumnInfo(name = `**"word"**`)`
  指定数据库表的列名。
- This sample provides a `getWord()` method.存储在数据库中的每个字段都需要一个公共“getter”方法。此示例提供了getWord（）方法

创建Word类，使用上述注解。

```java
@Entity(tableName = "word_table")
public class Word {
	@PrimaryKey
	@NonNull
	@ColumnInfo(name="word")
	private String mWord;

	public Word(String word) {
		this.mWord = word;
	}

	public String getWord() {
		return this.mWord;
	}
}
```



## 创建DAO

DAO，即数据访问接口。可以将SQL查询语句与方法相关联。

DAO必须是接口或抽象类。

默认情况下，所有查询必须在独立的线程。

```java
@Dao
public interface WordDao {
	@Insert
	void insert(Word word);

	@Query("select * from word_table order by word asc")
	List<Word> queryAll();

	@Query("delete from word_table")
	void deleteAll();
}
```

## LiveData 类

LiveData是 [lifecycle 库 ](https://developer.android.com/topic/libraries/architecture/lifecycle.html)中的类，主要是用于实时更新数据。类似于观察者模式。

在WordDao类中，将getWordAll的返回值改为LiveData的包装类

```java
@Query("select * from word_table order by word asc")
	LiveData<List<Word>> getWordAll();
```

## 添加Room数据库

### 什么是Room数据库

Room是置于SQLite数据库上面的数据库。之前我们使用 [`SQLiteOpenHelper`](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)类处理一些与数据相关的任务，而Room处理之前使用 [`SQLiteOpenHelper`](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)类处理普通的任务。

- Room使用DAO查询数据库
- 默认情况下，为了提高UI性能，Room不允许在主线程中执行查询数据库操作。LiveData在后台线程上异步更新数据。
- Room 提供SQLite语句编译时检查
- 自定义的Room类必须是抽象类且必须继承`RoomDatabase`
- 通常，在整个APP中，只需要一个Room database实例。

### 实现Room数据库



```java
@Database(entities = {Word.class}, version=1)
public abstract class WordRoomDatabase extends RoomDatabase {
	private static volatile WordRoomDatabase INSTANCE;

	public static WordRoomDatabase getDatabase(Context context){
		if (INSTANCE == null) {
			synchronized (WordRoomDatabase.class) {
				if (INSTANCE == null) {
					INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
							WordRoomDatabase.class, "word_database").build();
				}
			}
		}
		return INSTANCE;
	}

	@NonNull
	@Override
	protected SupportSQLiteOpenHelper createOpenHelper(DatabaseConfiguration config) {
		return null;
	}

	@NonNull
	@Override
	protected InvalidationTracker createInvalidationTracker() {
		return null;
	}

	@Override
	public void clearAllTables() {

	}

	public abstract WordDao wordDao();
}
```

## 创建Repository



### Repository是什么

Repository类用于访问多个数据源。Repository并不是架构组件库的一部分，而是代码解耦和架构比较推荐的方法。Repository类处理数据操作。它为应用程序提供了一个整洁的API。

![image.png](https://upload-images.jianshu.io/upload_images/7819453-d147e36bd0a8b69f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 为什么用Repository


Repository管理数据的查询线程，同时，可以使用多个后端。在常规情况下，Repository主要实现从服务端拉取数据还是从本地数据库拉取数据的逻辑。

## 实现Repository

```java
public class WordRepository {
	private WordDao mWordDao;
	private LiveData<List<Word>> mAllWords;

	public WordRepository(Application application) {
		WordRoomDatabase db = WordRoomDatabase.getDatabase(application);
		mWordDao = db.wordDao();
		mAllWords = mWordDao.getWordAll();
	}

	public LiveData<List<Word>> getAllWords() {
		return mAllWords;
	}

	public void insert(Word word) {
		new InsertAsyncTask(mWordDao).execute(word);
	}

	private static class InsertAsyncTask extends AsyncTask<Word, Void, Void> {
		private WordDao mAsyncDao;

		InsertAsyncTask(WordDao wordDao) {
			this.mAsyncDao = wordDao;
		}

		@Override
		protected Void doInBackground(Word... words) {
			mAsyncDao.insert(words[0]);
			return null;
		}
	}
}
```

## 创建ViewModel


ViewModel 主要扮演为UI提供数据的角色，同时，在配置更改后可以继续存在。ViewModel可以看做Repository和UI的通信中心。也可以使用ViewModel共享数据。ViewModel是[lifecycle library](https://developer.android.com/topic/libraries/architecture/lifecycle.html)的一部分。

![image.png](https://upload-images.jianshu.io/upload_images/7819453-b6175b062a24e71b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 为什么用ViewModel


ViewModel将UI数据和Activity和Fragment类进行分离，更符合单一职责原则：Activity和Fragment负责展示UI，而ViewModel负责持有并处理UI所需要的所有数据。

 在ViewModel 中，使用LiveData更新UI的数据。因为LiveData有以下几个优点：

-  可以监听数据，只有当数据更改时才会更新UI。
- 通过ViewModel可以将Repository和UI完全隔离。在ViewModel中不会直接进行数据库调用，这使得代码更方便进行测试。



```java
public class WordViewModel extends AndroidViewModel {
	private LiveData<List<Word>> mAllWord;
	private WordRepository mRepository;
	public WordViewModel(@NonNull Application application) {
		super(application);
		mRepository = new WordRepository(application);
		mAllWord = mRepository.getAllWords();
	}

	public LiveData<List<Word>> getAllWord() {
		return mAllWord;
	}

	public WordRepository getRepository() {
		return mRepository;
	}

	public void insert(Word word) {
		mRepository.insert(word);
	}
}
```



## 连接数据

从上文可知，ViewModel是UI与Repository的通信中心。即UI更新数据是通过ViewModel进行的。为了显示当前数据的内容，在ViewModel中添加一个观察者，用以监听LiveData的更改。

在MainActivity的onCreate()方法中创建ViewModel实例，并监听数据库数据的更新。

```java
mWordViewModel = ViewModelProviders.of(this).get(WordViewModel.class);
mWordViewModel.getAllWords().observe(this, new Observer<List<Word>>() {
   @Override
   public void onChanged(@Nullable final List<Word> words) {
       // Update the cached copy of the words in the adapter.
       adapter.setWords(words);
   }
});
```



## 总结

通过ViewModel可以将UI和数据层进行分离。DAO接口WordDao会在编译时生成DAO接口的实现类，而WordRoomDatabase也会生成相对应的实现类。其本质还是使用SQLiteOpenHelper来处理数据库操作，只不过开发起来更简单。