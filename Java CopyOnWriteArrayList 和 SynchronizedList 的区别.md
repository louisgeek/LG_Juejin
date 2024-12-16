# Java CopyOnWriteArrayList 和 SynchronizedList 的区别
- java.util.concurrent.CopyOnWriteArrayList 和 java.util.Collections$SynchronizedList 两者都是 Java 中线程安全的 List 实现
 
## CopyOnWriteArrayList
- CopyOnWrite 代表写时复制，在执行写操作（如添加、删除或修改元素）时，会复制一份新数组进行相关的操作，然后在操作完成后将原来的集合指向新的集合
- 由于读操作在原数组上进行，所以读操作不需要加锁，所以读操作比较高效，而写操作内存占用开销比较大，比较适合读多写少的情况

```java
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    //...
    /**
     * The lock protecting all mutators.  (We have a mild preference
     * for builtin monitors over ReentrantLock when either will do.)
     */
    final transient Object lock = new Object(); //保证不会有多个线程同时复制一个新数组

    /** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array; //volatile 关键字保证了变量的可见性

    /**
     * Gets the array.  Non-private so as to also be accessible
     * from CopyOnWriteArraySet class.
     */
    final Object[] getArray() {
        return array;
    }

    /**
     * Sets the array.
     */
    final void setArray(Object[] a) {
        array = a;
    }

    /**
     * Creates an empty list.
     */
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }

    /**
     * Creates a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection of initially held elements
     * @throws NullPointerException if the specified collection is null
     */
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] es;
        if (c.getClass() == CopyOnWriteArrayList.class)
            es = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            es = c.toArray();
            // Android-changed: Defend against c.toArray (incorrectly) not returning Object[]
            //                  (see b/204397945)
            // if (c.getClass() != java.util.ArrayList.class)
            if (es.getClass() != Object[].class)
                es = Arrays.copyOf(es, es.length, Object[].class);
        }
        setArray(es);
    }
    //...
    /**
     * Returns the number of elements in this list.
     *
     * @return the number of elements in this list
     */
    public int size() {
        return getArray().length;
    }
    
    public E get(int index) {
        return elementAt(getArray(), index);
    }

    /**
     * Replaces the element at the specified position in this list with the
     * specified element.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        synchronized (lock) {
            Object[] es = getArray();
            E oldValue = elementAt(es, index);

            if (oldValue != element) {
                es = es.clone();
                es[index] = element;
            }
            // Ensure volatile write semantics even when oldvalue == element
            setArray(es);
            return oldValue;
        }
    }

    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        synchronized (lock) {
            Object[] es = getArray();
            int len = es.length;
            es = Arrays.copyOf(es, len + 1);
            es[len] = e;
            setArray(es);
            return true;
        }
    }

    /**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).  Returns the element that was removed from the list.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        synchronized (lock) {
            Object[] es = getArray();
            int len = es.length;
            E oldValue = elementAt(es, index);
            int numMoved = len - index - 1;
            Object[] newElements;
            if (numMoved == 0)
                newElements = Arrays.copyOf(es, len - 1);
            else {
                newElements = new Object[len - 1];
                System.arraycopy(es, 0, newElements, 0, index);
                System.arraycopy(es, index + 1, newElements, index,
                                 numMoved);
            }
            setArray(newElements);
            return oldValue;
        }
    }

    /**
     * Removes all of the elements from this list.
     * The list will be empty after this call returns.
     */
    public void clear() {
        synchronized (lock) {
            setArray(new Object[0]);
        }
    }

    /**
     * Returns an iterator over the elements in this list in proper sequence.
     *
     * <p>The returned iterator provides a snapshot of the state of the list
     * when the iterator was constructed. No synchronization is needed while
     * traversing the iterator. The iterator does <em>NOT</em> support the
     * {@code remove} method.
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    //...
}
```




## Collections$SynchronizedList
- 通过包装一个普通的 List 来实现，大部分的读写操作都用 synchronized 关键字（对象锁）来保证线程安全，而迭代方法没有加，所以在使用迭代方法的时候需要自行处理线程同步问题，由于读操作也被加锁了，所以性能上受到了一定影响
- 常用方法 Collections.synchronizedList 返回一个 SynchronizedList 对象
- SynchronizedList 比较适合需要简单地将普通非线程安全的 List 转换为线程安全的 List 的情况

```java
//java.util.Collections
public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ? new SynchronizedRandomAccessList<>(list) : new SynchronizedList<>(list));
}
//Collections 的静态内部类 SynchronizedList
static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {
    //...
    @SuppressWarnings("serial") // Conditionally serializable
    final List<E> list;
    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    //...
    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }
    //...
    public ListIterator<E> listIterator() {
        return list.listIterator(); // Must be manually synched by user
    }
    //...
}
//Collections 的静态内部类 SynchronizedCollection
static class SynchronizedCollection<E> implements Collection<E>, Serializable {
    //...
    @SuppressWarnings("serial") // Conditionally serializable
    final Collection<E> c;  // Backing Collection
    @SuppressWarnings("serial") // Conditionally serializable
    final Object mutex;     // Object on which to synchronize   
    SynchronizedCollection(Collection<E> c) {
        this.c = Objects.requireNonNull(c);
        mutex = this;
    }
    //...
    public int size() {
        synchronized(this.mutex) {
            return this.c.size();
        }
    } 
    //...
    //使用 iterator 遍历的时候仍然需要手动加锁
    public Iterator<E> iterator() {
        return c.iterator(); // Must be manually synched by user!
    }   
    public boolean add(E e) {
        synchronized (mutex) {return c.add(e);}
    }
    public boolean remove(Object o) {
        synchronized (mutex) {return c.remove(o);}
    }
    //...
    public void clear() {
        synchronized (mutex) {c.clear();}
    }
    //...
}
```

使用
```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
//...
//需要自行处理 iterator 方法的线程同步问题
synchronized (list) {
    Iterator<String> iterator = list.iterator(); // Must be in synchronized block
    while (iterator.hasNext()) {
        String str = iterator.next();
        Log.e("TAG", "main: str=" + str);
    }
}
```

## 总结
- 并发性能：由于 CopyOnWriteArrayList 读操作上没有加锁，所以 CopyOnWriteArrayList 读操作性能更好，而 SynchronizedList 由于读写操作都加了锁，所以读写都会受到一定影响
- 内存占用：由于 CopyOnWriteArrayList 写操作会进行数组复制，所以 CopyOnWriteArrayList 在写操作上内存占用开销比 SynchronizedList 要大，所以不适合非常频繁写操作
- 适用场景：CopyOnWriteArrayList 适用于读多写少的场景，而 SynchronizedList 则适用于读写操作相对均衡或者写操作相对较频繁的场景

 