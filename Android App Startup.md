# Android App Startup
- 提供了一个 ContentProvider 来运行所有依赖项的初始化，避免每个第三方库单独使用一个 ContentProvider 进行初始化，从而提高了应用的程序的启动速度
- 提供了简单的 API 来声明初始化组件之间的依赖关系，确保依赖项按照正确的顺序被初始化
- 所有初始化逻辑都集中在 Initializer 类中，便于管理和维护
- 确保每个组件只被初始化一次，即使它们被多个地方引用或依赖

```groovy
implementation 'androidx.startup:startup-runtime:1.2.0'
```

Initializer 接口
```java
//androidx.startup.Initializer
/**
 * Initializes library components during app startup.
 *
 * Discovered and initialized by {@link InitializationProvider}.
 *
 * @param <T> The type of the component being initialized.
 */
public interface Initializer<T> {

    /**
     * Initializes a library component within the application {@link Context}.
     *
     * @param context The application context.
     */
    @NonNull
    T create(@NonNull Context context);

    /**
     * Gets a list of this initializer's dependencies.
     *
     * Dependencies are initialized before the dependent initializer. For
     * example, if initializer A defines initializer B as a dependency, B is
     * initialized before A.
     *
     * @return A list of initializer dependencies.
     */
    @NonNull
    List<Class<? extends Initializer<?>>> dependencies();
}
```

InitializationProvider 类

```java
//androidx.startup.InitializationProvider
/**
 * The {@link ContentProvider} which discovers {@link Initializer}s in an application and
 * initializes them before {@link Application#onCreate()}.
 */
public class InitializationProvider extends ContentProvider {

    @Override
    public final boolean onCreate() {
        Context context = getContext();
        if (context != null) {
            // Many Initializer's expect the `applicationContext` to be non-null. This
            // typically happens when `android:sharedUid` is used. In such cases, we postpone
            // initialization altogether, and rely on lazy init.
            // More context: b/196959015
            Context applicationContext = context.getApplicationContext();
            if (applicationContext != null) {
                // Pass the class context so the right metadata can be read.
                // This is especially important in the context of apps that want to use
                // InitializationProvider in multiple processes.
                // b/183136596#comment18
                AppInitializer.getInstance(context).discoverAndInitialize(getClass());
            } else {
                StartupLogger.w("Deferring initialization because `applicationContext` is null.");
            }
        } else {
            throw new StartupException("Context cannot be null");
        }
        return true;
    }

    @Nullable
    @Override
    public final Cursor query(
            @NonNull Uri uri,
            @Nullable String[] projection,
            @Nullable String selection,
            @Nullable String[] selectionArgs,
            @Nullable String sortOrder) {
        throw new IllegalStateException("Not allowed.");
    }

    @Nullable
    @Override
    public final String getType(@NonNull Uri uri) {
        throw new IllegalStateException("Not allowed.");
    }

    @Nullable
    @Override
    public final Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        throw new IllegalStateException("Not allowed.");
    }

    @Override
    public final int delete(
            @NonNull Uri uri,
            @Nullable String selection,
            @Nullable String[] selectionArgs) {
        throw new IllegalStateException("Not allowed.");
    }

    @Override
    public final int update(
            @NonNull Uri uri,
            @Nullable ContentValues values,
            @Nullable String selection,
            @Nullable String[] selectionArgs) {
        throw new IllegalStateException("Not allowed.");
    }
}
```

## 实现 Initializer 接口
- 通过重写 dependencies 方法来实现指定组件的初始化顺序

```kotlin
class WorkManagerInitializer: Initializer<WorkManager> {
    override fun create(context: Context): WorkManager {
        //androidx.work.Configuration
        val configuration = Configuration.Builder().build()
        WorkManager.initialize(context, configuration)
        return WorkManager.getInstance(context)
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {
        // No dependencies on other libraries.
        return listOf()
    }
}
//
class ExampleLoggerInitializer:Initializer<ExampleLogger> {
    override fun create(context: Context): ExampleLogger {
        // WorkManager.getInstance() is non-null only after
        // WorkManager is initialized.
        return ExampleLogger(WorkManager.getInstance(context))
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {
        // Defines a dependency on WorkManagerInitializer so it can be
        // initialized after WorkManager is initialized.
        return listOf(WorkManagerInitializer::class.java)
    }
}
```

## 注册 Initializer
- 可以在 meta-data 条目中使用 tools:node="remove" 停用单个组件的自动初始化
- 可以在 provider 条目中使用 tools:node="remove" 停用所有组件的自动初始化
- 这里无需为 WorkManagerInitializer 添加 meta-data 条目， 因为 WorkManagerInitializer 是 ExampleLoggerInitializer 的依赖项，可以省略

```xml
<!-- tools:node="merge" 属性合并解决冲突 -->
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes ExampleLoggerInitializer discoverable. -->
    <meta-data  android:name="com.example.ExampleLoggerInitializer"
          android:value="androidx.startup" />
</provider>
```


## 手动初始化组件
- 这里会自动启动 WorkManagerInitializer，WorkManagerInitializer 是 ExampleLoggerInitializer 的依赖项，可以省略

```kotlin
 val appInitializer = AppInitializer.getInstance(context)
 //子依赖项可以省略
 //appInitializer.initializeComponent(WorkManagerInitializer::class.java)
 appInitializer.initializeComponent(ExampleLoggerInitializer::class.java)
```






