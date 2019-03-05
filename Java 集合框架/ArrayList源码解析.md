# ArrayList源码解析

ArrayList的本质是动态数组。既然是数组，肯定是可以随机访问的。ArrayList是非线程安全的，当当个线程并发访问同一个ArrayList时，会抛出ConcurrentModificationException,这就是fail-fast机制。

``` java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable

```
由上面代码知


1. 实现List<E>接口表明实现了List接口中的size(),add(),set()等等方法
2. 实现RandomAccess接口表明支持随机访问
3. 实现Cloneable接口表明支持clone（）
4. 实现Serializable接口表明支持序列化。

#### 成员变量

``` java

//数组，由此表明ArrayList是数组
private transient Object[] elementData;

 //记录数组中所含元素的个数
 private int size;
 //数组的最大容量。如果请求更大的容量，可能会导致OutOfMemoryError
 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

```
####构造方法

``` java
//构造一个容量为initialCapacity的List
    public ArrayList(int initialCapacity) {
        super();
		//initialCapacity<0时，抛出参数异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
		
		//其本质是创建一个长度为initialCapacity的数组
        this.elementData = new Object[initialCapacity];
    }

   //默认容量为10
    public ArrayList() {
        this(10);
    }

    //构造一个内容为集合c的List
    public ArrayList(Collection<? extends E> c) {
    	//将c转化为数组，赋值给elementData数组
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        //如果当前c.toArray不是Object类型的，则将其拷贝为Object类型的
        if (elementData.getClass() != Object[].class)
        //数组拷贝
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }

```

####成员方法
我们最常用的就是add()和set()方法，我们先来看这两个方法

#####添加 时间复杂度为O(1)

add(E e)

```java

//在数组末尾添加一个元素
    public boolean add(E e) {
    	//确保数组增加元素后，数组有足够的容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //在size位置处添加e
        elementData[size++] = e;
        return true;
    }


```
添加元素时，会首先调用ensureCapacityInternal(minCapacity)

ensureCapacityInternal(int minCapacity)

```java

private void ensureCapacityInternal(int minCapacity) {
        modCount++;
        // overflow-conscious code
        //如果所需的容量大于当前数组的容量，那么，需要增加数组的容量，调用grow(int minCapacity)
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

     //扩大数组容量为原来的1.5倍
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果扩充后的容量大于MAX_ARRAY_SIZE，调用hugeCapacity()
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //数组拷贝，将数组拷贝到扩大容量后的数组中，耗时较长，尽量避免。
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
   	//当minCapacity<0时，一个内存数组的分配不能被创建，会抛出OutOfMemoryError
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }


```

在上述代码中，我们可以看到，当数组容量不够时，数组有一个扩容的过程，在扩容的过程中，会将原来数组的元素拷贝到新的数组中，这是一个很耗时的操作。由此可见，动态数组(ArrayList)在使用方便的同时，也会承担降低性能的风险。当然，如果我们在用ArrayList时，知道所需的容量，可以构造所需容量的ArrayList。

**注意**：在ArrayList中有两个情况可以导致OutOfMemoryError

1. 当minCapacity<0时，系统无法创建长度小于0 的数组；
2. 当数组容量超过VM中堆的剩余空间大小时，VM无法为其分配足够的内存。

add(int index, E element)

```java

 //在数组的特定位置插入一个元素
    public void add(int index, E element) {
    	//检查index是否越界
        rangeCheckForAdd(index);
		
		//确保数组有足够的容量，去存储新的数据
        ensureCapacityInternal(size + 1);  // Increments modCount!!

		//将elementData数组中index...size-1处的元素拷贝到index+1...size-index位置处。即从index处的元素向后移动一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }


```

在这里，ArrayList的add()操作时间复杂度为O(1)。确切的说应该是amortised constant time。可能你会认为，在数组扩充容量的时，需要把旧的数组拷贝到新的数组中，时间复杂度应该是O(N)。但是，你要知道，执行add很多次时，不能只关心它的最坏和最好的情况，你需要关心的是重复上百万次的总的时间。amortised constant time的意思就是说，你做了很多次操作后，每次操作的平均时间。
下面引用[StackOverFlow](http://stackoverflow.com/questions/200384/constant-amortized-time)上面的解释。

>Amortised constant time 

>If you do an operation say a million times, you don't really care about the worst-case or the best-case of that operation - what you care about is how much time is taken in total when you repeat the operation a million times.

>So it doesn't matter if the operation is very slow once in a while, as long as "once in a while" is rare enough for the slowness to be diluted away. Essentially amortised time means "average time taken per operation, if you do many operations". Amortised time doesn't have to be constant; you can have linear and logarithmic amortised time or whatever else.

>Let's take mats' example of a dynamic array, to which you repeatedly add new items. Normally adding an item takes constant time (that is, O(1)). But each time the array is full, you allocate twice as much space, copy your data into the new region, and free the old space. Assuming allocates and frees run in constant time, this enlargement process takes O(n) time where n is the current size of the array.

>So each time you enlarge, you take about twice as much time as the last enlarge. But you've also waited twice as long before doing it! The cost of each enlargement can thus be "spread out" among the insertions. This means that in the long term, the total time taken for adding m items to the array is O(m), and so the amortised time (i.e. time per insertion) is O(1).

#####修改 时间复杂度为O(N)

``` java

//将index处的元素更改为element
    public E set(int index, E element) {
    	//判断index是否越界
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

```

#####删除 时间复杂度为O(N)
remove(int index)

```java
//删除特定位置的元素
    public E remove(int index) {
    	//判断index是否越界
        rangeCheck(index);
		
        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work

        return oldValue;
    }

```

remove(Object o)

```java
//删除某个元素
    public boolean remove(Object o) {
        //由此可知，ArrayList允许元素为null
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
				//如果数组中存在o，则删除
                if (o.equals(elementData[index])) {
					//删除index位置的元素
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work
    }


//删除所有元素
public void clear() {
        modCount++;

        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

#####查找

```java
//如果元素o在数组中，则返回其下标，否则，返回-1.
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    //返回元素o在数组中的最后出现的下标，若数组中不存在元素o，则返回-1
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    //如果数组中包含元素o，则返回true，否则，返回false
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }


```

####Fail-Fast机制
Java容器中有一种保护机制，能够防止多个进程同时修改同一个容器的内容。如果在你迭代遍历某个容器的过程中，另一个进程介入其中，并且插入、修改或者删除此容器内的某个对象，那么就会出现各种奇怪的问题。Java容器类使用fail-fast机制。它会探查容器上除了当前进程所进行的操作以外的所有的变化，一旦发生变化，就会立刻抛出ConcurrentModificationException异常。在程序中，使用modCount变量记录容器更改的次数。

```java
	final void checkForComodification() {
	//如果modCount与expectedModCount不相等，就抛出ConcurrentModificationException异常
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```