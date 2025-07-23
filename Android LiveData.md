# Android LiveData
- LiveData 是一个可观察的数据持有类，旨在存储和传递数据，遵循观察者模式，基于 Lifecycle 生命周期感知组件进行工作，意味着 LiveData 有感知其他组件生命周期（比如 Activity 和 Fragment 等）的能力，在应用程序中实现数据和 UI 组件之间的响应式绑定（能在数据发生变化时通知 UI 进行相应的更新），这种感知能力可确保 LiveData 只更新处于活跃生命周期状态（比如 ```Lifecycle$State#STARTED 或 Lifecycle$State#RESUMED```）下的观察者，而且通常情况下不需要手动移除观察者（LiveData 会自动管理，当观察者被销毁时 LiveData 会自动取消对其的订阅），有助于避免在观察者不可见或已销毁时发送不必要的更新，从而减少内存泄漏和资源浪费等问题
- LiveData 在更新数据时是线程安全的，LiveData#setValue 必须在主线程中调用，而 LiveData#postValue 可以在后台子线程中调用，无论数据是在主线程还是子线程中更新，LiveData 都会确保在主线程中通知观察者，有助于简化多线程编程中的线程同步问题
- LiveData 默认被设计为 Sticky Events 粘性事件（事件发送后，观察者才订阅，能收到订阅之前的事件）的，先更新数据，再注册观察者依然能接收到之前的数据，新注册的观察者版本号为 -1 符合小于数据版本号的条件，所以注册时会立即收到一次数据（LiveData 每次改变数据都会对 LiveData#mVersion 数据版本号加 1，并触发 LiveData$ObserverWrapper#mLastVersion 观察者版本号小于 LiveData#mVersion 数据版本号的观察者进行 Observer#onChanged 回调，同时将数据版本号赋值给观察者版本号进行保持一致），所以粘性事件这一特性本身并不算是一种问题，反而这种特性在很多情况下都很实用，符合 LiveData 设计出来被用来处理的场景（返回的是 “UiState”，比如页面重建展示时，自动推送最后一次数据而不必重新进行请求），而在 “页面间通信” 这种场景（返回的是 “UiEvent”，比如显示 Toast、显示 Snackbar 、界面跳转或者 Dialog 的展示等）下，相当于侧重用来保存数据状态的，而不侧重用来事件传递的


被观察者
```java
public abstract class LiveData<T> {
    final Object mDataLock = new Object();
    static final int START_VERSION = -1;
    static final Object NOT_SET = new Object();
    //SafeIterableMap 是一个基于双向链表的数据结构，支持在遍历过程中安全地删除元素，线程不安全的
    private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers = new SafeIterableMap<>();
    //...
    private volatile Object mData; //mData 就是 LiveData 保存的最后一次更新的数据
    volatile Object mPendingData = NOT_SET;
    private int mVersion; //是LiveData的数据版本号，每一次更新数据版本号都会+1 
    //...
    private final Runnable mPostValueRunnable = new Runnable() {
        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                mPendingData = NOT_SET;
            }
            //最终还是调用了 LiveData#setValue 方法
            setValue((T) newValue);
        }
    };
    public LiveData(T value) {
        mData = value;
        mVersion = START_VERSION + 1;
    }
    public LiveData() {
        mData = NOT_SET;
        mVersion = START_VERSION;
    }
    private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        //
        observer.mObserver.onChanged((T) mData);
    }
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
            return;
        }
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    //
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore 直接 return 不处理了
            return;
        }
        //LifecycleBoundObserver 实现了 LifecycleObserver 接口并且包装了 LiveData 对应的 Observer 观察者，做到了将生命周期观察者和 LiveData 的观察者绑定在一起
        //每次走 observe 方法都会重新创建出 LifecycleBoundObserver 实例，LifecycleBoundObserver#onStateChanged 处理状态进入 DESTROYED 后，observer 会被自动移除（避免内存泄露）
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer); //LifecycleBoundObserver#shouldBeActive 判断至少是 STARTED（状态进入 STARTED 或 RESUMED 后才进行 Observer#onChanged 分发处理，可以节省资源）
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }
    //...
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        //通过 Handler#post 到主线程（内部是发异步消息的 Handler）
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
    //setValue 方法必须要在主线程调用
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        //分发数据（消息）
        dispatchingValue(null);
    } 
    //...  
}
```

