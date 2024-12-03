# Android IntentService
- IntentService 继承自 Service，组合了一个 Handler 和 Looper，内部封装了 HandlerThread 机制，HandlerThread 会随着 IntentService#onCreate 的回调而启动，而且在 IntentService#onDestroy 回调走完后会调 Looper#quit 方法
- 单线程执行：Service 默认在主线程中运行，而 IntentService 是运行在一个子线程中，不会阻塞主线程，默认无需去手动管理线程
- 自动停止：IntentService 不需要手动调用 Service#stopSelf 或 Context#stopService 去停止服务，当 onHandleIntent 方法执行完毕后，它会通过 stopSelf 传入指定的 startId 方法自动停止
- 多意图顺序执行：每个 Intent 请求都会被放入一个队列中，并按顺序处理，当所有请求处理完毕后，IntentService 会自动停止
- 适用于一次性或短期的后台任务，不需要与主线程交互
- 适用于需要按顺序处理多个异步任务的场景，保证了任务的有序性，不支持并发处理多个任务
- 适用于网络请求、文件上传下载、数据库操作等不需要持续运行的任务

## 使用
```java
public class MyIntentService extends IntentService {
    /**
     * 构造函数必须调用父类的构造函数并传入一个字符串作为服务的名称
     * Creates an IntentService.  Invoked by your subclass's constructor.
     * @param name Used to name the worker thread, important only for debugging.
     */
    public MyIntentService(String name) {
        super(name);
    }
    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        //这方法会在子线程中执行，
        if (intent != null) {
            String action = intent.getAction();
            if ("ACTION_DO_SOMETHING".equals(action)) {
                //处理
                //doSomething
            }
        }
    }
}
```

```xml
<application
    ...
    <service android:name=".MyIntentService" />
    ...
</application>
```

```java
//启动
Intent intent = new Intent(this, MyIntentService.class);
intent.putExtra("key", "value"); //需要传递的数据
intent.setAction("ACTION_DO_SOMETHING");
startService(intent);
```

## 源码
```java
public abstract class IntentService extends Service {
//IntentService 内部的 Looper 和 Handler
private volatile Looper mServiceLooper;
@UnsupportedAppUsage
private volatile ServiceHandler mServiceHandler;
private String mName;
private boolean mRedelivery;
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }
    @Override
    public void handleMessage(Message msg) {
        //msg.arg1 和 msg.obj 就是 IntentService#onStartCommand 调用 IntentService#onStart 传入的 startId 和 intent
        onHandleIntent((Intent)msg.obj);
        //在调用 stopSelf(int startId) 的过程中，系统会检测是否还有其他 startId 存在，如果存在则不会立即销毁 Service，当 onStartCommand 和 stopSelf 成对的时候，Service 会被销毁（不过如果直接调用 stopSelf(-1) 的话，那就会直接销毁 Service，系统就不会检测是否还有其他 startId 存在）
        stopSelf(msg.arg1);//执行完 onHandleIntent 后会接着通过 stopSelf 方法自动停止（指定 startId 来停止当前的任务）
    }
}
public IntentService(String name) {
    super();
    mName = name;
}
public void setIntentRedelivery(boolean enabled) {
    mRedelivery = enabled;
}
@Override
public void onCreate() {
    super.onCreate();
    //创建 HandlerThread
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();
    //
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
@Override
public void onStart(@Nullable Intent intent, int startId) {
    //会将请求 Intent 添加到队列中
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
@Override
public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
    onStart(intent, startId);
    //mRedelivery 重新交付，是否需要重启服务，默认 false 不重启
    return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
}
@Override
public void onDestroy() {
    //
    mServiceLooper.quit();
}
@Override
@Nullable
public IBinder onBind(Intent intent) {
    return null;
}
//抽象方法
@WorkerThread
protected abstract void onHandleIntent(@Nullable Intent intent);
}
```

## 总结
- IntentService 一次只能处理一个请求，如果有多个请求，它们会被顺序执行，这可能出现延迟等问题
- 当 IntentService 处理完所有的任务后，它会自动停止，这种特性在某些情况下可能是不希望的行为，尤其是需要持续运行的任务
- IntentService 也是 Service，可以用 bind 的方式去启动，但是不推荐这么做，因为 IntentService#onBind 返回为 null，而且也和 IntentService 的自动停止的特性不太符合
- IntentService 已经被标记为过时了，可以考虑用 JobIntentService 或 WorkManager 代替
