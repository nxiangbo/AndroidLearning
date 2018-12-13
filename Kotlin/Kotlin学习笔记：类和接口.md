# Kotlin学习笔记：类和接口



![](images\Kotlin-class.png)



## 类

在Kotlin中，类默认是final和public的。如果该类需要被继承，则可以用open关键字显示的声明类。

可见性修饰符

![](images\Kotlin-class01.png)



## 构造方法

Kotlin引入了constructor和init关键字。constructor关键字用于声明一个主构造方法或者从构造方法。而init是一个初始化语句块。这个语句块会在类创建时，与主构造方法一起使用。因为主构造方法不能包含初始化代码。

```kotlin
class User constructor(_name: String){
    val name: String
    init {
        this.name = _name
    }
}
```





### 从构造方法

虽然大多数在Java中需要重载的构造方法场景都可以使用参数默认值和参数命名的方法解决掉，但是，还是有些场景需要声明多个构造方法，以便于以不同的方式初始化类。

例如，在Android中自定义View，往往就需要声明多个构造方法。此时，可以使用从构造方法。从构造方法可以声明任意多个。

```kotlin
class CustomView: View {
    constructor(context: Context) : super(context) {
        //...
    }
    constructor(context: Context, attrs: AttributeSet) : super(context, attrs) {
		// ...
    }
}
```



## 内部类

与java不同的是，默认情况下，内部类不能访问外部类。如果内部类想要访问外部类，需要使用inline关键字显示声明内部类。

![](images\kotlin-class02.png)



## sealed 类（密封类）

如果父类使用sealed修饰符，那么它会对可能创建的子类做出严格限制，并且所有子类必须嵌套在父类中。

## 数据类

在Java中，会出现很多样板代码，比如setter/getter，toString,equals,hashCode等方法。为了消除这些样板代码，Kotlin引用了数据类，即使用data关键字修饰类即可。

下面我们创建了一个User数据类，它有两个属性，姓名和年龄。

```kotlin
data class Person(val name:String, val age: Int)
```

在使用时，我们可以直接user1.name或user1.toString()调用。

为了更清楚的看到数据类是如何工作的，我们可以查看这个类的字节码。将字节码反编译为java代码，如下所示。

```java
@Metadata(
   mv = {1, 1, 9},
   bv = {1, 0, 2},
   k = 1,
   d1 = {"\u0000 \n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010\u000e\n\u0000\n\u0002\u0010\b\n\u0002\b\t\n\u0002\u0010\u000b\n\u0002\b\u0004\b\u0086\b\u0018\u00002\u00020\u0001B\u0015\u0012\u0006\u0010\u0002\u001a\u00020\u0003\u0012\u0006\u0010\u0004\u001a\u00020\u0005¢\u0006\u0002\u0010\u0006J\t\u0010\u000b\u001a\u00020\u0003HÆ\u0003J\t\u0010\f\u001a\u00020\u0005HÆ\u0003J\u001d\u0010\r\u001a\u00020\u00002\b\b\u0002\u0010\u0002\u001a\u00020\u00032\b\b\u0002\u0010\u0004\u001a\u00020\u0005HÆ\u0001J\u0013\u0010\u000e\u001a\u00020\u000f2\b\u0010\u0010\u001a\u0004\u0018\u00010\u0001HÖ\u0003J\t\u0010\u0011\u001a\u00020\u0005HÖ\u0001J\t\u0010\u0012\u001a\u00020\u0003HÖ\u0001R\u0011\u0010\u0004\u001a\u00020\u0005¢\u0006\b\n\u0000\u001a\u0004\b\u0007\u0010\bR\u0011\u0010\u0002\u001a\u00020\u0003¢\u0006\b\n\u0000\u001a\u0004\b\t\u0010\n¨\u0006\u0013"},
   d2 = {"Lcom/nxiangbo/kotlin_learning/User;", "", "name", "", "age", "", "(Ljava/lang/String;I)V", "getAge", "()I", "getName", "()Ljava/lang/String;", "component1", "component2", "copy", "equals", "", "other", "hashCode", "toString", "production sources for module app"}
)
public final class User {
   @NotNull
   private final String name;
   private final int age;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final int getAge() {
      return this.age;
   }

   public User(@NotNull String name, int age) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      this.name = name;
      this.age = age;
   }

   @NotNull
   public final String component1() {
      return this.name;
   }

   public final int component2() {
      return this.age;
   }

   @NotNull
   public final User copy(@NotNull String name, int age) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      return new User(name, age);
   }

   // $FF: synthetic method
   // $FF: bridge method
   @NotNull
   public static User copy$default(User var0, String var1, int var2, int var3, Object var4) {
      if ((var3 & 1) != 0) {
         var1 = var0.name;
      }

      if ((var3 & 2) != 0) {
         var2 = var0.age;
      }

      return var0.copy(var1, var2);
   }

   public String toString() {
      return "User(name=" + this.name + ", age=" + this.age + ")";
   }

   public int hashCode() {
      return (this.name != null ? this.name.hashCode() : 0) * 31 + this.age;
   }

   public boolean equals(Object var1) {
      if (this != var1) {
         if (var1 instanceof User) {
            User var2 = (User)var1;
            if (Intrinsics.areEqual(this.name, var2.name) && this.age == var2.age) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }
}
```

由此可见，在编译器将代码编译为字节码时，为User类自动添加了上述这些方法。

除了常用的toString等方法外，还自动生成了一个copy方法。那么，这个copy方法的作用是什么呢？为了让使用不可变对象的数据类变得更容易，Kotlin提供了一个copy方法，以便于通过创建副本的方式修改数据类

## 委托类



## object关键字

object关键字有三种应用场景：

- 单例
- 伴生对象（companion object）
- object表达式，可以替代Java匿名内部类

### 单例

object声明将 类声明和单一实例声明结合在一起。下面创建一个object

```kotlin
object ObjectDemo{
    
}
```

反编译为Java代码为

```java
public final class ObjectDemo {
   public static final ObjectDemo INSTANCE;

   static {
      ObjectDemo var0 = new ObjectDemo();
      INSTANCE = var0;
   }
}
```



## companion object

可以包含工厂方法或者与该类相关但不需要该类实例的方法。因为顶层函数不能访问类的私有方法，而companion object 可以访问类的私有属性和方法。

```kotlin
abstract class RepoDatabase : RoomDatabase() {
    abstract fun repos(): RepoDao

    companion object {

        @Volatile
        private var INSTANCE: RepoDatabase? = null

        fun getInstance(context: Context): RepoDatabase = INSTANCE ?: synchronized(this) {
            INSTANCE ?: buildDatabase(context).also { INSTANCE = it }
        }

        private fun buildDatabase(context: Context) =
                Room.databaseBuilder(context, RepoDatabase::class.java, "zhihu.db").build()
    }
}
```

### object表达式

可以替代匿名内部类

```kotlin
        val vto = layout.viewTreeObserver
        vto.addOnGlobalLayoutListener(object: ViewTreeObserver.OnGlobalLayoutListener 		{
            @RequiresApi(Build.VERSION_CODES.JELLY_BEAN)
            override fun onGlobalLayout(){
                layout.viewTreeObserver.removeOnGlobalLayoutListener(this)
                val height = layout.measuredHeight
                val width = layout.measuredWidth
            }
        })
```



