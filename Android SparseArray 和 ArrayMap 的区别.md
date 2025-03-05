# Android SparseArray 和 ArrayMap 的区别
- 两者都是 Android 提供的适用于在中小型数据量的场景下来替代 HashMap 的数据结构，在内存占用和数据访问效率上有一定的优势，在数据量较小（千级以内）的场景下通常比 HashMap 更有效率，不过查询效率要低于 HashMap
- SparseArray 和 ArrayMap 都是通过数组实现的，避免了自动装箱和拆箱操作，内存使用效率比 HashMap 要高，在节省内存的同时兼顾一定的查找效率
 
## SparseArray 稀疏数组
- SparseArray 使用 int 类型数组存储 Key 键，Object 类型数组存储 Value 值（键数组和值数组的长度相同，用同一个索引来对应键值对），无哈希表结构，无额外的对象内存的开销（比如 HashMap 的 Entry）
- 排列顺序：键数组的顺序是按照键的自然顺序从小到大升序排列的，便于通过二分查找法查找定位键值对
- 延迟回收：删除元素时仅赋值标记为 DELETED，后续通过插入操作或主动调用 SparseArray#gc 来触发数组压缩，减少频繁扩容拷贝开销，非常适合需要频繁增删的场景
- SparseIntArray 用于 (int, int)，SparseLongArray 用于 (int, long)，SparseBooleanArray 用于 (int, boolean)
- LongSparseArray 用于 (long, Object)，支持使用 long 类型数组存储 Key 键

## ArrayMap 数组映射
- ArrayMap 使用 int 类型数组存储键的哈希值，Object 类型数组交替存储键和值（键值数组长度是哈希值数组的两倍，偶数位存键，奇数位存值），支持任意类型的键（包括 null），无额外的对象内存的开销（比如 HashMap 的 Entry），同时 ArrayMap 实现了 java.util.Map 接口，比 SparseArray 要多费一点内存
- 排列顺序：哈希值数组按照哈希值的自然排序从小到大升序排列的，同样便于通过二分查找实现查找检索
- 缓存机制：ArrayMap 引入了缓存机制，避免频繁创建对象时的内存申请和 GC 操作
- SimpleArrayMap 是一个更轻量级的实现，未实现 java.util.Map 接口

## 总结
- SparseArray 和 ArrayMap 适用于存储小到中等规模的数据两，当数据量较大时（超过千级），推荐改用 HashMap 更合适，如果对查找速度有较高要求，则应选择 HashMap，如果内存占用更重要，则可以考虑选择 SparseArray 或 ArrayMap
- SparseArray 适合场景：当键为 int 类型，且对内存使用要求较高时，特别是涉及频繁的插入和删除操作（比如缓存 View 视图对象、存储 ID 对应的用户信息和存储 ID 对应的列表项对象）
- ArrayMap 适合场景：如果键需要是任意类型时 （比如配置参数信息存储），由于 ArrayMap 需维护哈希值，且需要对哈希值数组进行排序，当数据量不断增加时，内存开销会逐渐增加，性能下降明显


