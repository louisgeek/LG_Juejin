# Java HashMap 和 TreeMap 的区别
- 两者都实现了 java.util.Map 映射接口，用于存储 key-value 键值对的数据结构

## HashMap
- HashMap 是基于 Hash Table 哈希表（又叫散列表）实现的键值对存储结构
- 键唯一，值可重复，允许使用 null 键和 null 值
- 底层实现基于哈希表，内部结构是数组 + 链表/红黑树，支持高效的增删查操作特性，查找、插入和删除操作的时间复杂度均为 O(1)，空间复杂度为 O(n)，其中 n 为键值对的数量
- 无序性：不保证元素的插入顺序，也不保证顺序恒定不变，需有序可改用 LinkedHashMap
- 非线程安全：可以改用 ConcurrentHashMap 或 SynchronizedMap 保证线程安全（虽然 Hashtable 也是线程安全的，但已被弃用，不推荐使用）

## TreeMap
- TreeMap 是基于 Red-Black Tree 红黑树（一种自平衡的二叉查找树，每个节点包含键、值、父节点、左右子节点及颜色标识信息，因此内存占用相对较高）实现的有序的键值对存储结构
- 不允许使用 null 键，不过值可以为 null
- 支持较高效的查找、插入和删除操作，时间复杂度均为 O(log n)，比 HashMap 的操作稍慢，空间复杂度为 O(n)，其中 n 为键值对的数量
- 范围查询：支持 subMap，headMap，tailMap 等方法快速获取指定范围的键值对
- 有序性：元素按照 key 键进行有序存储，默认情况下按照键的自然顺序进行排序
- 非线程安全：可以改用 ConcurrentSkipListMap 和 SynchronizedSortedMap 保证线程安全

## 总结
- HashMap 适用于不需要元素有序，只需要快速查找、插入和删除元素的场景，查找、插入和删除操作的时间复杂度均为 O(1)，哈希冲突时可能退化为 O(n)
- TreeMap 适用于需要按照键排序、​有序遍历或需要范围查询的场景，查找、插入和删除操作的时间复杂度均为 O(log n)
- 通常情况下 HashMap 比 TreeMap 性能要更好