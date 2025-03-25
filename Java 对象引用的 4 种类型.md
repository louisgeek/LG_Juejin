# Java 对象引用的 4 种类型
- Java 把对象引用的类型划分为了 4 个级别，级别由高到低分别是强引用、软引用、弱引用和虚引用，为了使程序能狗更加灵活地控制对象的生命周期

## Strong Reference 强引用
- 强引用是最常见的引用类型，只要强引用存在，垃圾回收器就永远不会回收被引用的对象，如果想中断强引用与对象之间的联系，可以显式的将强引用赋值为 null
```java
Object obj = new Object(); //obj 是强引用，引用指向对象
//
obj = null; //显式置为 null，原对象才会被垃圾回收
```

## Soft Reference 软引用
- 当在系统内存不足时，垃圾回收器才会回收这些对象（OOM 前最后一次 GC 操作）
- 通常用于实现内存敏感的缓存，缓存较大的对象（比如图片、文件、网页缓存等临时数据），当内存紧张时会被自动回收，避免出现 OOM 内存溢出
```java
SoftReference<Bitmap> softRef = new SoftReference<>(bitmap);
Bitmap bmp = softRef.get(); // 当内存不足时会返回 null
```

## Weak Reference 弱引用
- 对象在下一次垃圾回收时一定会被回收（不管当前内存是否充足），弱引用的强度比软引用要低
- 通常用于实现无状态的临时映射（比如事件监听器），避免出现内存泄漏
```java
WeakReference<Listener> weakRef = new WeakReference<>(listener);
Listener listener = weakRef.get(); // 当下一次 GC 运行后会返回 null
```

## Phantom Reference 虚引用
- 虚引用是最弱的一种引用类型，必须和 ReferenceQueue 引用队列配合使用，当垃圾回收器准备回收时，如果有虚引用就会在回收之前把这个虚引用加入到与之关联的引用队列里去
- 无法通过 get 方法获取对象的实例（始终返回null），仅在对象被回收时通过队列获取回收通知
- 通常仅用于跟踪对象回收状态（比如用于实现后期资源清理机制、监控对象的创建和销毁等）
```java
ReferenceQueue<Object> queue = new ReferenceQueue<>(); //可以通过 queue.poll 检测对象是否被回收
PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue); // 当对象被回收后，phantomRef 会被加入到 ReferenceQueue 中
``` 

## 总结
- 引用类型从强到弱依次为：强引用 -> 软引用 -> 弱引用 -> 虚引用
- 强引用：只要强引用存在，永不主动回收
- 软引用：内存不足时回收
- 弱引用：下一次垃圾回收时回收
- 虚引用：随时可能被回收，对象回收后通过队列通知