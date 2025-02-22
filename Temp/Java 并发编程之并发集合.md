# Java 并发编程之并发集合


## 并发集合类
- ConcurrentHashMap 线程安全的哈希表实现，相比于传统的 HashMap 通过 synchronized 关键字来保证同步，它采用了更复杂的分段锁等机制，在高并发情况下能够提供更好的性能，支持多个线程同时进行部分读操作和一定条件下的写操作，允许并发地插入、删除和检索元素等操作

CopyOnWriteArrayList 一个线程安全的 List 实现，其原理是在对列表进行修改操作（如添加、删除元素）时，会先复制一份然后在复制的副本上进行修改，最后再用修改后的副本替换原列表，而读操作就可以在原列表上并发进行，所以适合读多写少的场景，能保证线程安全且读取速度较快

CopyOnWriteArraySet 基于 CopyOnWriteArrayList 实现的线程安全的 Set 类型，利用了 CopyOnWriteArrayList 的特性来保证集合元素的唯一性以及在多线程环境下的操作安全性



## List
CopyOnWriteArrayList


## Set
CopyOnWriteArraySet

## Queue

### BlockingQueue

ArrayBlockingQueue 基于数组的有界阻塞队列
DelayQueue 支持延迟元素获取的无界阻塞队列
LinkedBlockingQueue 基于链表的可选有界阻塞队列
LinkedTransferQueue 线程安全的无界传输队列，基于链表实现
PriorityBlockingQueue 支持优先级排序的无界阻塞队列
SynchronousQueue 不存储元素的阻塞队列，每个插入操作必须等待另一个线程的对应移除操作

BlockingDeque 接口继承自 BlockingQueue 接口
TransferQueue 接口继承自 BlockingQueue 接口

LinkedBlockingDeque

## Deque
### BlockingDeque


## Map
ConcurrentHashMap



BlockingQueue
ConcurrentLinkedQueue 线程安全的无界非阻塞队列，基于链表实现
ConcurrentLinkedDeque 线程安全的无界双向队列，基于链表实现