# Kotlin 协程 coroutineScope 和 supervisorScope 的区别
- coroutineScope 和 supervisorScope 都是用于创建协程作用域的函数（不会创建启动协程），核心区别主要体现在异常传播机制和子协程的生命周期管理上
- 两者都是挂起函数，父协程（即 coroutineScope 和 supervisorScope 所在的协程）会等待其作用域内的所有子协程（通过 launch 或 async 启动）完成后才返回，以确保作用域内的所有操作执行完毕（通过挂起当前协程来等待子协程完成，不会阻塞线程）
- 结构化并发：如果父协程被取消，所有子协程会被取消

## coroutineScope
- 协同作用域（coroutine 协同），使用普通 Job 管理子协程
- 异常传播机制：子协程的异常会向上传播到父协程处理（父协程会被取消），导致整个作用域取消，其他子协程（包括未完成的）都会被取消
- 应用场景：原子性操作，所有子协程要么全部成功，要么全部失败（比如并行下载一个文件、事务操作等）

```kotlin
//最终结果：作用域因为子协程 1 的异常而整体失败
suspend fun coroutineScopeTest() =
    coroutineScope {
        launch {
            println("子协程 1 开始")
            delay(100)
            throw RuntimeException("子协程 1 失败")
        }
        launch {
            println("子协程 2 开始")
            delay(200)
            println("子协程 2 尝试完成...")
        }
    }
//-------------- 打印 --------------
//子协程 1 开始
//子协程 2 开始
//Exception in thread "main" java.lang.RuntimeException: 子协程 1 失败
```

## supervisorScope
- 主从作用域（supervisor 主管），使用 SupervisorJob 管理子协程（通过 SupervisorJob 实现异常隔离，SupervisorJob 重写了 childCancelled 返回 false）
- 异常传播机制：子协程的异常不会传播到父协程（父协程继续运行），不会取消整个作用域，仅取消自身，其他子协程会继续执行
- 应用场景：子任务之间相互独立，允许部分子协程失败（比如并行下载多个文件，日志上传与数据保存并行等）
- 子协程的异常需在子协程内部通过 try-catch 处理或者通过 CoroutineExceptionHandler 统一处理
- viewModelScope​ 默认使用结构化并发，如果想要实现异常隔离，可以显式包裹一层 supervisorScope 实现
- supervisorScope 只能作用一层

```kotlin
//最终结果：作用域等待所有子协程完成（子协程 2 成功，子协程 1 失败）
suspend fun supervisorScopeTest() =
    supervisorScope {
        launch {
            println("子协程 1 开始")
            delay(100)
            throw RuntimeException("子协程 1 失败")
        }
        launch {
            println("子协程 2 开始")
            delay(200)
            println("子协程 2 成功完成！")
        }
    }
//-------------- 打印 --------------
//子协程 1 开始
//子协程 2 开始
//Exception in thread "main" java.lang.RuntimeException: 子协程 1 失败
//子协程 2 成功完成！
```

```kotlin
//最终结果：supervisorScope 只能作用一层
suspend fun supervisorScopeTest2() =
    supervisorScope {
        launch {
            println("子协程 1 开始")
            launch {
                println("子子协程 A 开始")
                delay(100)
                throw RuntimeException("子子协程 A 失败")
            }
            launch {
                println("子子协程 B 开始")
                delay(200)
                println("子子协程 B 成功完成！")
            }
            delay(400)
            println("子协程 1 成功完成！")
        }
        launch {
            println("子协程 2 开始")
            delay(500)
            println("子协程 2 成功完成！")
        }
    }
//-------------- 打印 --------------
//子协程 1 开始
//子协程 2 开始
//子子协程 A 开始
//子子协程 B 开始
//Exception in thread "main" java.lang.RuntimeException: 子子协程 A 失败
//子协程 2 成功完成！
```

