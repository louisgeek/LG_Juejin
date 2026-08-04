# Android MediatorLiveData
- 可以监听多个 LiveData 源，并且能动态添加和移除源

## 合并多个 LiveData 源

```java
public class CombineViewModel extends ViewModel {
    private final MutableLiveData<One> _oneLiveData = new MutableLiveData<>();
    public final LiveData<One> oneLiveData = _oneLiveData;

    private final MutableLiveData<Two> _twoLiveData = new MutableLiveData<>();
    public final LiveData<Two> twoLiveData = _twoLiveData;

    //MediatorLiveData
    private final MediatorLiveData<CombinedData> _combinedData = new MediatorLiveData<>();
    public final LiveData<CombinedData> combinedData = _combinedData;

    private boolean isOneReceived = false;
    private boolean isTwoReceived = false;

    //构造函数
    public CombineViewModel() {
        // 监听 One 数据源
        _combinedData.addSource(_oneLiveData, one -> {
            isOneReceived = true;
            mergeData();
        });

        // 监听 Two 数据源
        _combinedData.addSource(_twoLiveData, two -> {
            isTwoReceived = true;
            mergeData();
        });
    }

    private void mergeData() {
        // 仅当两个数据源都收到新更新时，才执行合并
        if (isOneReceived && isTwoReceived) {
            // 获取当前两个数据源的最新值（允许 null）
            One currentOne = _oneLiveData.getValue();
            Two currentTwo = _twoLiveData.getValue();
            // 合并数据（即使有 null 也合并）
            _combinedData.setValue(new CombinedData(currentOne, currentTwo));
            // 重置标志位：保证每轮双源更新仅合并一次
            isOneReceived = false;
            isTwoReceived = false;
        }
    }

    @Override
    protected void onCleared() {
        super.onCleared();
        _combinedData.removeSource(_oneLiveData);
        _combinedData.removeSource(_twoLiveData);
        //
        //isOneReceived = false;
        //isTwoReceived = false;
    }
}
```

如果数据分布在不同 ViewModel 中
```java
public class CombineViewModel extends ViewModel {
    //MediatorLiveData
    private final MediatorLiveData<CombinedData> _combinedData = new MediatorLiveData<>();
    public final LiveData<CombinedData> combinedData = _combinedData;
    //
    private OneViewModel oneViewModel;
    private TwoViewModel twoViewModel;
    private boolean isOneReceived = false;
    private boolean isTwoReceived = false;

    public void init(ViewModelProvider viewModelProvider) {
        clearSources();

        oneViewModel = viewModelProvider.get(OneViewModel.class);
        twoViewModel = viewModelProvider.get(TwoViewModel.class);

        //监听 One 数据源
        _combinedData.addSource(oneViewModel.oneLiveData, one -> {
            isOneReceived = true;
            mergeData();
        });

        // 监听 Two 数据源
        _combinedData.addSource(twoViewModel.twoLiveData, two -> {
            isTwoReceived = true;
            mergeData();
        });
    }

    private void mergeData() {
        if (isOneReceived && isTwoReceived) {
            //双源都更新过才合并
            One currentOne = oneViewModel.oneLiveData.getValue();
            Two currentTwo = twoViewModel.twoLiveData.getValue();
            // 更新合并后的数据（允许null）
            _combinedData.setValue(new CombinedData(currentOne, currentTwo));
            // 重置标志位
            isOneReceived = false;
            isTwoReceived = false;
        }
    }

    //清理监听
    public void clearSources() {
        if (oneViewModel != null) {
            _combinedData.removeSource(oneViewModel.oneLiveData);
        }
        if (twoViewModel != null) {
            _combinedData.removeSource(twoViewModel.twoLiveData);
        }
        oneViewModel = null;
        twoViewModel = null;
        //
        isOneReceived = false;
        isTwoReceived = false;
    }

    @Override
    protected void onCleared() {
        super.onCleared();
        //
        clearSources();
    }
}
```


