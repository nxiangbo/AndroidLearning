# Kotlin Android 扩展

Kotlin android extension的目的是让我们从烦人的findViewById方法中解脱出来，我们可以直接使用View的id名字来获取这个View。虽然有很多优秀的开源库（如，butterknife）可以通过注解的方式绑定View。而Kotlin 安卓扩展插件提供了更方便快捷的方法。它不需要添加任何额外的代码就可以绑定View。



```java
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textview.text = "Hi Kotlin"
    }
}
```

`textView` 是对 `Activity` 的一项扩展属性，与在 `activity_main.xml` 中的声明具有同样类型 。

## 如何应用

在新建一个Kotlin的android项目后，项目顶层的build.gradle文件中会自动添加`classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"`

在项目模块的build.gradle 中会默认使用kotlin-android-extensions插件。

```java
apply plugin: 'kotlin-android-extensions'
```

由此可见，kotlin-android-extensions 是kotlin-gradle-plugin 的一部分。





## kotlin android extension的工作原理

在Android studio中，可以通过Tools->Kotlin->Show Kotlin ByteCode工具很方便的查看Kotlin的字节码。因为字节码阅读比较困难，我们可以将它反编译成Java代码。代码如下所示。

```java
public final class MainActivity extends AppCompatActivity {
   private HashMap _$_findViewCache;

   protected void onCreate(@Nullable Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      this.setContentView(2131296284);
      TextView var10000 = (TextView)this._$_findCachedViewById(id.textview);
      Intrinsics.checkExpressionValueIsNotNull(var10000, "textview");
      var10000.setText((CharSequence)"Hi Kotlin");
   }

   public View _$_findCachedViewById(int var1) {
      if (this._$_findViewCache == null) {
         this._$_findViewCache = new HashMap();
      }

      View var2 = (View)this._$_findViewCache.get(var1);
      if (var2 == null) {
         var2 = this.findViewById(var1);
         this._$_findViewCache.put(var1, var2);
      }

      return var2;
   }

   public void _$_clearFindViewByIdCache() {
      if (this._$_findViewCache != null) {
         this._$_findViewCache.clear();
      }

   }
}
```

从上面代码我们可以得出以下结论：

1. Kotlin Android扩展插件本质上还是调用findViewById方法绑定View，只不过插件帮我们在编译时做好了，不需要我们手动添加。
2. 相较于Java版本的代码，Kotlin多了一个 _$_findViewCache的变量和相关方法。这些代码有何作用呢？
3. Kotlin Android扩展插件在编译源文件时，做了一些字节码插入操作



## View 缓存

调用findViewById()时会比较慢，尤其是层级比较深的布局。因此，Kotlin Android扩展插件提供了一个缓存策略。主要目的是减少findViewById()的调用次数。

默认情况下，Android Extensions adds a hidden cache function and a storage field to each container ([`Activity`](https://developer.android.com/reference/android/app/Activity.html), [`Fragment`](https://developer.android.com/reference/android/support/v4/app/Fragment.html), [`View`](https://developer.android.com/reference/android/view/View.html) or a `LayoutContainer` implementation) written in Kotlin. The method is pretty small so it does not increase the size of APK much.

In the following example, `findViewById()` is only invoked once:

```
class MyActivity : Activity()

fun MyActivity.a() { 
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```

然而在下面的例子中：

```
fun Activity.b() { 
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```

We wouldn't know if this function would be invoked on only activities from our sources or on plain Java activities also. Because of this, we don’t use caching there, even if `MyActivity` instance from the previous example is passed as a receiver.

### View 缓存策略

You can change the caching strategy globally or per container. This also requires switching on the [experimental mode](https://www.kotlincn.net/docs/tutorials/android-plugin.html#experimental-mode).

Project-global caching strategy is set in the `build.gradle` file:

```
androidExtensions {
    defaultCacheImplementation = "HASH_MAP" // also SPARSE_ARRAY, NONE
}
```

By default, Android Extensions plugin uses `HashMap` as a backing storage, but you can switch to the `SparseArray` implementation, or just switch off caching. The latter is especially useful when you use only the [Parcelable](https://www.kotlincn.net/docs/tutorials/android-plugin.html#parcelable) part of Android Extensions.

Also, you can annotate a container with `@ContainerOptions` to change its caching strategy:

```
import kotlinx.android.extensions.ContainerOptions

@ContainerOptions(cache = CacheImplementation.NO_CACHE)
class MyActivity : Activity()

fun MyActivity.a() { 
    // findViewById() will be called twice
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```

