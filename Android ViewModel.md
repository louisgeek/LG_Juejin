# Android ViewModel
- ViewModel 是一个用于存储和管理与 UI 界面相关数据的组件，适用于需要在 Activity 或 Fragment 之间共享数据或者需要在配置更改（比如屏幕旋转）后仍旧能保持数据，从而避免了数据丢失和重复初始化
- 当 Activity 或 Fragment 在因配置更改后被销毁并重建的时候，其关联的 ViewModel 实例不会被销毁，而是会被保留下来，存放在内存中，这意味着可以使用 ViewModel 来保存那些不想随着配置更改而丢失的数据（比如用户输入的数据），直到 Activity 或 Fragment 完全销毁后销毁
- 每个 Activity 或 Fragment 都有自己对应的 ViewModelStore，用于存储该 Activity 或 Fragment 关联的 ViewModel 实例，当通过 ViewModelProvider 获取 ViewModel 时，如果 ViewModelStore 中的 ViewModel 实例不存在，则系统会使用反射来创建一个新的 ViewModel 实例并通过唯一 Key 值将其存储到 ViewModelStore 中，同样也通过这个 Key 值去进行获取，如果实例已经存在，就直接返回获取到的现有的 ViewModel 实例
- Activity 在因为配置更改而销毁重建过程中会先调用 Activity#onRetainNonConfigurationInstance 保存 ViewModelStore 实例，在重建后通过 Activity#getLastNonConfigurationInstance 方法获取之前的 ViewModelStore 实例，所以 onRetainNonConfigurationInstance 方法和 getLastNonConfigurationInstance 是成对出现的

## ViewModel 实例化
```kotlin
//这个 this 就是 ViewModelStoreOwner
val mainViewModel = ViewModelProvider(this).get(MainViewModel::class.java) //ViewModel 类类型
```

## ViewModelProvider
- 通常使用 ViewModelProvider#get 方法去创建 ViewModel 实例，如果 ViewModel 有其他依赖，那么可以通过实现 ViewModelProvider.Factory 接口来提供自定义的 ViewModel 实例化逻辑去创建 ViewModel 实例
- ViewModelProvider#get 方法会根据提供的 ViewModel 类类型和自定义工厂来创建 ViewModel 实例，如果没有提供自定义的工厂，系统会使用默认的工厂

```kotlin
@JvmOverloads
constructor(
    private val store: ViewModelStore,
    private val factory: Factory,
    private val defaultCreationExtras: CreationExtras = CreationExtras.Empty,
) {
    //...
    public constructor(
        owner: ViewModelStoreOwner
        //默认工厂是 SavedStateViewModelFactory
    ) : this(owner.viewModelStore, defaultFactory(owner), defaultCreationExtras(owner))

    public constructor(owner: ViewModelStoreOwner, factory: Factory) : this(
        owner.viewModelStore,
        factory,
        defaultCreationExtras(owner)
    )
    @MainThread
    public open operator fun <T : ViewModel> get(modelClass: Class<T>): T {
        val canonicalName = modelClass.canonicalName
            ?: throw IllegalArgumentException("Local and anonymous classes can not be ViewModels")
        //androidx.lifecycle.ViewModelProvider.DefaultKey
        return get("$DEFAULT_KEY:$canonicalName", modelClass)
    }
    @MainThread
    public open operator fun <T : ViewModel> get(key: String, modelClass: Class<T>): T {
        //ViewModelStore
        val viewModel = store[key]
        if (modelClass.isInstance(viewModel)) {
            (factory as? OnRequeryFactory)?.onRequery(viewModel!!)
            return viewModel as T
        } else {
            @Suppress("ControlFlowWithEmptyBody")
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        val extras = MutableCreationExtras(defaultCreationExtras)
        extras[VIEW_MODEL_KEY] = key
        // AGP has some desugaring issues associated with compileOnly dependencies so we need to
        // fall back to the other create method to keep from crashing.
        return try {
            factory.create(modelClass, extras)
        } catch (e: AbstractMethodError) {
            factory.create(modelClass)
        }.also { store.put(key, it) }
    }
    //...
}
```

## ViewModelStoreOwner
```kotlin
interface ViewModelStoreOwner {

    /**
     * The owned [ViewModelStore]
     */
    val viewModelStore: ViewModelStore
}
```
 
## ComponentActivity
- ComponentActivity 实现了 ViewModelStoreOwner 接口，且拥有一个 mViewModelStore 成员变量
```java
public class ComponentActivity extends androidx.core.app.ComponentActivity implements
        ContextAware,
        LifecycleOwner,
        ViewModelStoreOwner, //ComponentActivity 实现了 ViewModelStoreOwner 接口
        HasDefaultViewModelProviderFactory,
        //...
{
//...
//ComponentActivity 的静态内部类 NonConfigurationInstances
static final class NonConfigurationInstances {
    Object custom;
    ViewModelStore viewModelStore;
}
//...
// Lazily recreated from NonConfigurationInstances by getViewModelStore()
private ViewModelStore mViewModelStore;
//...
public ComponentActivity() {
    //...
    getLifecycle().addObserver(new LifecycleEventObserver() {
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            if (event == Lifecycle.Event.ON_DESTROY) {
                //...
                //当生命周期执行到 onDestroy 时对 ViewModel 进行销毁
                // And clear the ViewModelStore
                if (!isChangingConfigurations()) { //不是配置更改的场景
                    //调用 ViewModelStore 的 clear 方法
                    getViewModelStore().clear();
                }
                //...
            }
        }
    });
    //...
}
//...
@NonNull
@Override
public ViewModelStore getViewModelStore() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    ensureViewModelStore();
    return mViewModelStore;
}
@SuppressWarnings("WeakerAccess") /* synthetic access */
void ensureViewModelStore() {
    if (mViewModelStore == null) {
        //尝试获取之前的 ViewModelStore 实例（存放在静态内部类 NonConfigurationInstances 中）
        NonConfigurationInstances nc = (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            // Restore the ViewModelStore from NonConfigurationInstances
            mViewModelStore = nc.viewModelStore;
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
    }
}
//...
}      
```