```java
//实现对 LiveData#postValue 和 LiveData#setValue  这两个 protected 方法的访问
public class MutableLiveData<T> extends LiveData<T> {

    public MutableLiveData(T value) {
        super(value);
    }

    public MutableLiveData() {
        super();
    }

    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
}
```

观察者
```kotlin
fun interface Observer<T> {
    /**
     * Called when the data is changed to [value].
     */
    fun onChanged(value: T)
}
```

使用
```kotlin
class MainActivity : AppCompatActivity() {

    private val _testMutableLiveData = MutableLiveData<String>()
    val testLiveData:LiveData<String> = _testMutableLiveData

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        testLiveData.observe(this, Observer { data ->
            //根据接收到的数据进行 UI 更新
        })

        //设置值
        _testMutableLiveData.value = "new data"
        _testMutableLiveData.postValue("new data")
}
```

## 多线程频繁 postValue 会出现丢失数据
- 因为更新和分发值分别在两个不同的线程，如果生产者更新值速率过快，消费者分发至速率过低，就会导致上一次更新的值，还没有被分发出去就被新的值更新了，导致丢失了部分中途更新的值（生产者和消费者速度不匹配），不过 LiveData 一定能接收到 postValue 的最终值
- 由于 postValue 内部是通过将更新任务异步提交到主线程，并通过 mPendingData 暂存值来实现的，如果短时间内多次调用，只有最后一次值会被保留并触发回调内部是 Runnable 未执行，后续值会覆盖前值，最终仅最后一次值被观察者接收
- 避免使用 postValue 去传递高频事件，如果主线程可以优先使用 setValue 方法（不过要考虑 UI 可见状态）
- LiveData 本身是推荐作为驱动 UI 的数据流的，在过于频繁刷新值的场景，可以考虑改用 Kotlin 的 Flow，可以考虑通过自定义 LiveData 实现每次 postValue 都能触发独立的 Runnable 任务
```java
liveData.postValue("A") //丢失（由于 Runnable 没来得及执行，且值被后续值覆盖）
liveData.postValue("B")
//仅能接收到 "B"
```

## 在非活跃状态下多次调用 setValue 会出现丢失数据
- 观察者处于非活跃状态下去设置值不会被即时处理，仅会在观察者处于活跃状态（STARTED 或 RESUMED）时触发 onChanged 回调
- setValue 方法虽然会更新 LiveData 的内部值，但并不会触发观察者的回调，后续在 UI 重新处于活跃状态时，仅最新值会被传递，中间值丢失
- 可以考虑拆分重要的事件通知到单独的 LiveData，使得重要的通知不会被不重要的通知给覆盖
```java
//在 UI 不可见时连续执行（比如 onPause 后）
liveData.setValue("A") //丢失
liveData.setValue("B") 
//UI 可见后仅能接收到 "B"
```

## 数据倒灌
- 作者 KunMinX 定义的 Data backflow 数据倒灌是专指在页面通信（事件回调）的场景下，通过 SharedViewModel 的 LiveData 给当前页通知过一次，并返回上一页，下次再进入当前页时重复收到旧数据推送的情况
- 就是通常情况下当 Activity 或 Fragment 在因配置更改（比如屏幕旋转）后被销毁并重建的时候，由于 ViewModel 保留了内部的 LiveData，而导致可能会收到之前旧的观察者已经处理过的数据，就好像数据 “倒流、倒灌” 回来了一样
- 数据倒灌可以理解成在粘性事件前提条件的基础上，当第二次调用 observe 时候，如果还能收到第一次调用 observe 旧的观察者已经处理过的数据的情形，所以只要将 LiveData 变为非粘性事件的也能避免出现数据倒灌的问题，通常有以下几种解决方案：修改干涉版本号、SingleLiveEvent、Event Wrapper 事件包装器、UnPeekLiveData 和 SharedFlow 等

### 修改干涉版本号
- 利用反射等手段修改版本号，在 observe 方法之前将 mVersion 赋值给 mLastVersion 使得 mLastVersion = mVersion 以解决问题
- 通过实现非粘性事件以解决数据倒灌的问题

### SingleLiveEvent
- Google 提供的 SingleLiveEvent 是一个只会触发一次数据更新的 LiveData（使得 LiveData#observe 方法只会去触发一次 onChanged 方法回调）
- 利用 AtomicBoolean 标记位，初始化时默认 false，然后在给 LiveData 设置值的时候，把 AtomicBoolean 设置成 true，当 onChanged 方法回调触发后通过 AtomicBoolean#compareAndSet 方法进行比较并设置操作，将 AtomicBoolean 设置成 false，后续如果再次触发了 onChanged 方法回调，由于此时 AtomicBoolean 已经为 false ，但此时 compareAndSet 条件已经不再满足，所以就不会继续触发 onChanged 方法回调
- SingleLiveEvent 直接解决了数据倒灌的问题（仍旧是粘性事件），但是对于多个观察者监听就不能工作了（如果是存在多个观察者，只有一个能够正常收到通知），适用于只需要一个观察者接收一次更新的场景，所以局限性较大

