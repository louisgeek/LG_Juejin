# Android SimplePool 和 SynchronizedPool 的区别
- SimplePool 简单池和 SynchronizedPool 同步池都属于对象池的实现，都是 android.core.util.Pools 类的一部分，主要用于对象的复用，以此减少对象频繁创建和销毁带来的性能开销（避免频繁触发 GC 操作）
- SimplePool 和 SynchronizedPool 是基于数组实现的（内部使用一个数组来存储对象）
- PS：Android Message 类的 Message Pool 是基于链表实现的（通过 Message#obtain 复用消息），Glide 内部的 KeyPool 是基于 Queue 队列实现的，Glide 内部的 BitmapPool 是基于带了 LRU 算法的 Map 实现的

## SimplePool
- SimplePool 是非线程安全的对象池，适用于单线程环境
- SimplePool 继承自 androidx.core.util.Pools.Pool 接口，通过记录当前池中的对象数量，实现对象的入池和出池操作
- 对象池的大小在创建时指定，无法动态调整，按需求设置 maxPoolSize，过大的池会浪费内存，过小则无法发挥复用优势
```java
public class MyPooledObject {
    private static final SimplePool<MyPooledObject> sPool = new SimplePool<>(5);

    public static MyPooledObject obtain() {
        //从对象池中获取对象
        MyPooledObject instance = sPool.acquire(); //获取
        //如果池中没有可用对象就创建一个新的
        return (instance != null) ? instance : new MyPooledObject();
    }

    public void recycle() {
        //将对象放回对象池（存到数组末尾），如果池已满，对象会被丢弃
        sPool.release(this);
    }
}
```

## SynchronizedPool
- SynchronizedPool 是线程安全的对象池，使用 synchronized 关键字来保证线程安全（在 acquire 和 release 方法上加锁来实现）
- SynchronizedPool 继承自 SimplePool，是 SimplePool 的同步版本

```java
public class MySyncPooledObject {
    private static final SynchronizedPool<MySyncPooledObject> sPool = new SynchronizedPool<>(5);

    public static MySyncPooledObject obtain() {
        MySyncPooledObject instance = sPool.acquire();
        return (instance != null) ? instance : new MySyncPooledObject();
    }

    public void recycle() {
        sPool.release(this);
    }
}
```