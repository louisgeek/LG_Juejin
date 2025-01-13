# Android Lifecycle
- Lifecycle 是基于观察者模式实现的，提供了一种更简单的方式来管理和监控 Activity 或 Fragment 的生命周期
- 可以将生命周期相关的逻辑从 Activity 或 Fragment 中分离出来（不在原本生命周期函数中写大量管理逻辑代码，而是反过来让组件自身去感知去管理，能够有效减少 Activity 或 Fragment 中的代码），将生命周期相关的逻辑集中封装在组件里进行管理，使得代码更加清晰和易于维护
- 可以通过在适当的生命周期事件中释放组件资源，可以有效避免内存泄漏


被观察者
```kotlin
//LifecycleOwner 生命周期拥有者，表示拥有生命周期的组件，比如 ComponentActivity，Fragment 都实现了 LifecycleOwner 接口
public interface LifecycleOwner {
    //生命周期状态对象
    public val lifecycle: Lifecycle
}
```
 
```kotlin
//生命周期状态抽象类
public abstract class Lifecycle {
    //...
    @MainThread
    public abstract fun addObserver(observer: LifecycleObserver)

    @MainThread
    public abstract fun removeObserver(observer: LifecycleObserver)

    @get:MainThread
    public abstract val currentState: State
    //...
}
```

```kotlin
//LifecycleRegistry 继承了 Lifecycle 抽象类，负责管理 LifecycleOwner 的生命周期状态
open class LifecycleRegistry private constructor(
    provider: LifecycleOwner, //传入一个 LifecycleOwner
    private val enforceMainThread: Boolean
) : Lifecycle() {
  //FastSafeIterableMap 在 SafeIterableMap 的基础（是一个基于双向链表的数据结构，支持在遍历过程中安全地删除元素，线程不安全的）上增加了一个 HashMap 来实际存储数据，使得数据的查询速度更快
  private var observerMap = FastSafeIterableMap<LifecycleObserver, ObserverWithState>()
  //
  private var state: State = State.INITIALIZED

  override fun addObserver(observer: LifecycleObserver) {
      enforceMainThreadIfNeeded("addObserver")
      //...
      val previous = observerMap.putIfAbsent(observer, statefulObserver)
      //...
  }

  override fun removeObserver(observer: LifecycleObserver) {
      enforceMainThreadIfNeeded("removeObserver")
      //
      observerMap.remove(observer)
  }
  //
  override var currentState: State
    get() = state

    set(state) {
        enforceMainThreadIfNeeded("setCurrentState")
        moveToState(state)
    }
}
```

观察者
```kotlin
public interface LifecycleObserver
```

```kotlin
public interface DefaultLifecycleObserver : LifecycleObserver {
    public fun onCreate(owner: LifecycleOwner) {}
    public fun onStart(owner: LifecycleOwner) {}
    public fun onResume(owner: LifecycleOwner) {}
    public fun onPause(owner: LifecycleOwner) {}
    public fun onStop(owner: LifecycleOwner) {}
    public fun onDestroy(owner: LifecycleOwner) {}
}
```

```kotlin
public fun interface LifecycleEventObserver : LifecycleObserver {
    //
    public fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event)
}
```

使用
```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        //ComponentActivity#getLifecycle 方法返回一个 LifecycleRegistry 对象
        lifecycle.addObserver(object :DefaultLifecycleObserver{
            override fun onCreate(owner: LifecycleOwner) {
                //...
            }
            //...
            override fun onDestroy(owner: LifecycleOwner) {
                //...
            }
        })
        //或者
        lifecycle.addObserver(object : LifecycleEventObserver{
            override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                //...
            }
        })
    }
}
```


## DefaultLifecycleObserver 和 LifecycleEventObserver
- 两者都是用于观察生命周期事件的接口
- 如果一个类同时实现了这两个接口，那么 DefaultLifecycleObserver 中的各个方法会先被调用，接着才会调用 LifecycleEventObserver 的 onStateChanged 方法

```kotlin
class MyDefaultLifecycleObserver : DefaultLifecycleObserver {
    //每个生命周期事件单独实现一个方法
    //
    override fun onCreate(owner: LifecycleOwner) {
        super.onCreate(owner)
    }
    override fun onDestroy(owner: LifecycleOwner) {
        super.onDestroy(owner)
    }
    //...
}
```
```kotlin
class MyLifecycleEventObserver : LifecycleEventObserver {
    //在单个方法中处理所有生命周期事件
    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        when (event) {
            Lifecycle.Event.ON_CREATE -> {

            }
            Lifecycle.Event.ON_DESTROY -> {

            }
            //...
            Lifecycle.Event.ON_ANY -> {
            //匹配任意事件
            }
            else -> {

            }
        }
    }
}
```


## 自定义生命周期管理
- 主要是利用 LifecycleRegistry 对象来管理生命周期状态
- LifecycleRegistry#handleLifecycleEvent 和 LifecycleRegistry#setCurrentState 都是用于管理生命周期状态的方法
- handleLifecycleEvent 是通过事件触发，根据事件类型推导出目标状态，而 setCurrentState 是直接指定目标状态，handleLifecycleEvent 适用于标准的生命周期事件流转（比如 Activity 或 Fragment 的生命周期），而 setCurrentState 更适用于自定义的生命周期状态管理
- 通常情况下更应该使用 handleLifecycleEvent 方法来处理生命周期事件触发，而不是直接设置状态（可能违反状态机的规则出现错误的状态转换可能会导致不可预测的行为）

```kotlin
class MyLifecycleOwner : LifecycleOwner {
    //
    private val lifecycleRegistry = LifecycleRegistry(this)

    init {
        //设置初始状态
        lifecycleRegistry.currentState = Lifecycle.State.CREATED
        //lifecycleRegistry.setCurrentState(Lifecycle.State.CREATED);
        //或者使用 lifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.CREATED)
    }

    override val lifecycle: Lifecycle
        get() = lifecycleRegistry

    //模拟启动
    private fun onStart() {
        lifecycleRegistry.currentState = Lifecycle.State.STARTED
    }
    //模拟停止
    private void stop() {
        lifecycleRegistry.currentState = Lifecycle.State.STOPPED
    }
    //模拟销毁
    private fun onDestroy() {
        lifecycleRegistry.currentState = Lifecycle.State.DESTROYED
    }
}
```


## ProcessLifecycleOwner
- 以前可以用 Application#registerActivityLifecycleCallbacks 的方式来进行判断 App 是处于前台还是后台，但是需要自行维护 mStartedActivityCount，mStoppedActivityCount 等变量去实现，相对来说比较麻烦，现在可以直接用 ProcessLifecycleOwner 类来实现，不过实现原理和原先自行实现的方式思路是基本一致的

```groovy
implementation 'androidx.lifecycle:lifecycle-process:2.8.7'
```

```kotlin
class ProcessLifecycleOwner private constructor() : LifecycleOwner {
    //同样利用 LifecycleRegistry 对象来管理
    private val registry = LifecycleRegistry(this)
    //
    internal fun attach(context: Context) {
    //...
    registry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE)
    //
    val app = context.applicationContext as Application
    //
    app.registerActivityLifecycleCallbacks(object : EmptyActivityLifecycleCallbacks() {
        //
        override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {
        //...
            if (Build.VERSION.SDK_INT < 29) {
                //ReportFragment 是专门用于分发生命周期事件的 Fragment
                activity.reportFragment.setProcessListener(initializationListener)
            }
        }
        override fun onActivityPaused(activity: Activity) {
        //...
        }
        override fun onActivityStopped(activity: Activity) {
        //...
        }
    })
    }
}
```

 