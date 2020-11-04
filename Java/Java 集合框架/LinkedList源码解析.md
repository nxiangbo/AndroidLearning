####LinkedList源码解析

LinkedList的本质是双链表。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable

```

1. 实现了List接口表明需要实现List中的set，get等方法；
2. 实现了Deque接口表明LinkedList实现了双端队列的方法；
3. 实现了Clonable接口表明可以被clone；
4. 实现了Serializable接口表明可以被序列化。

#####LinkedList链表结点结构
LinkedList的静态内部类。
```java

//链表节点的结构，双向链表
	private static class Node<E> {
        E item;
		//指向下一个元素
        Node<E> next;
		//指向前一个元素
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

```

#####成员变量

```java

//记录链表的长度
    transient int size = 0;

    /**
     * Pointer to first node.
     * 一定满足: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
     //指向头节点的指针
    transient Node<E> first;

    /**
     * Pointer to last node.
     * 一定满足: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
     //指向尾结点的指针
    transient Node<E> last;
```

#####构造方法

```java
//空的构造方法
 public LinkedList() {
    }

//将集合c中的元素插入到链表中
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }

```

#####成员方法

######插入元素

- 头插法

```java
//在表头插入一个元素(头插法)
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
		//当前链表为空,last指针指向newNode
        if (f == null)
            last = newNode;
        else
			//如果不为空，则将f.prev指向newNode
            f.prev = newNode;
        size++;
        modCount++;
    }

```

- 尾插法

```java

 //在链表尾部插入一个元素
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
		//l==null,表明链表为空
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

- 在某个结点前插入元素

```java

 //在非空的succ结点前插入元素e。双向链表的插入操作
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

```

![](file:///E:/%E6%88%91%E7%9A%84%E6%96%87%E6%A1%A3/blog/Java/Java%20Collection%20Framework/LinkedList_insert.jpg)

######删除元素
- 删除头结点元素

```java

//删除链表的头节点
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
		//next==null,说明删除头结点后，链表为null
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

- 删除链表尾结点元素

```java

//删除尾结点
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

- 删除链表中的任意结点x

```java
 //删除结点x
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
		//如果prev==null，说明x为头结点
        if (prev == null) {
            first = next;
        } else {
        //不是头结点
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }


```

上述链表的插入和删除操作是LinkedList的核心，我们常用的add(),remove()等方法均是调用上述方法完成的。

然后，我们再看LinkedList的get()和set()方法

######get()方法

```java
//获取链表index处的节点
    public E get(int index) {
    	//检查index是否越界
        checkElementIndex(index);
        //找到index位置处的结点，并返回该结点的值
        return node(index).item;
    }

```

######set()方法

```java

 //将index位置的节点的元素设置为element
    public E set(int index, E element) {
        checkElementIndex(index);
        //找到index位置处的结点
        Node<E> x = node(index);
        E oldVal = x.item;
        //将element的值赋给该结点
        x.item = element;
        return oldVal;
    }

```

看完上述源代码，我们会发现，get和set操作都是通过node()方法完成操作的。下面我们看一下node()方法的代码。
node(int index)

```java
//返回index位置处的结点
    Node<E> node(int index) {
        // assert isElementIndex(index);
		//当index<size/2时，从头结点遍历链表
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
        //当index>=size/2时，从尾节点遍历链表
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }


```

######l链表的查找操作

```java
 //链表中是否包含元素o，如果是，返回true，否则返回false
    public boolean contains(Object o) {
    	//调用indexOf()方法
        return indexOf(o) != -1;
    }


 //查找操作，返回元素o在链表中第一次出现的位置，如果不在链表中，则返回-1
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

前面我们说过，LinkedList实现了Deque接口，所以，LinkedList拥有双端队列的操作。

######队列的操作

- 获取队头元素

```java
 //获得头结点数据，允许为null.一般用于获取队头元素
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

//双端队列

//获取队头元素

 //获取队头元素，允许为null
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

//获取队尾元素

//获取队尾元素，允许为null
    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }


```

- 入队操作

```java
 //等价于add(e),一般用于入队操作
    public boolean offer(E e) {
        return add(e);
    }

//双端队列

    //从队头入队

    //等价于addFirst(e)，一般用于从队头入队
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    //从对尾入队

     //等价于addLast(e)，一般用于从队尾入队
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }



```

- 出队操作

```java

//获得头结点数据，并删除头结点，允许为null。一般用于出队操作
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    //双端队列

    //从队头出队

    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

     //从队尾出队
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

```

LinkedList除了集成了队列的功能，还集成了栈的操作。栈的操作比较简单，如下。
#####栈的操作

- 出栈

```java
 //出栈
    public E pop() {
        return removeFirst();
    }
```

- 入栈

```java
//入栈
    public void push(E e) {
        addFirst(e);
    }
```

到此，LinkedList的源码大致分析完毕。by the way，LinkedList也是非线程安全的，同时也使用了fail-fast机制。