# Java 多线程顺序执行
- 最基础的方法就是按顺序调用各个线程去实现
- 可以利用 Executors.newSingleThreadExecutor 单线程池来保证
- 利用设计模式（比如职责链模式或者管道模式）也可以实现多线程顺序执行的逻辑

## 1 ExecutorService 和 Future 配合
- 利用 java.util.concurrent.Future 的阻塞特性，Future#get 方法会阻塞当前线程去获取异步执行任务的结果

```java
ExecutorService executorService = Executors.newFixedThreadPool(3);
Future<?> future1 = executorService.submit(new Runnable() {
    @Override
    public void run() {
        //线程 1 的逻辑
    }
});
future1.get(); //等待线程 1 执行完毕
Future<?> future2 = executorService.submit(new Runnable() {
    @Override
    public void run() {
        //线程 2 的逻辑
    }
});
future2.get(); //等待线程 2 执行完毕
Future<?> future3 = executorService.submit(new Runnable() {
    @Override
    public void run() {
        //线程 3 的逻辑
    }
});
future3.get(); //等待线程 3 执行完毕
//
executorService.shutdown();
```

## 2 Thread#join 
- join 方法可以让调用该方法的线程等待被调用的线程执行完毕后再继续执行（使调用者所处的线程转换为 Thread$State#WAITING 状态直到调用者逻辑执行完毕）
- 底层使用 Object 的 wait 和 notify 等待通知机制来实现，Object#wait 方法必须在同步代码块或者同步方法中调用（即在持有锁的情况下，而 Thread#join 方法里确实有 `synchronized(lock) { }` 的逻辑）
 
方式 1
```java
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 1 的逻辑
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 2 的逻辑
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 3 的逻辑
    }
});
thread1.start(); //启动线程 1
thread1.join(); //当前线程等待线程 1 执行完毕
//
thread2.start(); //启动线程 2
thread2.join(); //当前线程等待线程 2 执行完毕
//
thread3.start(); //启动线程 3
thread3.join(); //当前线程等待线程 3 执行完毕（这里可按需省略）
```

方式 2
```java
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 1 的逻辑
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        thread1.join(); //当前线程等待线程 1 执行完毕
        //线程 2 的逻辑
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        thread2.join(); //当前线程等待线程 2 执行完毕
        //线程 3 的逻辑
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
//
thread3.join(); //当前线程等待线程 3 执行完毕（这里可按需省略）
```

## 3 CountDownLatch   
- java.util.concurrent.CountDownLatch 倒计时闭锁，CountDownLatch 是一次性的，不能重置
- 可以让一个或多个线程等待其他线程完成某些操作后继续执行，可按需定义多个 CountDownLatch 实现复杂的控制逻辑
- 注意线程要异常捕获，出异常也需要减计数器，建议在 finally 里执行 countDown

方式 1
- 定义 1 个信号，其他地方 await 等待，直到 countDown 执行，计数器到 0 后通知所有 await 继续执行（类似跑步指令枪打出后所有人才开始跑）
```java
CountDownLatch countDownLatch = new CountDownLatch(1);
//
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        countDownLatch.await();
        //
        //线程 1 的逻辑
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        countDownLatch.await();
        //
        //线程 2 的逻辑
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        countDownLatch.await();
        //
        //线程 3 的逻辑
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
//
countDownLatch.countDown(); //通知所有 await 继续执行
```

方式 2 
- 定义 N 个信号，总的一个地方 await 等待，其他 N 个线程分别执行 countDown，直到计数器到 0 后通知 await 继续执行（类似跑步所有人跑完了整体才能收场，不过 await 有带超时参数方法，给个超时时间，不无限制等待）
- 通常用于汇总若干个线程的结果后再继续执行一个线程的场景
```java
CountDownLatch countDownLatch = new CountDownLatch(3);
//
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 1 的逻辑
        //
        countDownLatch.countDown();
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 2 的逻辑
        //
        countDownLatch.countDown();
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        //线程 3 的逻辑
        //
        countDownLatch.countDown();
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
//
countDownLatch.await(); //等待其他所有线程完成后继续执行
```

## 4 CyclicBarrier
- java.util.concurrent.CyclicBarrier 循环屏障，CyclicBarrier 可以通过 reset 方法重置并循环使用
- 可以让多个线程相互等待直到所有线程都准备好后再一起继续执行，多个参与同步的线程在某个地方互相等待直到所有线程都到达屏障点（互相等待的线程通过调用 await 方法表示到达）后再一起继续执行后续操作
- CyclicBarrier 可以设置一个可选的屏障动作，当所有参与同步的线程都到达屏障点时可以执行一个指定的动作，可以执行一些公共任务

```java
CyclicBarrier  cyclicBarrier  = new CyclicBarrier(3, new Runnable() {
    @Override
    public void run() {
       //屏障动作，所有参与同步的线程到达屏障点后需要执行的操作
    }
});
//
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
       //线程 1 的逻辑
       //代表自身线程已经到达屏障点，然后等待其他线程到达
       cyclicBarrier.await();
       //所有线程都到达屏障点后继续执行的逻辑
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
       //线程 2 的逻辑
       //代表自身线程已经到达屏障点，然后等待其他线程到达
       cyclicBarrier.await();
       //所有线程都到达屏障点后继续执行的逻辑
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
       //线程 3 的逻辑
       //代表自身线程已经到达屏障点，然后等待其他线程到达
       cyclicBarrier.await();
       //所有线程都到达屏障点后继续执行的逻辑
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
```

## 5 Semaphore
- java.util.concurrent.Semaphore 信号量，用于控制并发资源访问数量，内部维护一个许可计数，每个线程在进入时需要 acquire 申请一个许可，完成后 release 释放该许可，当许可计数为零时，其他线程会阻塞，直到有线程释放许可
- 通过 Semaphore#acquire 获取许可，如果许可不可用，则线程会阻塞
- 通过 Semaphore.tryAcquire 尝试获取许可，如果许可不可用立即返回 false，线程不会阻塞
- 通过 Semaphore#release 释放许可，允许其他线程获取许可
```java
Semaphore semaphore = new Semaphore(1); //初始许可数 1
//
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore.acquire(); //获取一个许可（阻塞直到可用）
            //线程 1 的逻辑
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); //释放许可
        }
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore.acquire(); //获取一个许可（阻塞直到可用）
            //线程 2 的逻辑
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); //释放许可
        }
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore.acquire(); //获取一个许可（阻塞直到可用）
            //线程 3 的逻辑
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); //释放许可
        }
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
```