# Java 并发编程之线程池
- Java Thread Pool 线程池是通过 java.util.concurrent 并发工具包提供的一种管理和复用线程的机制，通过复用已经创建好的线程来执行新的任务，减少线程在创建和销毁时所产生的性能开销，提高了程序性能和响应性并且减少了资源的消耗
- Java 线程池其实就是一种执行器（Executor），通常拥有一组预先创建好的线程，用于在后台中去执行一些任务，它一般会关联一个任务队列（工作队列）来存放待执行的任务
- Executor 接口有个 execute 方法用于执行 Runnable 任务
- ExecutorService 接口继承自 Executor，提供了一些管理线程池生命周期和提交任务的方法（比如 shutdown、shutdownNow、isShutdown、isTerminated 和 submit 等），用于管理线程池和任务执行，可以做到将任务的提交和执行进行解耦，使用线程池可以对线程进行有效统一的管理
- ScheduledExecutorService 接口，提供用于执行延迟执行或定期执行的任务
- ThreadPoolExecutor 是 ExecutorService 的具体实现类，可以灵活配置核心线程数、最大线程数、存活时间、任务队列和拒绝策略等
- ScheduledThreadPoolExecutor 是 ScheduledExecutorService 的具体实现类
- Executors 是一个工厂类，提供了若干个静态方法用于创建不同类型的线程池（比如 Executors#newSingleThreadExecutor 和 Executors#newCachedThreadPool 等），不过不太推荐使用，而是更推荐直接使用 ThreadPoolExecutor 来创建自定义线程池


```java
public interface Executor {
    //可以使用 execute 方法向线程池提交 Runnable 任务
    void execute(Runnable command);
}
public interface ExecutorService extends Executor {
    //
    void shutdown();
    //
    List<Runnable> shutdownNow();
    //可以用来判断线程池是否已经开始关闭，在调用了 shutdown 或 shutdownNow 方法后 isShutdown 方法会返回 true
    boolean isShutdown();
    //可以用来判断线程池是否已经彻底关闭，在所有任务都执行完毕且线程池已关闭的情况下 isTerminated 方法返回 true
    boolean isTerminated();
    //...
    //可以使用 submit 方法向线程池提交 Callable 任务
    <T> Future<T> submit(Callable<T> task);
    //...
    //可以使用 submit 方法向线程池提交 Runnable 任务
    Future<?> submit(Runnable task);
    //...
}
public abstract class AbstractExecutorService implements ExecutorService {
    //...
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        //newTaskFor 包装了 new FutureTask 逻辑
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask); //submit 实际上还是调用L execute 方法，只不过利用了 Future 来获取任务执行结果
        return ftask;
    }
    //...
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        //newTaskFor 包装了 new FutureTask 逻辑
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
    //...   
}
//线程池的核心实现类
public class ThreadPoolExecutor extends AbstractExecutorService {
    //...
    private final BlockingQueue<Runnable> workQueue; //工作队列（阻塞队列）
    //...
    private volatile RejectedExecutionHandler handler; //任务拒绝策略，当工作队列已满且线程数达到最大线程数时对新任务采取的处理策略
    //...
    private volatile long keepAliveTime; //空闲线程（非核心线程，超过核心线程数以外的线程）存活时间，非核心线程空闲时在终止前允许等待新任务的最长时间，确保空闲时线程能在合适的时间被回收
    private volatile boolean allowCoreThreadTimeOut; //如果 allowCoreThreadTimeOut 为 true 时，那么线程池中核心线程在 keepAliveTime 超时后也将回收
    private volatile int corePoolSize; //核心池大小（核心线程数，保持活跃的线程）
    //...
    private volatile int maximumPoolSize; //最大池大小（最大线程数）
}
public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements ScheduledExecutorService {
    //...
    //通过 schedule 调度提交任务，以固定频率来执行一个任务，按照上一次任务的发起时间计算下一次任务的开始时间
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
        //...
    }
    //...
    //通过 schedule 调度提交任务，以上一次任务的结束时间计算下一次任务的开始时间，在你不能预测调度任务的执行时长时是很有用
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) {
        //... 
    }
    //...
}
```


## 线程池的工作原理
- 1 当有新任务提交到线程池时，如果当前线程池中正在运行的线程数小于 corePoolSize 核心线程数的话，那么立即创建新的线程（核心线程）来执行该任务（即使此时线程池中存在空闲线程）
- 2 如果当前线程数已经达到了 corePoolSize 核心线程数的话，那么新任务会被添加到 workQueue 工作队列（阻塞队列）中等待后续的调度执行
- 3 当 workQueue 工作队列（阻塞队列）满了且当前线程数小于 maxPoolSize 最大线程数，那么会创建新的线程（非核心线程）来处理队列中的任务
- 4 当 workQueue 工作队列（阻塞队列）满了且当前线程数已经达到 maxPoolSize 最大线程数，那么就会根据设置的 RejectedExecutionHandler 拒绝策略来响应，不同的拒绝策略会采取不同的处理行为


