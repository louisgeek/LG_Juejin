# Java 并发编程之原子类
- java.util.concurrent.atomic 原子类，可以在多线程环境中实现无锁的、但线程安全的、不可中断的原子操作，从而减少锁竞争的开销，大大提高了并发程序的性能，特别适合用于计数器、累加器等场景
- 原子类就是利用了底层硬件的原子指令特性来实现 CAS 操作（Compare and Swap 比较并交换也称为 Compare And Set 比较并设置），天然地保证了操作的原子性，所以能够在不使用显式重量级锁机制（如 synchronized 关键字或 Lock 接口）的情况下也能够保证变量操作的原子性


## 原子类
```java
AtomicBoolean 对布尔类型进行原子操作

AtomicInteger 对整型进行原子操作
AtomicLong 对长整型进行原子操作
AtomicReference 对任意引用类型对象进行原子操作

AtomicIntegerArray 对整型数组进行原子操作
AtomicLongArray 对长整型数组进行原子操作
AtomicReferenceArray 对任意引用类型对象数组进行原子操作

AtomicMarkableReference 提供带有布尔标记位的任意引用类型对象，可以原子地更新两者，可以用来解决 ABA 问题（即一个变量在两次检查之间从 A 变为 B 再变回 A 的问题）
AtomicStampedReference 提供带有整数版本戳（版本号）的任意引用类型对象，可以原子地更新两者，比 AtomicMarkableReference 更详细，能够记录对象被修改的次数或者版本变化等信息，同样可以用来解决 ABA 问题
```


## 原子操作
```java
get 获取当前值
set 设置新值
lazySet 延迟设置新值，但不保证立即可见性（允许编译器和处理器对操作进行优化，减少内存屏障的使用，能够提供一定程度的性能优势，适用于不需要立即可见性的场景）
getAndSet 设置新值，并返回旧值
compareAndSet 基于 CAS 机制来比较并设置值，如果当前值等于预期的 expect 值，就更新为 update 值，该方法是实现原子操作的核心所在，很多其他方法内部也是基于这个方法来实现的
weakCompareAndSet 跟 compareAndSet 类似，是一种比较弱的比较并设置值操作，可能在某些情况下会返回 false 即使当前值等于预期值（系统优化结果）

getAndIncrement 以原子方式递增，并返回旧值
getAndDecrement 以原子方式递减，并返回旧值
getAndAdd 以原子方式添加一个增量到当前值，并返回旧值

incrementAndGet 以原子方式递增，并返回新值
decrementAndGet 以原子方式递减，并返回新值
addAndGet 以原子方式添加一个增量到当前值，并返回新值
```

## FieldUpdater 字段更新器
- 更新器类用于对字段进行原子性更新，这些类基于反射机制，必须是使用 volatile 修饰的字段，而且不能是 static 修饰的字段，通常适用于对现有的类（不想改变现有类的原来的结构或者去重新设计整个类）里某些字段进行原子操作的情况
- AtomicIntegerFieldUpdater 对指定类中 volatile 修饰的 int 字段的原子操作
- AtomicLongFieldUpdater 对指定类中 volatile 修饰的 long 字段的原子操作
- AtomicReferenceFieldUpdater 对指定类中 volatile 修饰的任意引用类型对象的原子操作


## Adder 和 Accumulator 累加器
- LongAdder 和 DoubleAdder 是为了解决在高并发场景下使用传统原子操作（如 AtomicLong）可能导致的性能瓶颈（高并发场景下更新操作频繁竞争而导致性能下降的问题）而设计的，减少了线程竞争，具有更好的性能表现
- LongAccumulator 和 DoubleAccumulator 同样支持高效的并发更新，并允许自定义累加规则，提供了更大的灵活性，能够执行更复杂的累加逻辑，比如求最大值、最小值和科学计算等场景
 

## ABA 问题
- 在并发编程中，ABA 问题是指一个线程在操作一个共享变量时，另一个线程如果先将变量的值从 A 改为 B，然后再改回 A，而第一个线程在检查变量时发现其值还是 A，就误以为没有发生变化而继续执行后续操作，这可能会导致错误的结果
- AtomicMarkableReference 或 AtomicStampedReference 在进行 CAS 操作时不仅会比较引用对象的值是否相等，还会比较一个额外的布尔标记位或整数版本戳是否一致，只有两个值都预期相符时才会执行更新操作，从而有效避免了 ABA 问题

```java
AtomicMarkableReference<String> atomicMarkableReference = new AtomicMarkableReference<>("A", false);
//CAS01: reference=A marked=false
System.out.println("CAS01: reference=" + atomicMarkableReference.getReference() + " marked=" + atomicMarkableReference.isMarked());
//线程 1 尝试设置 B 和 true
boolean success = atomicMarkableReference.compareAndSet("A", "B", false, true);
//test01 success=true
System.out.println("test01 success=" + success);
if (success) {
    //CAS02: reference=B marked=true
    System.out.println("CAS02: reference=" + atomicMarkableReference.getReference() + " marked=" + atomicMarkableReference.isMarked());
    //...
    //尝试将引用设置回 A 不过标记还是 true
    atomicMarkableReference.set("A", true);
    //CAS03: reference=A marked=true
    System.out.println("CAS03: reference=" + atomicMarkableReference.getReference() + " marked=" + atomicMarkableReference.isMarked());
}
//线程 2
//尝试将引用设置为 C 但期望标记为 false
success = atomicMarkableReference.compareAndSet("A", "C", false, true);
//test02 success=false
System.out.println("test02 success=" + success);
//CAS04: reference=A marked=true
System.out.println("CAS04: reference=" + atomicMarkableReference.getReference() + " marked=" + atomicMarkableReference.isMarked());
```


## 总结
- 原子类实现了轻量级的线程同步安全操作，非常适合需要高效、无锁的并发编程场景，在高并发场景中，避免使用重量级的同步机制带来的开销，通常情况下原子类性能表现非常突出
- 使用场景包括计数器、标志位、共享引用对象更新和数组元素操作等
