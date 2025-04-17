


## 循环按顺序执行
- 每个线程执行完成后释放下一个线程的信号量，形成闭环循环
- 多个线程按特定顺序执行不同任务（能够循环进行）

```java
Semaphore semaphore1 = new Semaphore(1); // 任务 1 可立即执行
Semaphore semaphore2 = new Semaphore(0); // 任务 2 需等待任务 1 完成
Semaphore semaphore3 = new Semaphore(0); // 任务 3 需等待任务 2 完成
//
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore1.acquire(); //获取任务 1 许可（阻塞直到可用）
            //线程 1 的逻辑
            semaphore2.release(); //通知任务 2 可执行
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore2.acquire(); //获取任务 2 许可（阻塞直到可用）
            //线程 2 的逻辑
            semaphore3.release(); //通知任务 3 可执行
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
Thread thread3 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            semaphore3.acquire(); //获取任务 3 许可（阻塞直到可用）
            //线程 3 的逻辑
            semaphore1.release(); //通知任务 1 可执行（循环回到任务 1）
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
thread1.start(); //启动线程 1
thread2.start(); //启动线程 2
thread3.start(); //启动线程 3
```