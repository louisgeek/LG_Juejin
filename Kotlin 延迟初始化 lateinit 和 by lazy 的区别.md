# Kotlin 延迟初始化 lateinit 和 by lazy 的区别
- lateinit 用来声明稍后初始化的非空属性，表明这个属性会在后续被初始化，允许开发者在类实例化时不提供初始值，该修饰符意味着开发者对系统承诺会在访问它之前对其完成初始化工作，所以需要开发者自行保证延迟初始化的逻辑
- lateinit 用于修饰非空类型的属性，只能用于 var 可变属性，而且不能用来修饰基本数据类型（比如 Int、Float 等，因为它们在内存中有默认的初始值，不太符合延迟初始化需要后续再设置值的场景），而是要用来修饰类的实例等引用类型
- by lazy 采用了 Kotlin 属性委托的这一个特性，用于创建延迟初始化的 val 只读属性，其初始化操作会被推迟到该属性首次被访问的时候才执行，本质上是利用了 Kotlin 的委托机制，将属性的读取和初始化逻辑进行了分离
- by lazy 和 lateinit 不同的是它由 Kotlin 自行管理，在属性首次被访问时自动触发初始化逻辑，不需要额外手动干预其初始化的时机

## lateinit
- 通常应用于那些无法在类构造阶段就能确定初始值，而需要在稍后某个合适时机才能确定的变量（比如 Activity#onCreate 之后才能初始化的 View 相关的变量、Dagger 类依赖注入框架场景中）
- 如果尝试在它初始化之前去访问它那么就会抛出 UninitializedPropertyAccessException 异常，所以要合理安排初始化的代码逻辑，保证初始化操作能在属性被使用之前执行，另外可以用 ::Xxx.isInitialized 属性检查器来确保属性是否已经初始化

```kotlin
class MainActivity : AppCompatActivity() {
    //使用 lateinit 修饰属性，表明稍后初始化
    lateinit var myView: View

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //这里对 lateinit 修饰的属性进行初始化赋值
        myView = findViewById(R.id.myview)
        //属性检查器,检查是否已初始化
        if (::myView.isInitialized) {
          //访问 myView
        } else {
          //可提示错误
        }
    }
}
```

## by lazy
- by 是 Kotlin 的一个关键字（用于委托），而 lazy 是一个顶层函数，通常应用于那些初始化成本较高（比如创建复杂对象、读取大文件等情况）或者可能不一定会被用到的属性，只有在真正需要使用该属性时才去进行初始化值，并且这个初始值会被缓存起来，后续访问时会直接返回缓存的值，避免了不必要的初始化开销，有助于提高程序的性能和资源利用效率
- by lazy 默认是线程安全的，如果多个线程同时首次访问使用这个属性，能够确保只有一个线程执行了初始化逻辑，其他线程会等初始化完成后获取到已初始化的值，不过函数 lazy 支持通过传递一个 LazyThreadSafetyMode 额外参数来改变线程安全的这种行为（默认使用 SynchronizedLazyImpl 实现，可以使用 LazyThreadSafetyMode.NONE 指定非线程安全的 UnsafeLazyImpl）换取一定的性能提升


```kotlin
//lazy 函数返回的是一个 Lazy 接口的实现类 SynchronizedLazyImpl，内部逻辑基本和 Java 的双重检验单例一致 ，线程安全，值只会初始化一次
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
//
public actual fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
    when (mode) {
        //默认提供线程安全保证
        LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
        //允许多个线程可能各自独立地计算初始值，但最终所有的线程都会看到同一个值
        LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
        //不提供任何线程安全保证，适用于单线程环境或者外部已经确保了线程安全的情况
        LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
    }

//Lazy 接口
public interface Lazy<out T> {
    /**
     * Gets the lazily initialized value of the current Lazy instance.
     * Once the value was initialized it must not change during the rest of lifetime of this Lazy instance.
     */
    public val value: T

    /**
     * Returns `true` if a value for this Lazy instance has been already initialized, and `false` otherwise.
     * Once this function has returned `true` it stays `true` for the rest of lifetime of this Lazy instance.
     */
    public fun isInitialized(): Boolean
}
//Lazy 接口的扩展函数实现了属性委托 getValue 约定函数
@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

//SynchronizedLazyImpl 实现类
private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}
```


## 总结
- lateinit 和 by lazy 都可以实现类似于延迟初始化的情景
- lateinit 适用于非空类型的 var 属性，需要自行确保在属性访问前被初始化，不保证线程安全，不能用于基本数据类型，没有默认值
- by lazy 适用于任何类型（可空和非空都行）的属性（var 和 val），在首次访问时才完成延迟初始化（懒加载），默认线程安全，适用于延迟初始化开销较大的对象