```java
public class SingleLiveEvent<T> extends MutableLiveData<T> {
    private static final String TAG = "SingleLiveEvent";
    //
    private final AtomicBoolean mPending = new AtomicBoolean(false);
    @MainThread
    public void observe(LifecycleOwner owner, final Observer<? super T> observer) {
        if (hasActiveObservers()) {
            Log.w(TAG, "Multiple observers registered but only one will be notified of changes.");
        }
        // Observe the internal MutableLiveData
        super.observe(owner, new Observer<T>() {
            @Override
            public void onChanged(@Nullable T t) {
                //expectedValue 预期值 true，最终设置成 false
                if (mPending.compareAndSet(true, false)) {
                    //compareAndSet 操作成功
                    observer.onChanged(t);
                }
            }
        });
    }
    @MainThread
    public void setValue(@Nullable T t) {
        //设置成 true
        mPending.set(true);
        super.setValue(t);
    }
    @MainThread
    public void call() {
        setValue(null);
    }
}
```

### Event Wrapper 事件包装器
- 理念就是把 Event 事件设计为一种 State 状态，通过包装事件让事件带处理状态，形成了一种把事件视为一种状态（Consumed 已消费或者 Not Consumed 未消费）的概念
- 直接解决了数据倒灌的问题（仍旧是粘性事件），不过相对于 SingleLiveEvent 来说 Event Wrapper 可以将多个观察者添加到一次性事件中

```kotlin
open class Event<out T>(private val content: T) {

    var hasBeenHandled = false
        private set // Allow external read but not write

    //返回数据并防止再次使用
    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            //改变 hasBeenHandled 的值
            hasBeenHandled = true
            content
        }
    }

    //返回数据，即使它已被处理
    fun peekContent(): T = content
}
```

```kotlin
class ListViewModel : ViewModel {
    private val _navigateToDetails = MutableLiveData<Event<String>>()

    val navigateToDetails : LiveData<Event<String>>
        get() = _navigateToDetails


    fun userClicksOnButton(itemId: String) {
        //通过将新 Event 设置为新值来触发事件
        _navigateToDetails.value = Event(itemId)
    }
}

```

```kotlin
myViewModel.navigateToDetails.observe(this, Observer {
    //需要通过调用 getContentIfNotHandled 方法来将事件与观察者关联起来
    it.getContentIfNotHandled()?.let { 
        //仅当从未处理过事件时才进行处理
        startActivity(DetailsActivity...)
    }
})
```

### UnPeekLiveData
- KunMinX 提供的 UnPeekLiveData 维护了 UnPeekLiveData#mCurrentVersion 和 UnPeekLiveData$ObserverWrapper#mVersion 两个版本号，相当于外部自行维护一套版本号进行判断处理
- 通过实现非粘性事件以解决数据倒灌的问题


### SharedFlow
- 可以通过用 MutableSharedFlow 构造将 replay 参数设置成 0 以使得重放 0 个之前发射的值，来替代 LiveData 的功能
- 通过实现非粘性事件以解决数据倒灌的问题


## 总结
- LiveData 只在 Observer 至少处于 STARTED 状态时才能收到事件通知（如果处于 STARTED 或 RESUMED 状态，认为该观察者当前处于活跃状态，此时 LiveData 才会向观察者发送事件通知，非活跃状态的观察者不会收到任何事件通知）
- LiveData 被设计成是黏性的（当界面被意外销毁后，我们需要根据已有的数据来进行界面重建，所以 LiveData 被设计为黏性的）
- LiveData 中间值可以被新值直接覆盖，当界面处于后台时，如果 LiveData 先后接收到了多个值，那么当界面回到前台时也只会收到最新值的一次回调，中间值直接被覆盖了（对于界面状态值来说，往往需要的只是其最新状态，而不需要处理中间值，所以 LiveData 的中间值可以被新值直接覆盖）
- 所以界面相关的刷新可以使用 LiveData、事件相关的可以采用 SingleLiveEvent 等方案，避免二次触发

