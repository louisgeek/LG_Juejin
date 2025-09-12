# Java 线程池取消的方式
- 线程池取消分为取消线程池中单个任务和关闭线程池本身，过程分为停止接收新任务​​、​​终止正在执行的任务​​以及​​清理未执行的任务​​

## Future#cancel 取消单个任务
```java
public static void main(String[] args) {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<?> future = executor.submit(() -> {
        try {
               while (!Thread.currentThread().isInterrupted()) {
                   //执行任务逻辑
               }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); //重设中断状态
        }
    });
    //演示逻辑
    try {
        //等待一段时间后去中断 thread 线程
        Thread.sleep(10);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("请求线程取消");
    //true 尝试中断正在执行的任务（依赖任务本身是否处理中断，任务内部​需要有响应中断的逻辑），false 仅取消未开始执行的任务（不会中断正在运行的任务，会继续工作）
    boolean cancelled = future.cancel(true); //FutureTask#cancel 传 true 内部会调用 Thread#interrupt 方法
    System.out.println("任务是否取消成功：" + cancelled);
    //
    executor.shutdown();
}
```

## shutdown 平滑关闭
- 停止接收新任务，会等待已提交的任务（包括正在执行的和队列中等待的）执行完成后再关闭
- 可以确保已提交的任务都是正常结束的
```java
//停止接收新任务，等待已提交任务完成
executor.shutdown();
```

## shutdownNow 强制关闭
- 内部会调用 Thread#interrupt 方法，能否通过 interrupt 有效取消任务，取决于任务本身响应​​中断的逻辑​​
- 停止接收新任务，尝试立即关闭线程池，返回尚未执行的任务列表，并尝试中断正在执行的任务
- 后续可以对尚未执行的任务进行处理，比如记录日志或重新提交
```java
//立即停止所有任务，返回未执行的任务列表
List<Runnable> cancelledTasks = executor.shutdownNow();
System.out.println("被取消的任务数: " + cancelledTasks.size());
```

## 线程池关闭最佳实践
```java
private void executorShutdown() {
    if (executor.isShutdown()) { //判断线程池是否已调用 shutdown 或 shutdownNow 过了
        return;
    }
    executor.shutdown(); //触发关闭，停止接收新任务，等待已提交任务完成
    try {
        //最多等待 30 秒，尽量能够让已提交任务执行完成
        if (!executor.awaitTermination(30, TimeUnit.SECONDS)) { //awaitTermination 会阻塞当前线程等待线程池关闭
            //超时后强制关闭
            executor.shutdownNow();
            //再次等待（可选）
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                System.err.println("线程池未完全终止");
            }
        }
    } catch (InterruptedException e) {
        //处理中断（如果当前线程在 awaitTermination 等待期间被其他线程调用 Thread#interrupt 方法中断）
        executor.shutdownNow();
        Thread.currentThread().interrupt(); //重设中断状态
    }
}
```

## 总结
- 任务响应中断：使用 Future.cancel(true) 或 shutdownNow 时，任务自身需要通过 Thread.currentThread().interrupted() 或InterruptedException 检测中断状态并响应中断
- 优先使用 shutdown 仅在必要时使用 shutdownNow