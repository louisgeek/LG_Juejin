# Java synchronized 和 Lock
- 通常使用 Lock 接口的实现类 ReentrantLock，Lock 比 synchronized 更加灵活
- synchronized 和 ReentrantLock 都是可重入锁（递归锁），外层方法获取到锁后调用内层方法仍旧能获取到锁（即可以重复获取同一个锁）
- synchronized 和 ReentrantLock 都是排他锁，同一时刻只允许一个线程访问

## synchronized
- synchronized 是一个关键字，原理基于 Monitor 对象监视器锁机制（内置锁），通过 Object Header 对象头结构中的锁状态信息管理，结合锁状态升级（锁膨胀，无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁）优化性能，从而实现了高效且安全的线程同步
- synchronized 不需要手动释放锁，当 synchronized 方法或者 synchronized 代码块执行完后，线程执行发生了异常，系统都会自动释放锁
- synchronized 不是可中断锁，当一个线程处于等待某个锁的状态后是无法被中断的，只有一直等待下去
- synchronized 无法知道有没有成功获取锁
- synchronized 是非公平锁，无法保证等待的线程获取锁的顺序

## Lock
- Lock 是一个接口，原理基于 AQS（AbstractQueuedSynchronizer），通过队列化线程等待（CLH 双向队列，遵循 FIFO 原则，用来存放等待获取锁的线程）管理，结合 CAS 操作同步状态变量（volatile int state 共享变量，用于记录同步状态信息）来实现更加高效灵活的锁机制
- 使用 Lock 的时候必须手动释放，即使发生异常了也不会自动释放锁，所以一般都是逻辑写在 try-catch 中，将 unlock 解锁操作放在 finally 中，防止死锁
- Lock 是可中断锁，是可以有能力让等待锁的线程响应中断的
- Lock 是可以知道有没有成功获取锁
- Lock 实现默认情况下是非公平锁，但可以设置成公平锁
- 可以提高多个线程进行读操作的效率