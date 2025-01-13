# Java 线程间通信之 Condition Variable 条件变量 synchronized 与 Object 的 wait 和 notify 等待通知机制
- Condition Variable 条件变量通常与 Mutex 互斥锁（互斥锁可以确保一次只有一个线程可以访问共享资源）一起使用，以确保能够线程安全地访问共享变量（资源）和条件变量，Object 的 wait 等待和 notify 通知唤醒的操作必须在互斥锁（synchronized 关键字）的保护下进行
- 通常用于实现生产者消费者模型场景，不管是生产者还是消费者，一次只能一个线程能获取到锁，如果生产者获取到锁就校验是否需要生成数据，如果消费者获取到锁就校验是否有数据可消费，当一个线程等待另一个线程执行完特定操作后继续执行，或者通知其他线程在某个条件已经满足后可以继续执行
- 可能存在通知信号丢失问题. 如果在一个线程调用 wait 等待操作之前，另一个线程就已经调用了 notify 方法，那么等待的线程将错过这个信号，不会被唤醒，因为信号已经丢失了，为了避免信号丢失，在使用等待通知机制时，应该确保等待线程在正确的条件下进行等待，通知线程在正确的条件下去通知，并且通知操作应该在等待操作之后去触发
- Spurious Wakeup 虚假唤醒指的是线程在等待某个条件满足时，可能会出现因为其他原因（如系统唤醒）而被唤醒的情况，即没有线程调用 notify 或 notifyAll 方法，等待的线程也有可能从 wait 方法返回，所以实际上此时等待的条件并未真正满足，因此通常需要在一个循环中去检查等待条件是否满足，而不是仅仅依赖一次唤醒（比如 if 语句判断），当线程被虚假唤醒后会立即重新检查条件是否真的满足，如果发现不满足则执行 wait 则继续等待
- android.app.SharedPreferencesImpl 里面用到了这个机制，get 数据均需要走 awaitLoadedLocked 方法去执行 mLock.wait() 方法，等到 loadFromDisk 方法加载完文件后执行 mLock.notifyAll() 方法

Object 的 wait、notify 和 notifyAll 方法必须在同步代码块或者同步方法中调用（即在持有锁的情况下），否则会抛出 IllegalMonitorStateException 异常，因为这些方法是基于 object's monitor 对象的监视器（锁）来实现线程间通信的

## Object#wait
- 在调用 wait 方法之前，当前线程必须已经是获得了锁，同时在执行 wait 方法后会立即释放锁，让其他线程有机会获取到锁
- 调用 wait 方法可以使当前线程进入等待（阻塞）状态，并释放所持有的锁，然后一直等待直到其他线程调用同一对象的 notify 或 notifyAll 方法来唤醒它，wait 支持指定一个等待超时时间
- 当前线程释放所持有的锁会把该线程放置到与锁对象关联的等待队列中（等待线程池）

## Object#notify 
- 在调用 notify 或 notifyAll 方法之前，当前线程必须已经是获得了锁，执行后不会立即释放锁，而需要执行完当前同步的代码块或方法后才会释放锁，所以接到通知被唤醒的线程也不会立即获得锁，也需要等执行 notify 方法的线程释放锁后再去获取锁，也就是说被唤醒的线程不会立即从 wait 方法返回并继续执行，而是它将要与其他线程重新竞争获取到锁，只有获得锁后才能从 wait 方法返回并继续执行后续代码
- 调用 notify 方法可以唤醒一个正在等待该锁的线程，即唤醒一个在与该锁对象关联的等待队列中（等待线程池）的线程，如果有多个线程在等待，一次只会唤醒一个，而且是任意的（随机的），可以多次调用 notify 方法

## Object#notifyAll 
- 可以唤醒所有正在等待该锁的线程，即唤醒全部与该锁对象关联的等待队列中（等待线程池）的线程，所有线程被唤醒后，需要重新竞争获取到锁，只有一个线程能获得锁后从 wait 方法返回并继续执行后续代码


synchronized 方法锁（对象锁）
```java
//一个或多个线程可能会调用 waitForFlag 方法并进入等待状态，直到另一个线程调用 setFlag 方法来设置条件后唤醒它们
private boolean flag = false;

public synchronized void waitForFlag() {
        while (!flag) { //使用 while 循环而不是 if 语句判断，以避免虚假唤醒
            //循环检查条件
            try {
                wait(); //synchronized 方法锁默认的锁对象是 this，就是当前类对象锁，而这里 this.wait 也是省略了 this.
            } catch (InterruptedException e) {
                //处理中断
                Thread.currentThread().interrupt();
            }
        }
       // 条件满足成立后的逻辑
}


public synchronized void setFlag() {
        flag = true;
        notifyAll(); //或者使用 notify
}

```

synchronized(lock) 代码块锁（对象锁）
```java
    private boolean flag = false;
    private final Object lock = new Object();

    public void waitForFlag() {
        synchronized (lock) {
          while (!flag) { //使用 while 循环而不是 if 语句判断，以避免虚假唤醒
            //循环检查条件
            try {
                lock.wait(); //lock 对象锁
            } catch (InterruptedException e) {
                // 处理中断
                Thread.currentThread().interrupt();
            }
          }
          // 条件满足成立后的逻辑
        }
    }

    public void setFlag() {
        synchronized (lock) {
          flag = true;
          lock.notifyAll(); //或者使用 notify
        }
    }
```


## 生产者消费者模型
- 当缓冲区为空时，消费者线程调用 get 方法会进入等待状态，直到生产者线程调用 put 方法放入数据。类似地，当缓冲区非空时，生产者线程调用 put 方法会进入等待状态，直到消费者线程调用 get 方法取出数据
```java
    private int sharedValue;
    private boolean flag = true;//is empty

    public synchronized void put(int value) {
        while (!flag) {
            try {
                wait();// 等待条件成立
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 条件满足成立后的逻辑
        sharedValue = value;
        flag = false;
        notify();
    }

    public synchronized int get() {
        while (flag) {
            try {
                wait();// 等待条件成立
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 条件满足成立后的逻辑
        flag = true;
        notify();
        return sharedValue;
    }
```
 
 
 ## 总结
 - 1 wait、notify 和 notifyAll 方法必须在同步代码块或者同步方法中调用，调用的对象必须是被锁定的对象
 - 2 wait 用来阻塞线程并立即释放锁，线程进入等待（阻塞）状态
 - 3 notify 用来唤醒线程，执行后不会立即释放锁，而需要执行完当前同步的代码块或方法后才会释放锁，同样被唤醒的线程不会立即恢复执行，而是要等到当前持有锁的线程释放锁后，它才有机会重新竞争锁并继续执行