## Fragment
- Fragment 的 ViewModelStore 是通过 FragmentManager 进行管理获取的
```java
public class Fragment implements ComponentCallbacks, OnCreateContextMenuListener, LifecycleOwner,
        //Fragment 实现了 ViewModelStoreOwner 接口
        ViewModelStoreOwner, HasDefaultViewModelProviderFactory, SavedStateRegistryOwner,
        ActivityResultCaller {
//...
@NonNull
@Override
public ViewModelStore getViewModelStore() {
    if (mFragmentManager == null) {
        throw new IllegalStateException("Can't access ViewModels from detached fragment");
    }
    if (getMinimumMaxLifecycleState() == Lifecycle.State.INITIALIZED.ordinal()) {
        throw new IllegalStateException("Calling getViewModelStore() before a Fragment "
                + "reaches onCreate() when using setMaxLifecycle(INITIALIZED) is not "
                + "supported");
    }
    return mFragmentManager.getViewModelStore(this);
}
//...

//androidx.fragment.app.FragmentManager
//...
private FragmentManagerViewModel mNonConfig;
//...
@NonNull
ViewModelStore getViewModelStore(@NonNull Fragment f) {
    //mNonConfig 是一个 FragmentManagerViewModel 对象
    return mNonConfig.getViewModelStore(f);
}
//...

//androidx.fragment.app.FragmentManagerViewModel
//...
private final HashMap<String, ViewModelStore> mViewModelStores = new HashMap<>();
//...
@NonNull
ViewModelStore getViewModelStore(@NonNull Fragment f) {
    ViewModelStore viewModelStore = mViewModelStores.get(f.mWho);
    if (viewModelStore == null) {
        viewModelStore = new ViewModelStore();
        mViewModelStores.put(f.mWho, viewModelStore);
    }
    return viewModelStore;
}
void clearNonConfigState(@NonNull Fragment f) {
    //...
    // Clear and remove the Fragment's ViewModelStore
    ViewModelStore viewModelStore = mViewModelStores.get(f.mWho);
    if (viewModelStore != null) {
        //调用 ViewModelStore 的 clear 方法
        viewModelStore.clear();
        //
        mViewModelStores.remove(f.mWho);
    }
}
//...
```

## ViewModelStore
- ViewModelStore 里面维护了一个 HashMap，根据 Key 值来对 ViewModel 值进行存储，同时提供 clear 方法，可以将所有 ViewModel 清除

```kotlin
open class ViewModelStore {
    //LinkedHashMap
    private val map = mutableMapOf<String, ViewModel>()
    /**
     * @hide
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    fun put(key: String, viewModel: ViewModel) {
        val oldViewModel = map.put(key, viewModel)
        oldViewModel?.onCleared()
    }
    /**
     * Returns the `ViewModel` mapped to the given `key` or null if none exists.
     */
    /**
     * @hide
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    operator fun get(key: String): ViewModel? {
        return map[key]
    }
    /**
     * @hide
     */
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    fun keys(): Set<String> {
        return HashSet(map.keys)
    }
    /**
     * Clears internal storage and notifies `ViewModel`s that they are no longer used.
     */
    fun clear() {
        //循环调用 ViewModel 的 clear 方法，内部会调用 ViewModel#onCleared 方法，可按需重写做一些释放资源的操作
        for (vm in map.values) {
            vm.clear()
        }
        map.clear()
    }
}
```

## NonConfigurationInstances 存取
```java
//Activity 销毁时保存 NonConfigurationInstances
ActivityThread#performDestroyActivity
-Activity#retainNonConfigurationInstances 返回一个 NonConfigurationInstances 对象然后保存到 ActivityThread$ActivityClientRecord#lastNonConfigurationInstances 变量里

//Activity 创建的时候取出 NonConfigurationInstances
ActivityThread#performLaunchActivity
-Activity#attach 把之前的 ActivityThread$ActivityClientRecord#lastNonConfigurationInstances 存放的值赋给 Activity#mLastNonConfigurationInstances 变量里

//ComponentActivity 使用 NonConfigurationInstances 里的 ViewModelStore
ComponentActivity#getViewModelStore
-ComponentActivity#ensureViewModelStore
--Activity#getLastNonConfigurationInstance 里就返回了上述的 Activity#mLastNonConfigurationInstances 对象
```
 
## 总结
- ViewModel 的主要作用：
    - 在配置更改（比如屏幕旋转）时保持数据，避免数据丢失和重复初始化
    - 能够实现在 Activity 和 Fragment 之间共享数据
- 通过 ViewModelProvider 创建或获取 ViewModel 实例，结合 LiveData 或 StateFlow 等组件进行数据观察和更新，可以实现在数据变化时更新 UI
- ViewModel 的生命周期往往比 Activity 或 Fragment 的生命周期长，因为 ViewModel 在对应关联的 Activity 或 Fragment 创建时被实例化，并一直保存在内存中（所以要避免存放大量的不必要的大数据），直到其关联的 Activity 或 Fragment 经历完整的生命周期直至销毁，可按需重写 onCleared 去执行一些清理操作（比如关闭数据库连接、取消网络请求或移除监听器等）来确保资源会被适当地释放，避免可能出现的内存泄漏


 


