# Android 内存泄漏
- 内存泄漏：本该被 GC 回收的内存实际没有被回收，由于不再使用的对象仍然被其他对象持有引用，所以导致 GC 无法回收它们，后续导致内存占用不断增加，最终导致设备性能下降
- 根本原因：短生命周期的对象被长生命周期的对象持有引用，导致短生命周期对象的内存在其生命周期结束后无法被 GC 回收，从而导致了内存泄露

## 可能导致内存泄漏的场景

### 1 静态变量
- 静态变量（类变量）在 Java 中是类级别的，通常它的生命周期和应用程序的生命周期相同，如果一个静态变量持有了一个对象的引用，而这个对象的生命周期如果比静态变量短，但由于静态变量的存在，这个对象无法被垃圾回收器回收，直到应用程序结束，从而导致内存泄漏
- 比如把一个 Activity 的 Context 赋值给一个静态变量，然后未及时置空变量释放引用
```java
public class MainActivity extends AppCompatActivity {

    public static MainActivity mainActivity;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //
        mainActivity = this;
    }
}
```

### 2 静态集合
- 静态集合持有了被添加进来的对象的引用，解决办法可以是及时移除释放引用，或者是在集合一开始存放的时候就引入 WeakReference 弱引用进行规避

### 3 单例类
- 单例类的生命周期通常和应用程序的生命周期相同，假设单例类持有了一个 Activity 的 Context 的引用，当该 Activity 在生命周期结束后本该被 GC 回收的却未能被回收，就导致了内存泄露，可以考虑改用 Application 的 Context

```java
public class Singleton {
    private Context context;

    private static Singleton sInstance;

    private Singleton(Context context) {
        this.context = context; //可以改用 this.context = context.getApplicationContext(); 避免
    }

    public synchronized static Singleton getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new Singleton(context);
        }
        return sInstance;
    }
}
```

### 4 非静态内部类或匿名内部类
- 每一层内部类由一个 `$` 符号隔开，有名字的内部类在 `$` 后面的是类的名字（`OuterClass$InnerClass.class`），而匿名内部类在 `$` 后面的是数字序号（`OuterClass$1.class`），从 1 开始递增
- 非静态内部类或匿名内部类都可以访问外部类的变量，非静态内部类 `OuterClass$InnerClass.class` 通过变量 `this.this$0` 持有外部类的引用，而匿名内部类 `OuterClass$1.class` 通过变量 `this.this$0` 持有外部类的引用，这个变量都是编译器添加的
- 由于非静态内部类或匿名内部类都会隐式地持有外部类的引用，如果非静态内部类或匿名内部类的生命周期长于外部类，而外部类已经不再需要时，外部类无法被 GC 回收，从而就导致了内存泄露
- 比如一个 Activity 中创建了非静态内部类或匿名内部类的实例，而这个实例在 Activity 生命周期结束后仍然存在，就会导致这个 Activity 无法被 GC 回收

### 5 Handler 相关
- MainActivity 生命周期结束时，如果 Handler 消息队列中还有未处理完的消息，所以要考虑及时移除消息，将 Handler 改成静态内部类和引入 WeakReference 弱引用
```java
public class MainActivity extends AppCompatActivity {
    //使用匿名内部类的方式创建 Handler，持有外部类 MainActivity 的引用，可能会导致内存泄露
    public Handler handler = new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            //
        }
    };
    //
    public Handler handler2 = new Handler();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //使用匿名内部类的方式创建 Runnable，持有外部类 MainActivity 的引用，可能会导致内存泄露
        handler2.post(new Runnable() {
            @Override
            public void run() {
                //
            }
        });
    }
}
```
### 6 线程相关
- MainActivity 生命周期结束时，线程任务还没有执行完毕的情况，所以要考虑及时终止线程

### 7 资源未关闭相关
- 数据库连接、网络连接、IO 连接（比如 File、Stream）、Cursor 游标和 Bitmap（通常不需要手动回收，除非大量大尺寸或者很明确不再使用的）等资源使用后忘记关闭
- 注册 BroadcastReceiver 广播时忘记反注册，设置完自定义监听器后忘记在合适的地方 remove 移除，EventBus 等观察者模式忘记反注册
- Service 服务执行完成后忘记 stopSelf
- WebView 内存泄漏
- 动画相关的内存泄露

## 避免内存泄漏方法
- 1 减少静态对象的使用，需要及时进行清理（比如赋值为 null）释放引用
- 2 确保静态集合中的对象在不需要的时候能够被及时移除
- 3 使用 WeakReference 弱引用
- 4 减少单例类的使用，约束滥用行为
- 5 尽可能的改用 Application 的 Context
- 6 避免使用非静态内部类或匿名内部类
- 7 注意 Handler 导致的内存泄漏，及时在 onStop 或 onDestroy 时候用 removeCallbacksAndMessages 移除消息
- 8 针对线程相关及时做好取消、终止等工作
- 9 针对资源性对象（比如数据库、File 等）、WebView 和动画等需要及时做好回收关闭工作，做好 BroadcastReceiver 广播、监听器和观察者等的移除反注册工作
- 10 多使用 try–catch-finally、try-with-resources 特性或者 Kotlin 的 T.use 函数
 