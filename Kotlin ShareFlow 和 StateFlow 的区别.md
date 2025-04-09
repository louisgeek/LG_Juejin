# Kotlin ShareFlow 和 StateFlow 的区别
- ShareFlow 和 StateFlow 都属于 Hot Flow 热流（热数据流），无论是否有订阅者都会持续产生并发射数据的可观察数据流（在创建后就会持续运行并主动发射数据的流，不需要依赖 collect 收集来触发数据的流动）
- ShareFlow 用于在多个协程之间共享数据的发射和收集操作，适用于各种需要在协程间传递数据的场景，适用于事件处理、EventBus 事件总线、消息传递等，比如传递用户登录成功事件、网络连接变化等事件，传递请求成功、请求失败等消息
- StateFlow 是一个状态容器式可观察数据流，当前状态值也可以通过其 value 属性读取，主要侧重于表示一个单一的、不断更新的状态，更适用于表示 UI 的状态或应用程序的全局状态

## SharedFlow
- SharedFlow 是一个接口，继承自 Flow 接口，重写了 collect 函数，另外有一个 List<T> 类型的 replayCache 作缓存
- 通过 MutableSharedFlow 创建，创建时不需要指定初始值，通过 replay 参数可以决定是否允许新的收集者接收之前已经发射过的数据以及数据的数量（重放功能）

```kotlin
public interface SharedFlow<out T> : Flow<T> {
    /**
     * A snapshot of the replay cache.
     */
    public val replayCache: List<T>

    /**
     * Accepts the given [collector] and [emits][FlowCollector.emit] values into it.
     * To emit values from a shared flow into a specific collector, either `collector.emitAll(flow)` or `collect { ... }`
     * SAM-conversion can be used.
     *
     * **A shared flow never completes**. A call to [Flow.collect] or any other terminal operator
     * on a shared flow never completes normally.
     *
     * @see [Flow.collect] for implementation and inheritance details.
     */
    override suspend fun collect(collector: FlowCollector<T>): Nothing
}
```

```kotlin
public fun <T> MutableSharedFlow(
    replay: Int = 0,//要 replay 重放至每个新收集者的数据数量（新订阅者订阅时重放多少个之前发射的值），如果赋值为 0 表示不进行重放
    extraBufferCapacity: Int = 0, //设置 replayCache 以外还可以为 buffer 额外追加的缓存量（bufferCapacity = replay + extraBufferCapacity）
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND
): MutableSharedFlow<T> {
    require(replay >= 0) { "replay cannot be negative, but was $replay" }
    require(extraBufferCapacity >= 0) { "extraBufferCapacity cannot be negative, but was $extraBufferCapacity" }
    require(replay > 0 || extraBufferCapacity > 0 || onBufferOverflow == BufferOverflow.SUSPEND) {
        "replay or extraBufferCapacity must be positive with non-default onBufferOverflow strategy $onBufferOverflow"
    }
    val bufferCapacity0 = replay + extraBufferCapacity
    val bufferCapacity = if (bufferCapacity0 < 0) Int.MAX_VALUE else bufferCapacity0 // coerce to MAX_VALUE on overflow
    return SharedFlowImpl(replay, bufferCapacity, onBufferOverflow)
}
```

## StateFlow
- StateFlow 状态流，总是保持一个状态值，并且可以被多个观察者同时观察（共享的数据流）
- StateFlow 是一个接口，继承自 SharedFlow 接口，所以 StateFlow 是一个特殊的 SharedFlow，另外有一个 T 类型的 value 属性值
- 初始值：通过 MutableStateFlow 创建，必须指定一个初始值作为当前的状态值，这个值会立即被发射给收集者，所以收集者在订阅后能马上获取到一个有效的状态值
- 重放：新的收集者订阅时，不管是何时订阅的都会立即接收到当前的状态值，这确保了新加入的观察者能快速同步到当前的实际状态
- 唯一性：在任意时刻，只有一个最新的状态值存在，每次发射都会覆盖之前的状态值，所以收集者只会收到最新的状态值更新
- StateFlow 每次发送数据都会与上次缓存的数据作比较，只有不一样才会发送
- StateFlow 的 replay 为 1，即可缓存最近的一次粘性事件，如果想避免粘性事件问题，可考虑改用 SharedFlow 实现，SharedFlow 的 replay 默认为 0
- 官方推荐可以考虑用 StateFlow 替代 LiveData

```kotlin
public interface StateFlow<out T> : SharedFlow<T> {
    /**
     * The current value of this state flow.
     */
    public val value: T
}
```

```kotlin
class MyViewModel : ViewModel() {
   //可变的 StateFlow
    private val _stateFlow = MutableStateFlow(0) //需要一个默认值
    //也可以直接赋值
    //val stateFlow = _stateFlow //不可变的
    val stateFlow = _stateFlow.asStateFlow() //不可变的
    
    fun test() {
        //更新 StateFlow 的值
        _stateFlow.value = 1
    }
}
```

用 SharedFlow 实现 StateFlow
```kotlin
val initialValue = 0
val _stateFlow = MutableStateFlow(initialValue)
//让一个 SharedFlow 拥有 StateFlow 一样的行为
val _sharedFlow: MutableSharedFlow<Int> = MutableSharedFlow(
    replay = 1, //重放数据的数量
    onBufferOverflow = BufferOverflow.DROP_OLDEST //缓存溢出时抛弃最旧的数据
)
_sharedFlow.tryEmit(initialValue) //发射初始值
val sharedFlow = _sharedFlow.distinctUntilChanged() //过滤相同的数据
 ```

## 总结
- SharedFlow 比 StateFlow 更通用
- SharedFlow 默认不会出现粘性事件而 StateFlow 会
- StateFlow 默认空安全，强制 value 需要被赋值一个初始数据，而且 value 也是非空的，意味着 StateFlow 永远有值的 
- StateFlow 默认防抖，因为 StateFlow 每次发送数据都会与上次缓存的数据作比较，只有不一样才会发送
- SharedFlow 和 StateFlow 的使用场景侧重有所不同，前者更适用于 Event 事件相关，后者则更适用于 State 状态相关
- 不要直接使用 launch 或 launchIn 扩展函数从界面去收集数据流，因为即使 View 不可见，这些函数也会处理事件，所以就可能会导致应用崩溃，为避免这种情况需要用 Lifecycle.repeatOnLifecycle 配合使用，或者是直接显式取消协程




 