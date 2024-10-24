# Java 线程间通信之 BlockingQueue 阻塞队列
- BlockingQueue 阻塞队列是一种支持阻塞、线程安全的队列，支持在队列为空时阻塞消费者线程，在队列满时阻塞生产者线程
- BlockingQueue 是一个接口，常用实现有 ArrayBlockingQueue、LinkedBlockingQueue 等
 
 
```kotlin
  //capacity 默认 Integer.MAX_VALUE
    private val queue: BlockingQueue<Int> = LinkedBlockingQueue()
    //produce
    fun put(value: Int) {
        try {
            queue.put(value) //如果队列已满，则阻塞
        } catch (e: InterruptedException) {
            Thread.currentThread().interrupt()
        }
    }
    //consume
    fun get(): Int {
        try {
            return queue.take() //如果队列为空，则阻塞直到有元素可用
        } catch (e: InterruptedException) {
            Thread.currentThread().interrupt()
            return 0
        }
    }
```