## Transformations#map
- 基于 MediatorLiveData
```kotlin
private val _userLD = MutableLiveData<User>()
val userLD: LiveData<User> = _userLD
//更新用户信息
fun updateUser(firstName: String, lastName: String, age: Int = 0) {
    _userLD.value = User(firstName, lastName, age)
}
// User 转成 String
val userFullNameLD: LiveData<String> = _userLD.map { user -> user.firstName + user.lastName }
// Int 转成 String
val ageTextLD: LiveData<String> = _userLD.map { user -> user.age.toString() }
```

感受一下源码
```kotlin
//相当于 MediatorLiveData 收集（合并） 1 个 LiveData 源
//从 LiveData<X> 转成 LiveData<Y>，比如 LiveData<User> 转成 LiveData<String> 
public fun <X, Y> LiveData<X>.map(mapFunction: Function<X, Y>): LiveData<Y> {
    val combinedData = MediatorLiveData<Y>()
    //监听 this 这个 LiveData 数据源
    //转换好需要的值后再调用 combinedData#setValue
    combinedData.addSource(this) { x -> combinedData.value = mapFunction.apply(x) }
    return combinedData
}
```

## Transformations#switchMap
- 基于 MediatorLiveData
```kotlin
private val _userIdLD: MutableLiveData<String> = MutableLiveData() 
val userIdLD: LiveData<String> = _userIdLD
fun updateUserId(userId: String) {
     _userIdLD.value = userId
}
//String 转 LiveData<User> （数据源动态切换）
val userLD: LiveData<User> = _userIdLD.switchMap { userId -> userRepository.getUserByUserId(userId) }
//
```

感受一下源码
```kotlin
public fun <X, Y> LiveData<X>.switchMap(switchMapFunction: Function<X, LiveData<Y>>): LiveData<Y> {
    val combinedData = MediatorLiveData<Y>()
    //监听 this 这个 LiveData 数据源
    combinedData.addSource(this, object : Observer<X> {
            //主要是为了记录旧数据源，用作对比和移除操作
            var liveData: LiveData<Y>? = null
            override fun onChanged(value: X) {
                //根据最新的 X 转成 LiveData(Y)
                //比如 switchMapFunction.apply("002") 返回用户 userId=002 的 LiveData<User> 信息
                val newLiveData = switchMapFunction.apply(value)
                if (liveData === newLiveData) {
                    //引用相等，说明 newLiveData 已经在被 combinedData 收集监听了，不需要继续 removeSource 再 addSource 重走一遍
                    return
                }
                if (liveData != null) {
                    //如果有旧的数据源，先移除监听，否则旧的数据还会被收集监听
                    combinedData.removeSource(liveData!!)
                }
                liveData = newLiveData
                if (liveData != null) {
                    combinedData.addSource(liveData!!) { y -> combinedData.setValue(y) }
                }
            }
        }
    )
    return combinedData
}
```

## ViewModel 复用 UseCase 的方法
```java
private final MutableLiveData<String> _liveDataSetConfig = new MutableLiveData<>();
//<success,param>     回传 param 是为了方便监听的时候使用
public final LiveData<Pair<Boolean, String>> liveDataSetConfig = Transformations.switchMap(_liveDataSetConfig, configUseCase::execute);
public void setConfig(String param) {
    _liveDataSetConfig.setValue(param);
}
```

```java
//ConfigUseCase
public LiveData<Pair<Boolean, String>> execute(String param) {
        MutableLiveData<Pair<Boolean, String>> result = new MutableLiveData<>();
        ThreadUtil.runXxx(() -> {
            try {
                //耗时操作
                boolean success = Api.setConfig(param);
                XLog.i(TAG + " setConfig success= " + success);
                result.postValue(Pair.create(success, param));
            } catch (Exception e) {
                result.postValue(Pair.create(false, param));
            }
        });
        return result;
}
```

