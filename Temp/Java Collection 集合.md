# Java Collection 集合



## java.util.Collection
- 集合类的上层接口，实现 Collection 接口的主要有 Set、List 和 Map






## java.util.Collections
- 针对集合类的一个工具类，提供一系列静态方法，实现对集合进行搜索、排序、线程安全化相关操作等功能









- 一个 Java 对象可以在内部持有若干个其他 Java 对象，并且对外提供访问的接口



数组也可以看作是一种集合

- 但是数组初始化后大小就固定了，不可变了
- 只能按照索引取数据



所以需要一些功能更加强大的集合：List，Set，Map 等



List

- 有顺序的列表集合，元素可以重复，可以添加 null



Set

- 元素唯一无重复的集合



Map

- 通过键值对形成的映射表集合，key 不会重复，value 可以重复，没有顺序





由于长期的迭代和历史原因，有些集合类不推荐使用：Hashtable，Vector，Stack 等





Collections 集合工具类的 Collections#SynchronizedMap

并发包下的 ConcurrentHashMap





### Collections#SynchronizedSet




# Collection 集合



Collection 接口

- List
  - java.util.ArrayList
  - Vetor
  - LinkedList
  - CopyOnWriteArrayList

- Set
  - HashSet
  - LinkedHashSet
- TreeSet
  
  
  
  ！！！！
  
- java.util.Collections.SynchronizedCollection
  - java.util.Collections.SynchronizedList
    - java.util.Collections.SynchronizedRandomAccessList
  
- java.util.Collections.SynchronizedSet



## List

- 不推荐 for 和 get 进行循环变量
- 推荐用 Iterator 迭代器进行遍历
- 可以直接使用 for each 遍历 List，Java 编译器会自动改用 Iterator 

| List     | ArrayList          | Vector         | CopyOnWriteArrayList | LinkedList         |
| -------- | ------------------ | -------------- | -------------------- | ------------------ |
| 底层实现 | 数组               | 数组           |                      | 链表               |
| 线程安全 | 线程不安全         | 线程安全       |                      |                    |
| 效率     | 效率高             | 效率低         |                      |                    |
|          | 查快，增删慢  改？ | 查快，增删改慢 |                      | 查慢，增删快  改？ |
| 增加     |                    |                |                      |                    |
| 删除     |                    |                |                      | 快                 |
| 修改     |                    |                |                      |                    |
| 查找     | 快                 | 快             |                      | 慢                 |





## Set





| Set      | HashSet        | LinkedHashSet  | TreeSet        |
| -------- | -------------- | -------------- | -------------- |
| 线程安全 | 线程不安全     | 线程安全       |                |
| 效率     | 效率高         | 效率低         |                |
| 增删改查 | 查快，增删改慢 | 查快，增删改慢 | 查慢，增删改快 |
| 顺序     | 无序           |                | 有序           |







## Collection#toArray

- ArrayList#toArray

```java
 /**
     * Returns an array containing all of the elements in this list
     * in proper sequence (from first to last element).
     *
     * <p>The returned array will be "safe" in that no references to it are
     * maintained by this list.  (In other words, this method must allocate
     * a new array).  The caller is thus free to modify the returned array.
     *
     * <p>This method acts as bridge between array-based and collection-based
     * APIs.
     *
     * @return an array containing all of the elements in this list in
     *         proper sequence
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * Returns an array containing all of the elements in this list in proper
     * sequence (from first to last element); the runtime type of the returned
     * array is that of the specified array.  If the list fits in the
     * specified array, it is returned therein.  Otherwise, a new array is
     * allocated with the runtime type of the specified array and the size of
     * this list.
     *
     * <p>If the list fits in the specified array with room to spare
     * (i.e., the array has more elements than the list), the element in
     * the array immediately following the end of the collection is set to
     * <tt>null</tt>.  (This is useful in determining the length of the
     * list <i>only</i> if the caller knows that the list does not contain
     * any null elements.)
     *
     * @param a the array into which the elements of the list are to
     *          be stored, if it is big enough; otherwise, a new array of the
     *          same runtime type is allocated for this purpose.
     * @return an array containing the elements of the list
     * @throws ArrayStoreException if the runtime type of the specified array
     *         is not a supertype of the runtime type of every element in
     *         this list
     * @throws NullPointerException if the specified array is null
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

- toArray() 会丢失类型信息
- 如果传入的数组不够大，那么`List`内部会创建一个新的刚好够大的数组，填充后返回；如果传入的数组比`List`元素还要多，那么填充完元素后，剩下的数组元素一律填充`null`



## 

## Collections



### Collections#binarySearch



### Collections#addAll



### Collections#reverse



### empty 空集合

```
List<String> list = Collections.emptyList();
Map<String,String> map = Collections.emptyMap();
Set<String> set = Collections.emptySet();
```

- 空集合是不可变集合



### synchronized

```
Collection<String> synchronizedCollection = Collections.synchronizedCollection(collection);
Map<String,String> synchronizedMap = Collections.synchronizedMap(map);
List<String> synchronizedList = Collections.synchronizedList(list);
Set<String> synchronizedSet = Collections.synchronizedSet(set);
```





Java 官方推荐用 Deque 替代 Stack 


