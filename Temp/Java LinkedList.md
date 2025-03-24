# Java LinkedList
- LinkedList 是基于 Doubly Linked List 双向链表的数据结构，实现了 java.util.List 和 java.util.Deque 接口（支持列表操作和双端队列操作），通过链表结构优化了插入和删除操作性能，但随机访问性能较差
- 每个 Node 节点需额外存储指向前一个（prev 前驱）和后一个（next 后继）节点的引用，因此内存占用相对较高
- 支持高效的插入和删除操作（在头尾操作，如果是中间的话，需要先找到指定位置的节点，时间复杂度为 O(n)），时间复杂度为 O(1)，而通过索引随机访问（查询，需遍历）操作的时间复杂度为 O(n)
动态大小：无需预先定义容量，可动态扩展
有序性：元素按照的插入顺序进行存储
非线程安全：可以改用 SynchronizedList 保证线程安全

## 操作元素
- 使用 add 在指定位置插入元素
- 使用 addFirst 方法在列表头部添加元素
- 使用 addLast 方法在列表末尾添加元素  
- 使用 get 方法获取指定位置的元素
- 使用 getFirst 方法获取列表的第一个元素
- 使用 getLast 方法获取列表的最后一个元素
- 使用 set 修改指定位置的元素
- 使用 remove 删除指定位置的元素
- 使用 removeFirst 删除列表的第一个元素
- 使用 removeLast 删除列表的最后一个元素

## LinkedList 和 ArrayList
- LinkedList 适用于需要频繁在头尾进行插入和删除操作、较少随机访问的场景，而对于需要频繁随机访问元素的场景（因为 LinkedList 访问元素需要从头或尾遍历）来说 ArrayList 可能更合适
- 当需要一个栈、队列或双端队列时，LinkedList 是一个合适的选择，因为它实现了 Deque 接口，支持需要快速访问链表头尾元素

