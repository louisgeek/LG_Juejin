# Kotlin Mutex 互斥锁
- Mutex 基于协程的挂起机制实现锁的等待锁释放（不阻塞线程），用于保证多个协程对共享资源的互斥访问，避免数据的不一致性
- Mutex 不是可重入锁（递归锁），外层方法获取到锁后调用内层方法需要重新获取锁
- Mutex 是互斥锁（排他锁、独占锁），同一时刻只允许一个协程访问
- Mutex 默认不是公平锁，但可以设置成公平锁（通过构造函数显式设置）
- 可以使用 Mutex#holdsLock 检查当前协程是否持有锁
- 尽量缩小锁的作用域，避免长时间持有锁（锁范围过大会降低并发性能），高频操作可结合原子变量（比如 AtomicInteger）减少锁竞争

## 手动管理锁
- lock 和 unlock 必须要成对出现，两者都是挂起函数
```kotlin
//声明 Mutex
val mutex = Mutex()
//
var sharedData = 0
suspend fun updateResource() {
    mutex.lock() //获取锁
    try {
      //操作共享资源
      sharedData++
    } finally {
      mutex.unlock() //释放锁，允许其他协程获取
    }
}
```

## 自动管理锁
- 通过 withLock 扩展函数简化锁管理（避免手动），withLock 也是挂起函数
```kotlin
//声明 Mutex
val mutex = Mutex()
//
var sharedData = 0
suspend fun updateResource() {
    //自动获取和释放锁
    mutex.withLock  { 
       //操作共享资源
       sharedData++
    }
}
```

## 超时设置
- 通过 tryLock 或者 withLock(timeout) 实现尝试获取锁
```kotlin
val mutex = Mutex()
//
var sharedData = 0
//返回 true 表示是否成功
val acquired = mutex.tryLock(1000) //尝试获取锁，tryLock 是挂起函数，超时时间 1000 毫秒
if (acquired) {
    try {
        sharedData++
    } finally {
       mutex.unlock() //tryLock 返回 true 之后也必须在使用完执行 unlock 释放锁
    }
} else {
   //无法获取锁，执行其他逻辑
}

//withLock(timeout)
mutex.withLock(1000) { //超时时间 1000 毫秒
    sharedData++
}
```