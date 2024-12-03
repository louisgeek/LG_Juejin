
# CoroutineScope 协程作用域
- 协程都是由 CoroutineScope 创建的，这个 CoroutineScope 协程作用域是用来约束协程的边界的，这样能够较好的提供对应协程的取消功能，限定了协程的运行范围
- 换句话说所有协程都必须在一个作用域中运行，一个 CoroutineScope 协程作用域可以管理一个或多个协程，用来限制协程的作用范围和控制其生命周期
- 可以调用 CoroutineScope#cancel 函数来显式取消作用域，它会取消当前协程以及关联的所有子协程，已取消的作用域无法再创建协程，通常在销毁的地方调用，比如在 Activity 的 onDestroy 回调里

```kotlin
//接口
public interface CoroutineScope {
    /**
     * The context of this scope.
     * Context is encapsulated by the scope and used for implementation of coroutine builders that are extensions on the scope.
     * Accessing this property in general code is not recommended for any purposes except accessing the [Job] instance for advanced usages.
     *
     * By convention, should contain an instance of a [job][Job] to enforce structured concurrency.
     */
    public val coroutineContext: CoroutineContext //协程上下文
}
//扩展函数 cancel，其实内部就是调用了 Job#cancel 方法
public fun CoroutineScope.cancel(cause: CancellationException? = null) {
    val job = coroutineContext[Job] ?: error("Scope cannot be cancelled because it does not have a job: $this")
    job.cancel(cause)
}
```

## 1 GlobalScope 全局的协程作用域
 - GlobalScope 是一个 object 类（单例类意味着作用域的生命周期是跟 App 绑定的）实现了 CoroutineScope 接口，拥有一个 coroutineContext 是 EmptyCoroutineContext
 - 官方不推荐使用：在 GlobalScope 中启动的协程将一直运行，直到它们自己完成或被显式取消（GlobalScope#cancel），可能导致难以管理的生命周期和无法预料的内存泄漏

```kotlin
public object GlobalScope : CoroutineScope {
    /**
     * Returns [EmptyCoroutineContext].
     */
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}
```
自定义一个类似 GlobalScope 的 MyGlobalScope
```kotlin
public object MyGlobalScope : CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.IO + CoroutineName("MyGlobalScope")
}
```
默认的协程调度器
```kotlin
//launch 里的 newCoroutineContext 函数逻辑里判断如果 GlobalScope 和 launch 都未指定 Dispatchers 的话，就默认指定为 Dispatchers.Default
val job = GlobalScope.launch {
    //coroutineContext=[StandaloneCoroutine{Active}@71ee7f8, Dispatchers.Default] currentThread=DefaultDispatcher-worker-1
    Log.d("TAG", "coroutineContext=${coroutineContext} currentThread=${Thread.currentThread().name}")
}
//launch 的 CoroutineContext 合并了 MyGlobalScope 的 CoroutineContext
val job = MyGlobalScope.launch {
    //coroutineContext=[CoroutineName(MyGlobalScope), StandaloneCoroutine{Active}@cd1d35b, Dispatchers.IO] currentThread=DefaultDispatcher-worker-1
    Log.d("TAG", "coroutineContext=${coroutineContext} currentThread=${Thread.currentThread().name}")
}
//launch 的 CoroutineContext 里的 Dispatchers 覆盖 MyGlobalScope 的 CoroutineContext 里的 Dispatchers
val job = MyGlobalScope.launch(Dispatchers.Main) {
    //coroutineContext=[CoroutineName(MyGlobalScope), StandaloneCoroutine{Active}@d67a5a5, Dispatchers.Main] currentThread=main
    Log.d("TAG", "coroutineContext=${coroutineContext} currentThread=${Thread.currentThread().name}")
}
```

## 2 CoroutineScope(context: CoroutineContext) 自定义的协程作用域
- 这个 CoroutineScope 方法会返回一个 CoroutineScope 接口的实现类 ContextScope 的对象
- 比如 CoroutineScope(Dispatchers.Main)，CoroutineScope(CoroutineName("name"))
- 通常需要在销毁的时候比如 onDestroy 中调用 coroutineScope 的 cancel 方法手动取消

```kotlin
//CoroutineScope 接口的同名方法，PS：方法名为啥用大写？
@Suppress("FunctionName")
public fun CoroutineScope(context: CoroutineContext): CoroutineScope =
    //ContextScope 是 CoroutineScope 接口的简单实现，将传入的 CoroutineContext 参数赋值给 CoroutineScope#coroutineContext
    //如果不包含 Job 类型的元素就给它添加一个再返回
    ContextScope(if (context[Job] != null) context else context + Job())
```

## 3 MainScope() 运行在主线程的协程作用域
- MainScope 方法会返回一个 CoroutineScope 接口的实现类 ContextScope 的对象，其中 Dispatchers.Main 指定在 Android 主线程运行
- 其中这个 SupervisorJob 方法返回的是一种特殊的 Job，当子协程发生异常被取消时不会同时取消该 Job 的其它子协程
- 通常需要在销毁的时候比如 onDestroy 中调用 mainScope 的 cancel 方法手动取消

```kotlin
@Suppress("FunctionName")
public fun MainScope(): CoroutineScope = ContextScope(SupervisorJob() + Dispatchers.Main)

@Suppress("FunctionName")
public fun SupervisorJob(parent: Job? = null) : CompletableJob = SupervisorJobImpl(parent)
```

### 4 Lifecycle.coroutineScope 属性
- coroutineScope 是 Lifecycle 的扩展属性，Lifecycle 常用的比如 ComponentActivity 和 Fragment 里的 Lifecycle
- Dispatchers.Main.immediate 是一种特殊的 Dispatchers.Main，当已经在主线程的情况下，它就不再用 post 到下一次消息循环中了，而是直接执行
- 在此作用域中启动的协程会在适当的时候自动取消，以避免不必要的计算和内存泄漏

 ```kotlin
 public val Lifecycle.coroutineScope: LifecycleCoroutineScope
    get() {
        while (true) {
            val existing = internalScopeRef.get() as LifecycleCoroutineScopeImpl?
            if (existing != null) {
                return existing
            }
            val newScope = LifecycleCoroutineScopeImpl(
                this,
                SupervisorJob() + Dispatchers.Main.immediate //immediate 立即的
            )
            if (internalScopeRef.compareAndSet(null, newScope)) {
                newScope.register()
                return newScope
            }
        }
    }
```

### 5 LifecycleOwner.lifecycleScope 属性
- lifecycleScope 是 LifecycleOwner 的扩展属性，LifecycleOwner 常用的比如 ComponentActivity 和 Fragment
- lifecycleScope 就是 lifecycle.coroutineScope
- 在此作用域中启动的协程会在适当的时候自动取消，以避免不必要的计算和内存泄漏

```kotlin
public val LifecycleOwner.lifecycleScope: LifecycleCoroutineScope
    get() = lifecycle.coroutineScope
 ```

### 6 ViewModel.viewModelScope
- viewModelScope 是 ViewModel 的扩展属性
- 在此作用域中启动的协程会在适当的时候自动取消，以避免不必要的计算和内存泄漏

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
    get() {
        val scope: CoroutineScope? = this.getTag(JOB_KEY)
        if (scope != null) {
            return scope
        }
        return setTagIfAbsent(
            JOB_KEY,
            CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        )
    }
```