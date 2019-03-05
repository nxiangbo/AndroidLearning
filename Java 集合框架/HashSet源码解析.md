# HashSet源码解析

HashSet 是基于`HashMap`实现的，将`Set`的值设置为`HashMap`的键，因为`key`是不可能重复的。`HashSet`实现了`Set`接口，而`Set`接口方法基本与`Collection`方法一致。此外，`HashSet`是无序的。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

## 成员变量

```java
// 存放HashSet的值
private transient HashMap<E,Object> map;

// 用于填充map的value
private static final Object PRESENT = new Object();
```

## 构造方法

```java
//其实质就是实例化map
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
```

## 成员函数

add(E e)方法

```java
 public boolean add(E e) {
    //HashMap的put操作，以e为key， PRESENT为value
    return map.put(e, PRESENT)==null;
}
```

contains(Object o)

```java
public boolean contains(Object o) {
    return map.containsKey(o);
}

public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

 public void clear() {
    map.clear();
}
```


​    
​    
​    

