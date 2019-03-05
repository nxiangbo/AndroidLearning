# HashMap源码分析



HashMap 的底层实现是链表散列。key-value,key允许为null，value允许为null。
HashMap使用`Entry`存储键值对信息。使用链表法解决冲突问题。

![](https://leanote.com/api/file/getImage?fileId=577a6fe5ab64414433000746)

     static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        final int hash;
    
        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
    }

```java
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable

```

1. HashMap 实现了Map接口
2. 实现了Cloneable接口表明支持clone操作
3. 实现了Serializable接口表明支持序列化。

#####成员变量

```java

//默认容量为16，它是指hash表的桶的个数，容量大小一定是2^n。
    static final int DEFAULT_INITIAL_CAPACITY = 16;

     //HashMap的容量上限，最大值为2^30
    static final int MAXIMUM_CAPACITY = 1 << 30;

   //负载因子和容量是影响HashMap性能的重要因素，后面后

     //默认的负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

	//存储键值对等信息，它的长度为2^n
    transient Entry[] table;

     //键值对的个数
    transient int size;

	//在hash表扩容时的阈值，threshold = capacity*loadfactor
    int threshold;

	//负载因子
    final float loadFactor;
```

#####构造方法

```java
//initialCapacity为hash表桶的个数，loadFactor是负载因子
public HashMap(int initialCapacity, float loadFactor) {
    	//初始容量不能小于0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
		//初始容量不能大于最大值
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
		//负载因子不能小于0
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        //由此可知，HashMap的容量都是2的次幂
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        this.loadFactor = loadFactor;
		//设置阈值，当键值对的个数大于capacity * loadFactor时，扩容
        threshold = (int)(capacity * loadFactor);
		//capacity大小的Entry数组
        table = new Entry[capacity];
        init();
    }

public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
    }


 public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        putAllForCreate(m);
    }

```

##成员方法
put()方法

```java
 public V put(K key, V value) {
        //如果key为null，则找到键值为null的Entry，将value放入该entry中。
        if (key == null)
            return putForNullKey(value);
		
        int hash = hash(key.hashCode());
		//找到key应该放在哪个桶里
        int i = indexFor(hash, table.length);
		//遍历桶table[i]中的列表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
			//如果存在键为key的值，则将旧的替换为新的
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
		//如果不存在键为key的值，将其放入桶中
        addEntry(hash, key, value, i);
        return null;
    }

 private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
    
static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        //为什么这样写??
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
    
    //h mod length;
static int indexFor(int h, int length) {
        return h & (length-1);
    }
//将键为key，值为value的元素放入相应的桶中。
void addEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        //table[bucketIndex].next = e 插入到链表中
        table[bucketIndex] = new Entry<>(hash, key, value, e);
		//size+1 如果size大于等于阈值，则容量扩大两倍。
        if (size++ >= threshold)
            resize(2 * table.length);
    }
//重新计算容量
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
		//将当前table复制到newTable中
        transfer(newTable);
        table = newTable;
		//更新阈值
        threshold = (int)(newCapacity * loadFactor);
    }
    
 //将当前的Entry数组复制到新newTable中
    void transfer(Entry[] newTable) {
        Entry[] src = table;
        int newCapacity = newTable.length;
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;
                do {
                    Entry<K,V> next = e.next;
                    int i = indexFor(e.hash, newCapacity);
                    e.next = newTable[i];
                    newTable[i] = e;
                    e = next;
                } while (e != null);
            }
        }
    }
```

get()方法

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
	//key的hashCode值进行hash
    int hash = hash(key.hashCode());
	//找到相应的bucket，遍历该bucket的链表
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
	// 如果没有找到，则返回null值
    return null;
}
```


​    
```java
//获取键为null的value
private V getForNullKey() {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
```
 remove(Object key)方法


​    
```java
//删除键为key的值
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}

final Entry<K,V> removeEntryForKey(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
			//链表的删除操作
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
```

containsKey(Object key)方法

```java
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}

final Entry<K,V> getEntry(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

containsValue(Object value)方法

```java
//查找value
public boolean containsValue(Object value) {
    if (value == null)
        return containsNullValue();

    Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (value.equals(e.value))
                return true;
    return false;
}

/**
 * Special-case code for containsValue with null argument
 */
private boolean containsNullValue() {
    Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (e.value == null)
                return true;
    return false;
}
```


​    
​    
​    