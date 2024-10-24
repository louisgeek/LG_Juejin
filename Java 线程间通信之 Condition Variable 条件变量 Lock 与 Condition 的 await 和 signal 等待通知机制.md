# Java 线程间通信之 Condition Variable 条件变量 Lock 与 Condition 的 await 和 signal 等待通知机制
- Condition Variable 条件变量通常与 Mutex 互斥锁（互斥锁可以确保一次只有一个线程可以访问共享资源）一起使用，以确保能够线程安全地访问共享变量（资源）和条件变量，java.util.concurrent.locks.Condition 的 await 等待和 signal 通知唤醒的操作必须在互斥锁（java.util.concurrent.locks.Lock（java.util.concurrent.locks.ReentrantLock））的保护下进行
- 通常用于实现生产者消费者模型场景，不管是生产者还是消费者，一次只能一个线程能获取到锁，如果生产者获取到锁就校验是否需要生成数据，如果消费者获取到锁就校验是否有数据可消费，当一个线程等待另一个线程执行完特定操作后继续执行，或者通知其他线程在某个条件已经满足后可以继续执行
- 可能存在通知信号丢失问题. 如果在一个线程调用 await 等待操作之前，另一个线程就已经调用了 signal 方法，那么等待的线程将错过这个信号，不会被唤醒，因为信号已经丢失了，为了避免信号丢失，在使用等待通知机制时，应该确保等待线程在正确的条件下进行等待，通知线程在正确的条件下去通知，并且通知操作应该在等待操作之后去触发
- Spurious Wakeup 虚假唤醒指的是线程在等待某个条件满足时，可能会出现因为其他原因（如系统唤醒）而被唤醒的情况，即没有线程调用 signal 或 signalAll 方法，等待的线程也有可能从 await 方法返回，所以实际上此时等待的条件并未真正满足，因此通常需要在一个循环中去检查等待条件是否满足，而不是仅仅依赖一次唤醒（比如 if 语句判断），当线程被虚假唤醒后会立即重新检查条件是否真的满足，如果发现不满足则执行 await 则继续等待
- 相对于 Object 的 wait、notify 等待通知机制，Condition 可以实现多条件等待，而且 Condition 与 Lock 配合使用，提供了更灵活的锁机制（显式锁提供了更多的灵活性和高级功能，如尝试获取锁、可中断的获取锁、公平锁等）
 


Condition 的 await、signal 和 signalAll 方法必须在持有锁的情况（lock.lock() 与 lock.unlock() 方法之间）下调用，否则会抛出 IllegalMonitorStateException 异常

## Condition#await
- 在调用 await 方法之前，当前线程必须已经是获得了锁，同时在执行 await 方法后会立即释放锁，让其他线程有机会获取到锁
- 调用 await 方法可以使当前线程进入等待（阻塞）状态，并释放所持有的锁，然后一直等待直到其他线程调用同一对象的 signal 或 signalAll 方法来唤醒它，await 支持指定一个等待超时时间和单位
- 当前线程释放所持有的锁会把该线程放置到与锁对象关联的等待队列中（等待线程池）

## Object#signal 
- 在调用 signal 或 signalAll 方法之前，当前线程必须已经是获得了锁，执行后不会立即释放锁，而需要执行完当前同步逻辑后才会释放锁，所以接到通知被唤醒的线程也不会立即获得锁，也需要等执行 signal 方法的线程释放锁后再去获取锁，也就是说被唤醒的线程不会立即从 await 方法返回并继续执行，而是它将要与其他线程重新竞争获取到锁，只有获得锁后才能从 await 方法返回并继续执行后续代码
- 调用 signal 方法可以唤醒一个正在等待该锁的线程，即唤醒一个在与该锁对象关联的等待队列中（等待线程池）的线程，如果有多个线程在等待，一次只会唤醒一个，而且是任意的（随机的），可以多次调用 signal 方法

## Object#signalAll 
- 可以唤醒所有正在等待该锁的线程，即唤醒全部与该锁对象关联的等待队列中（等待线程池）的线程，所有线程被唤醒后，需要重新竞争获取到锁，只有一个线程能获得锁后从 await 方法返回并继续执行后续代码



```java
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();//绑定了 Lock 实例的 Condition 对象
    //
    private boolean flag = false;

    public void waitForFlag() {
        lock.lock();
        try {
            while (!flag) {
                condition.await(); // 等待条件成立
            }
            // 条件满足成立后的逻辑
        } catch (InterruptedException e) {
            // 处理中断
            Thread.currentThread().interrupt();
        } finally {
            lock.unlock();
        }
    }

     public void setFlag() {
        lock.lock();
        try {
            flag = true;
            //通知所有等待的线程
            condition.signalAll(); //或者使用 signal
        } finally {
            lock.unlock();
        }
    }

    ```


## 生产者消费者模型
-  notFull 和 notEmpty 是两个不同的 Condition 对象，分别用于控制队列满和队列空的情况

```java
    private final int MAX_SIZE = 10;
    private final Queue<Integer> queue = new LinkedList<>();
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public void produce(int item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == MAX_SIZE) {
                // 如果队列满了，生产者线程等待
                notFull.await();
            }
            // 条件满足成立后的逻辑
            queue.add(item);
            // 通知消费者线程队列非空
            notEmpty.signalAll(); //或者使用 signal
        } finally {
            lock.unlock();
        }
    }

    public int consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                // 如果队列空了，消费者线程等待
                notEmpty.await();
            }
            // 条件满足成立后的逻辑
            int item = queue.poll();
            // 通知生产者线程队列非满
            notFull.signalAll(); //或者使用 signal
            return item;
        } finally {
            lock.unlock();
        }
    }
```
 

 ## 小结
 - 1 await、signal 和 signalAll 方法必须在持有锁的情况（lock.lock() 与 lock.unlock() 方法之间）下调用
 - 2 await 用来阻塞线程并立即释放锁，线程进入等待（阻塞）状态
 - 3 signal 用来唤醒线程，执行后不会立即释放锁，而需要执行完当前同步逻辑后才会释放锁，同样被唤醒的线程不会立即恢复执行，而是要等到当前持有锁的线程释放锁后，它才有机会重新竞争锁并继续执行
 - 4 相比于 Object 的 wait、notify 等待通知机制来说，它支持多条件等待、更灵活锁机制等特点