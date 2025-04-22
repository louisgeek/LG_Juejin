## Java 线程的状态
- 线程的状态分为 NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING 和 TERMINATED 六种状态
- 可以通过 Thread#getState 方法获取线程状态

```java
public static enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;

    private State() {
    }
}
```

## NEW 新建
- 线程被创建但尚未调用 Thread#start 方法，此时线程仅占用内存，不占用 CPU 资源

## RUNNABLE 运行
- 线程已调用 Thread#start 方法，可能正在运行（已经获取 CPU 时间片）或者是在等待调度的就绪状态

## BLOCKED 阻塞
- 线程请求锁资源因等待（比如 synchronized 同步锁，此时锁被其他线程持有）而被阻塞，不占用 CPU 资源，获得锁后线程会重新进入 RUNNABLE 状态

## WAITING 等待
- 线程主动进入无限期等待（比如 Object#wait 和 Thread#join 等），进入无限期等待，直到其他线程显式唤醒（比如 Object#notify 或 Object#notifyAll），重新进入 RUNNABLE 状态

## TIMED_WAITING 超时等待
- 线程进入带有超时时间的等待状态（比如 Object#wait(long)、Thread#join(long) 和 Thread#sleep(long) 等）
- 与 WAITING 类似，但等待时间有限，超时后自动恢复为 RUNNABLE 状态

## TERMINATED 终止
- 线程执行完毕或者在执行过程中因出现异常而异常退出
- 一旦线程进入终止状态，它就不能再次被启动


![线程状态](https://gitee.com/louisgeek/LG_Notes/raw/master/images/xiancheng_zhuangtai.png)


```java
//状态流转
NEW -> RUNNABLE -> BLOCKED/WAITING/TIMED_WAITING -> RUNNABLE -> TERMINATED
```



