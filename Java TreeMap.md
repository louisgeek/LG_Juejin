# Java TreeMap
- TreeMap 是基于 Red-Black Tree 红黑树（一种自平衡的二叉查找树，每个节点包含键、值、父节点、左右子节点及颜色标识信息，因此内存占用相对较高）实现的有序的 key-value 键值对存储结构，实现了 java.util.Map 映射接口
- 不允许使用 null 键，不过值可以为 null
- 支持较高效的查找、插入和删除操作，时间复杂度均为 O(log n)，比 HashMap 的操作稍慢，空间复杂度为 O(n)，其中 n 为键值对的数量
- 范围查询：支持 subMap，headMap，tailMap 等方法快速获取指定范围的键值对
- 有序性：元素按照 key 键进行有序存储，默认情况下按照键的自然顺序进行排序
- 非线程安全：可以改用 ConcurrentSkipListMap 和 SynchronizedSortedMap 保证线程安全

## 操作元素
- 使用 put 方法插入键值对
- 使用 get 方法根据键获取对应的值
- 使用 remove 方法根据键删除对应的键值对
- 使用 entrySet 遍历键值对

## TreeMap 和 HashMap
- TreeMap 适用于需要按照键排序、​有序遍历或需要范围查询的场景，而对于不需要元素有序，只需要快速查找、插入和删除元素的场景来说 HashMap 可能更合适