# Lint代码检查

## Android

### Performance

#### handler 引用泄露

> Since this Handler is declared as an inner class, it may prevent the outer class from being garbage collected. If the Handler is using a Looper or MessageQueue for a thread other than the main thread, then there is no issue. If the Handler is using the Looper or MessageQueue of the main thread, you need to fix your Handler declaration, as follows: Declare the Handler as a static class; In the outer class, instantiate a WeakReference to the outer class and pass this object to your Handler when you instantiate the Handler; Make all references to members of the outer class using the WeakReference object.  Issue id: HandlerLeak

#### HashMap VS  SparseArray

- 修改类

FileSystemApi



| Standard Java Structure                                      | Corresponding Android Structure                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [HashMap](https://developer.android.com/reference/java/util/HashMap.html) | [ArrayMap](https://developer.android.com/reference/android/util/ArrayMap.html) |
| HashMap<Integer, V>                                          | [SparseArray](https://developer.android.com/reference/android/util/SparseArray.html) |
| HashMap<Integer, Boolean>                                    | [SparseBooleanArray](https://developer.android.com/reference/android/util/SparseBooleanArray.html) |
| HashMap<Integer, Integer>                                    | [SparseIntArray](https://developer.android.com/reference/android/util/SparseIntArray.html) |
| HashMap<Integer, Long>                                       | [SparseLongArray](https://developer.android.com/reference/android/util/SparseLongArray.html) |
| HashMap<Long, V>                                             | [LongSparseArray](https://developer.android.com/reference/android/util/LongSparseArray.html) |



- 参考文档

> [Android: Should you use HashMap or SparseArray](https://greenspector.com/en/articles/2017-04-11-android-containers/)
>
> [SparseArray vs HashMap](https://stackoverflow.com/questions/25560629/sparsearray-vs-hashmap)

#### 无用的资源

- aiapps_scope_desc.png 



#### Static Field Leaks 

单例引用Activity或Fragment实例

异步任务



### FrameLayout can be replaced with <merge> tag



- 参考文档

>  [Android Layout Tricks #3: Optimize by merging](https://android-developers.googleblog.com/2009/03/android-layout-tricks-3-optimize-by.html)





## Java

### Performance

#### manual array copy

- SwanAppRoundCornerListView

> [copy an array](http://www.javapractices.com/topic/TopicAction.do?Id=3)
>
> [Difference between various Array copy methods ](https://stackoverflow.com/questions/1697250/difference-between-various-array-copy-methods)



#### Collection.toArray call style

- 修改类

  Palette




- 参考文档

>  [.toArray(new MyClass[0\]) or .toArray(new MyClass[myList.size()])?](https://stackoverflow.com/questions/174093/toarraynew-myclass0-or-toarraynew-myclassmylist-size)
>
>  [Arrays of Wisdom of the Ancients](https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)

#### String or StringBuilder



- 参考文档

> [Is chain of StringBuilder.append more efficient than string concatenation?](https://stackoverflow.com/questions/7586266/is-chain-of-stringbuilder-append-more-efficient-than-string-concatenation)





### Verbose or Redundant code constructs





#### Redundant type cast

SwanAppHttpAuthenticationDialog.java

SwanAppPageDialogsHandler.java

SwanAppMessengerObserver.java

BigBgHeaderLoadingLayout.java

HeaderLoadingLayout.java

HeaderLoadingLayout.java

#### Too weak variable type leads to unnecessary cast

> his inspection reports type casts which could be removed if the variable type is narrowed to the cast type. Example:
>   Object x = "  string  ";
>   System.out.println(((String)x).trim());
> Here changing the type of x to String will make the cast redundant. The suggested quick-fix updates variable type and removes all redundant casts on that variable.



### Probable bugs

#### String comparison using '==' instead of equals

SwanAppUBCEvent.java

NetworkBroadcastReceiver.java



>  [How do I compare strings in Java?](https://stackoverflow.com/questions/513832/how-do-i-compare-strings-in-java)
>
>  [What is the Java string pool and how is “s” different from new String(“s”)? ](https://stackoverflow.com/questions/2486191/what-is-the-java-string-pool-and-how-is-s-different-from-new-strings)

### Code Style issues





7






