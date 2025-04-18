# Java ArrayList 和 LinkedList 的区别
- 两者都实现了 java.util.List 列表接口，用于存储元素的数据结构

## ArrayList
- ArrayList 是基于数组（利用 Object[] elementData 数组存储数据）的数据结构，用于实现一个动态数组（由于数组本身实现增删元素操作比较麻烦，而 ArrayList 的实现恰恰可以看成是对数组操作的一种封装），具有高效随机访问元素、动态扩容等特点
- 支持通过索引直接快速访问元素，时间复杂度为 O(1)，但是在中间插入和删除元素操作时（需要移动后续元素），时间复杂度为 O(n)
- 动态大小：无需预先指定大小，可动态地调整大小
- 有序性：元素按添加的顺序存储
- 非线程安全：可以改用 CopyOnWriteArrayList 或 SynchronizedList 保证线程安全（虽然 Vector 也是线程安全的，但效率较低，不推荐使用）
- 扩容：默认初始容量为 10，当元素数量超过容量时会进行自动扩容操作，扩容机制是将当前容量增加到原来的 1.5 倍，新建一个新的数组，并将旧数组中的元素复制到新数组中（基于数组拷贝），所以在能够预估数据量时，可按需通过构造函数指定初始容量，避免频繁扩容操作

```java
//操作元素
- 使用 add 插入指定的元素
- 使用 remove 移除指定元素
- 使用 get 返回指定位置的元素
- 使用 set 设置替换列表中指定位置的元素
```

## LinkedList
- LinkedList 是基于 Doubly Linked List 双向链表的数据结构，同时实现了 java.util.Deque 接口（支持双端队列的操作），通过链表结构优化了插入和删除操作性能，但随机访问性能较差
- 每个 Node 节点需额外存储指向前一个（prev 前驱）和后一个（next 后继）节点的引用，因此内存占用相对较高
- 支持高效的插入和删除操作（在头尾操作，如果是中间的话，需要先找到指定位置的节点，时间复杂度为 O(n)），时间复杂度为 O(1)，而通过索引随机访问（查询，需遍历）操作的时间复杂度为 O(n)
- 动态大小：无需预先定义容量，可动态扩展
- 有序性：元素按照的插入顺序进行存储
- 非线程安全：可以改用 SynchronizedList 保证线程安全

```java
//操作元素
使用 add 在指定位置插入元素
使用 addFirst 方法在列表头部添加元素
使用 addLast 方法在列表末尾添加元素  
使用 get 方法获取指定位置的元素
使用 getFirst 方法获取列表的第一个元素
使用 getLast 方法获取列表的最后一个元素
使用 set 修改指定位置的元素
使用 remove 删除指定位置的元素
使用 removeFirst 删除列表的第一个元素
使用 removeLast 删除列表的最后一个元素
```

## 总结
- ArrayList 适用于需要频繁随机访问元素，而插入和删除操作较少的场景
- LinkedList 适用于需要频繁在头尾进行插入和删除操作、而随机访问较少的场景（因为 LinkedList 访问元素需要从头或尾遍历）
- 当需要一个栈、队列或双端队列时，LinkedList 是一个合适的选择，因为它实现了 Deque 接口，支持需要快速访问链表头尾元素
