 # ContentProvider基础知识



ContentProvider是Android四大组件之一。主要作用是应用程序之间共享数据。例如，读取通讯录，短信到自己的APP中就需要用到ContentProvider。因为通讯录和短信是独立的应用，自己的APP读取通讯录和短信的内容需要进程间通信。ContentProvider为此提供了便捷的API。



## 基本使用

在访问ContentProvider时，需要通过ContentResolver类的实例

![Interaction between ContentProvider, other classes, and storage.](images/content-provider-interaction.png)



### ContentProvider类

#### 组织数据方式

ContentProvider主要以 表格的形式 组织数据 
同时也支持文件数据，只是表格形式用得比较多
每个表格中包含多张表，每张表包含行 & 列，分别对应记录 & 字段 
同数据库

#### 主要方法

进程间共享数据的本质是：添加、删除、获取 & 修改（更新）数据
所以ContentProvider的核心方法也主要是下面四个方法。

这四个方法都不是在主线程中执行的。

```java
  // 外部进程向 ContentProvider 中添加数据
  public Uri insert(Uri uri, ContentValues values) 
  
   // 外部进程 删除 ContentProvider 中的数据
  public int delete(Uri uri, String selection, String[] selectionArgs) 
 
// 外部进程更新 ContentProvider 中的数据
  public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
  
// 外部应用 获取 ContentProvider 中的数据
  public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,  String sortOrder)　 
  
```

除以上四个方法外，ContentProvider还有两个比较重要的方法。

```java
// 在启动ContentProvider时，初始化ContentProvider组件
// 运行在主线程，不要执行耗时操作
// 当使用SQLite作为数据源时，不要在此方法中调用SQLiteOpenHelper#getReadableDatabase和SQLiteOpenHelper#getWritableDatabase方法
public abstract boolean onCreate();

// 得到数据类型，即返回当前 Url 所代表数据的MIME类型
public abstract @Nullable String getType(@NonNull Uri uri);
```




Android为常见的数据（如通讯录、日程表等）提供了内置了默认的ContentProvider
但也可根据需求自定义ContentProvider，但上述6个方法必须重写 



### ContentResolver类

#### 作用

统一管理不同 `ContentProvider`间的操作
即通过 URI 即可操作 不同的ContentProvider 中的数据
外部进程通过 ContentResolver类 从而与ContentProvider类进行交互

#### 为什么要使用通过ContentResolver类从而与ContentProvider类进行交互，而不直接访问ContentProvider类？

答：一般来说，一款应用要使用多个ContentProvider，若需要了解每个ContentProvider的不同实现从而再完成数据交互，操作成本高 & 难度大
所以再ContentProvider类上加多了一个 ContentResolver类对所有的ContentProvider进行统一管理。



ContentResolver 具有与ContentProvider相对应的增删改查方法。

```java
// 外部进程 插入 ContentProvider 中的数据
public Uri insert(Uri uri, ContentValues values)　 

// 外部进程 删除 ContentProvider 中的数据
public int delete(Uri uri, String selection, String[] selectionArgs)

// 外部进程更新 ContentProvider 中的数据
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　 

// 外部应用 获取 ContentProvider 中的数据
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)

```



使用方法

```java
// Queries the user dictionary and returns results
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause,                   // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows
```



## 辅助工具类

### UriMatcher
作用：操作 URI

### ContentObserver
定义：内容观察者
作用：观察 Uri引起 ContentProvider 中的数据变化 & 通知外界（即访问该数据访问者） 
当ContentProvider 中的数据发生变化（增、删 & 改）时，就会触发该 ContentObserver类

### UriMatcher
在ContentProvider 中注册URI
根据 URI 匹配 ContentProvider 中对应的数据表





## 读取通讯录例子



```java
 Uri uri = Uri.parse(CONTRACTS);

 Cursor cursor = getContentResolver().query(uri, null, null, null, null);

 while (cursor.moveToNext()) {
      int index = cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME);
      String name = cursor.getString(index);
      Log.d("nxb", "name="+name);
}
```

