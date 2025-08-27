# Java ConcurrentHashMap 和 SynchronizedMap 的区别
- java.util.concurrent.ConcurrentHashMap 和 java.util.Collections$SynchronizedMap 两者都是 Java 中线程安全的 Map 实现


## ConcurrentHashMap
- ConcurrentHashMap 是专门为并发环境设计的，性能优势显著，尤其在读多写少或高并发写场景，适用于绝大多数高并发场景（比如缓存、实时计数器和高频数据统计等）
- ConcurrentHashMap 的 key 和 value 均不允许为 null
- 提供 putIfAbsent、computeIfAbsent 和 replace 等原子方法，避免手动同步的复杂性

## Collections$SynchronizedMap
- SynchronizedMap 通过包装一个普通的 Map（比如 HashMap）并对其操作进行同步的方式来实现的，大部分的读写操作都用 synchronized 关键字（对象锁）来保证线程安全，而迭代方法没有加锁，所以在使用迭代方法的时候必须要手动处理线程同步问题（避免抛出异常），由于读写操作都需要竞争同一把锁，所以高并发下性能显著下降
- 常用方法 Collections.synchronizedMap 返回一个 SynchronizedMap 对象
- SynchronizedMap 比较适合需要简单地将普通非线程安全的 Map 转换为线程安全的 Map 的情况，适用于旧代码兼容场景，需避免在高并发环境中使用
- 复合操作需要额外同步逻辑

```java
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
//需要手动加锁才能保证原子性
synchronized (synchronizedMap) { 
    if (!synchronizedMap.containsKey(key)) {
        synchronizedMap.put(key, value);
    }
}
```

```java
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
//迭代时必须处理同步
synchronized (synchronizedMap) { 
    for (Map.Entry<String, Integer> entry : synchronizedMap.entrySet()) {
        System.out.println(entry.getKey() + ": " + entry.getValue());
    }
}
```

## 总结
- 迭代器：ConcurrentHashMap 是弱一致性的，迭代过程中允许其他线程修改 Map，不会抛出异常，但迭代器可能不会反映最新的修改（仅保证迭代时的数据是线程安全的），而 SynchronizedMap 在迭代过程中如果被其他线程修改时（比如 put 或 remove），会立即抛出异常
- 空值处理：ConcurrentHashMap 禁止 null 键或值（避免歧义，为了避免出现 get 方法返回 null 无法区分是 key 不存在还是 value 为 null 的情况），而 SynchronizedMap 允许 null 键和值（沿用底层 Map 特性）
- 锁机制：ConcurrentHashMap 只对需要修改的部分进行加锁，而不是整个 Map，而 SynchronizedMap 使用 synchronized 关键字对整个 Map 对象加锁，读写操作都需要获取全局锁，性能较低
- 适用场景：ConcurrentHashMap 适用于高并发、高性能要求场景，特别是读多写少的情况，而 SynchronizedMap 则适用于并发程度较低且对性能要求不高的场景，实现简单，兼容性好，如果需要支持 null 键或值，可以选用 SynchronizedMap，如果需要保证迭代操作的强一致性，可以考虑使用 SynchronizedMap 并手动加锁