## 线程池的类型
- Executors#newCachedThreadPool 缓存线程池，创建一个可根据需要动态调整线程数量的线程池，核心线程数为 0，最大线程数为 Integer.MAX_VALUE，默认空闲线程存活时间 60 秒被回收，适合需要执行很多短期异步任务的场景中，使用 SynchronousQueue 作为阻塞队列，可能会因为创建过多的线程（比如任务提交速度很快但是任务执行比较耗时，会不断创建新的线程来处理任务，频繁地创建和销毁线程可能导致系统性能下降）而导致系统资源耗尽
- Executors#newFixedThreadPool 固定大小的线程池，创建一个固定线程数量（传参 nThreads）的线程池，核心线程数和最大线程数都是 nThreads，固定数量的线程一直存在不会被回收，如果没有任务执行就会一直等待，适合可以预测线程数量的场景中，使用 LinkedBlockingQueue 作为阻塞队列，可能会导致内存溢出的情况（比如无界队列 LinkedBlockingQueue 理论上可以不断地往队列中添加任务，最终可能耗尽内存，引发内存溢出）
- Executors#newScheduledThreadPool 调度线程池，支持定时或者周期性执行任务的线程池，传参 corePoolSize 确定核心线程数，而最大线程数也是 Integer.MAX_VALUE，所以也可能有导致系统资源耗尽的问题
- Executors#newSingleThreadExecutor 单线程的线程池，创建只有一个线程的线程池，即核心线程数和最大线程数都是 1，使用 LinkedBlockingQueue 作为阻塞队列，可以保证所有任务按顺序执行
- Executors#newSingleThreadScheduledExecutor 单线程的调度线程池，其 corePoolSize 核心线程数为 1，最大线程数是 Integer.MAX_VALUE
- Executors#newWorkStealingPool 创建一个具有工作窃取特性的线程池（基于 Work Stealing 工作窃取算法实现），工作窃取算法允许空闲的线程从其他忙碌线程的任务队列中窃取任务来执行，从而能够充分利用多核 CPU 资源，提高任务执行效率，每个线程都维护一个自己的任务队列（Deque 双端队列），当线程执行完自己队列中的任务后，它会尝试从其他线程的队列末尾 “窃取” 任务来执行，从而实现负载均衡，能够减少线程间的竞争，提高系统的整体性能


## 线程池的关闭
- shutdown 平滑地关闭线程池，不再接受新的任务提交，会等待所有已提交任务执行完毕后再关闭线程池
- shutdownNow 尝试立即关闭线程池，并会向线程池中正在执行的线程发送中断信号，同时清空阻塞队列中尚未执行的任务，并返回这些未执行的任务列表，不过任务不一定会立即停止（取决于任务对线程中断的响应情况）


## 线程池的 BlockingQueue 阻塞队列
- LinkedBlockingQueue 基于链表实现的无界阻塞队列（默认 capacity 容量长度传值 Integer.MAX_VALUE），理论上可以存放无限个任务，但受限于系统资源
- ArrayBlockingQueue 基于数组实现的有界阻塞队列，需要在创建时指定队列容量大小
- SynchronousQueue 同步队列，是一种特殊的阻塞队列，队列容量为 0，即不存储元素，本质上只是一个同步机制，允许线程间在队列中直接交换元素，SynchronousQueue 的 put 插入和 take 移除操作会相互阻塞，插入操作和移除操作必须是成对出现的并且是同步进行的
- PriorityBlockingQueue ：优先队列，可以针对任务排序


## 线程池的 RejectedExecutionHandler 拒绝策略
- AbortPolicy 默认策略，直接丢弃后来的任务并抛出 RejectedExecutionException 异常
- CallerRunsPolicy 让提交任务的线程自己去执行这个任务（直接调用 run 方法），由调用线程（提交任务的线程）运行任务
- DiscardPolicy 直接丢弃后来的任务，不会有抛出异常或别的额外处理
- DiscardOldestPolicy 丢弃阻塞队列中最老的任务（即排在队列最前面的任务），然后重新尝试执行任务（尝试重复此过程）


## 自定义线程池
- maximumPoolSize 取值推荐数量
    - 如果是 CPU 密集型任务，参考值可以设为 NUMBER_OF_CORES + 1 或者 NUMBER_OF_CORES + 2
    - 如果是 IO 密集型任务，参考值可以设置为 NUMBER_OF_CORES * 2

```java
//获取活跃的 cpu 数量
int NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors();
int KEEP_ALIVE_TIME = 1;
TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;
//
BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<Runnable>();
                                                        //corePoolSize, maximumPoolSize
ExecutorService executorService = new ThreadPoolExecutor(NUMBER_OF_CORES, NUMBER_OF_CORES * 2, KEEP_ALIVE_TIME, KEEP_ALIVE_TIME_UNIT,
                                                            workQueue,
                                                            new BackgroundThreadFactory(),
                                                            new DefaultRejectedExecutionHandler());

//执行任务
executorService.execute(new Runnnable() {
//...
});
```

## 总结
- 使用线程池的主要优势是减少在创建和销毁线程上所花的时间以及性能的开销，可有效控制最大并发线程数，提高系统资源的使用率，避免过多资源竞争，还能够提供定时执行、定期执行、单线程等特性
- 如果不恰当的使用线程池，也有可能造成 “过度切换” 等导致的系统资源耗尽和内存溢出等问题（比如允许阻塞队列默认容量长度为 Integer.MAX_VALUE 的线程池，可能会堆积大量的任务从而导致内存溢出，而允许创建线程数量为 Integer.MAX_VALUE 的线程池，可能会创建大量的线程从而导致系统资源耗尽）
- 另外不使用线程池而是采用创建匿名线程的方式不便于对线程进行有效统一的管理，对资源使用等可能会造成混乱