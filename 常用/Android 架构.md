# Android 架构
- 使用 ViewModel 而非 AndroidViewModel：不建议使用 AndroidViewModel，不应该在 ViewModel 中使用 Application 类，应该将依赖项移至界面层或数据层
- 如果是要共享数据的话，应该在业务层或者repo层就做了，不应该在vm这层来做。如果多个 ViewModel 需要共享同一份用户数据，它们应该各自通过 Repository 获取，而不是在 ViewModel 层手动共享数据（比如通过单例或全局变量）。Repository 层本身会负责保证数据的一致性和共享性。

## 命名规范
- Repository 接口
NewsRepository（以往是 INewsRepository、NewsRepositoryInterface）

- Repository 实现 
DefaultNewsRepository（默认）、OfflineFirstNewsRepository、FakeNewsRepository（以往是 NewsRepository、NewsRepositoryImpl）

data
- local
-- entitity
-- dao
-- XxxDatabase
- network remote
-- NetApi
-- model
- repository
- model

```kotlin
ViewModel 里的模板代码
private val _loading MutableLiveDataString = MutableLiveData()
val loading LiveDataString = _loading
```

 


data class NewsState(val list: List<Article>, val isLoading: Boolean)
class NewsViewModel : ViewModel() {
    private val _state = MutableLiveData<NewsState>()
    val state: LiveData<NewsState> = _state.asLiveData()

    fun loadNews() {
        viewModelScope.launch {
            _state.postValue(NewsState(isLoading = true))
            val data = repository.fetchNews() // 异步获取数据
            _state.postValue(data.copy(isLoading = false))
        }
    }
}

## UI ViewModel
- 核心职责是 "准备和管理 UI 所需的数据"，它应该专注于与 UI 交互相关的逻辑，比如处理用户输入、维护 UI 状态、将数据转换为 UI 友好的格式等。它的生命周期与 UI 控制器（Activity/Fragment）绑定，不适合承担数据共享的职责。
- 仅作为 UI 层与业务层之间的桥梁，负责将数据以 UI 友好的形式暴露给界面（如通过 LiveData、StateFlow 等），并处理用户交互事件。它不应该直接持有或操作全局共享的数据。
调用业务层的 UseCase，将结果暴露给 UI。
管理 UI 状态（如加载中、成功、错误）。
不直接处理数据共享逻辑。
管理与 UI 相关的临时状态（如加载中、错误提示等）
处理用户输入事件并转换为业务逻辑调用
通过 LiveData/StateFlow 等机制向 UI 暴露不可变数据
暴露UI状态（通过LiveData/StateFlow）
• 处理用户交互事件
• 管理屏幕配置变化时的数据持久化
• 协调UI层与业务逻辑层的交互
 LiveData 应该避免用在 Domain 甚至 Data 层等非 UI 场景（尤其应该避免在 Repo 中的使用）
ViewModel 为 View 暴露的 API 应该是非挂起且无法返回值的方法（不对外暴露 suspend 挂起方法）

## Data Repository
- 是数据的 "统一出入口"，负责协调本地数据源（数据库、SP 等）和远程数据源（API），本身就具备数据聚合和分发的能力。在这一层处理数据共享，可以确保所有依赖该数据的组件（包括多个 ViewModel）获取到的是一致的数据，避免数据不一致问题。
- 统一管理数据的加载、缓存和更新，业务层则负责封装共享的业务规则。这样既能保证数据共享，又能维持各层的清晰职责。
负责数据的获取、缓存、持久化以及与数据源的交互（如网络或数据库）。如果多个模块需要共享数据，应该在 Repository 层实现统一的访问接口，确保数据的一致性和共享性。
封装所有数据源（本地数据库、网络请求、文件存储等）。
提供统一的接口供上层调用，隐藏底层实现细节。
处理数据的缓存、同步、异步加载等逻辑
作为单一可信数据源（Single Source of Truth）
协调本地数据库、网络 API 等不同数据源
提供不可变数据给上层，避免外部直接操作数据源
通常以单例形式

## Domain UseCase
跨模块数据聚合（如用户信息需合并本地缓存与网络数据）
复杂业务规则校验（如支付流程中的多步骤验证）
数据共享的统一入口
封装具体的业务逻辑（如登录、注册、数据过滤等）。
调用 Repository 提供的数据方法，并对结果进行加工（如格式转换、错误处理）。
作为 ViewModel 与数据层之间的桥梁。
封装具体的业务规则，协调 Repository 层完成业务需求。如果数据共享涉及复杂的业务逻辑（如权限控制、数据转换），应该在业务层处理
- 负责封装具体的业务逻辑，当多个模块需要共享某部分业务数据或逻辑时，在这一层实现共享，可以避免业务逻辑在多个 ViewModel 中重复，同时保持业务规则的单一性。
Android 最新的架构规范中，引入了 Domain Layer（译为领域层or网域层），建议大家使用 UseCase 来封装一些复杂的业务逻辑。 --多个 ViewModel 可以复用同一个 UseCase，每个UseCase专注一个业务场景（比如GetTopHeadlinesUseCase、GetTopNewsUseCase 、SearchNewsUseCase、SaveImageUseCase  动词命名）
UseCase的目的是成为 ViewModels 和 Repository 之间的中介。--调用一个或多个 Repository，对数据做进一步处理（过滤、排序、转换、组合等）。，比如数据转换、组合多个 Repository 调用或添加业务规则。--ViewModel负责 UI 状态管理，UseCase封装业务逻辑
UseCase 用来封装可复用的单一业务逻辑。这里的两个关键词一个是单一、一个是业务逻辑、不持有状态。--最好不要沦为只是一个 Repository 的包装器
UseCase 不应该依赖任何 Android 平台相关对象，甚至 Context、UseCase 内部不应该包含 mutable 的数据 与 UI 或平台无关的代码（如网络请求、数据库操作、数据转换等）（将 Context 相关操作移至 Repository 或 Data Source）（或者通过接口抽象隐藏 Context 依赖，比如获取字符串供 UseCase 使用）
- UseCase 应该是 Main-safe 的，即可以在主线程安全的调用，其中的耗时处理应该自动切换到后台线程
单一职责的 UseCase 应该只有一个方法或一类重载方法，而且方法最好是纯函数逻辑，不依赖 UseCase 对象的任何状态，因此从这个角度讲，UseCase 可以是一个单例，直接使用 object 定义---所以不应该把 UseCase 搞成抽象的








```kotlin
data class CounterState(
    val count: Int = 0,
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)
```

UIIntent
用户操作意图，通常用密封类表示
- 用户操作或系统事件被封装为Intent对象（如点击按钮、网络请求），作为状态变更的触发器
- 使用密封类封装用户操作或系统事件：

```kotlin
sealed class CounterIntent {
    object Increment : CounterIntent()
    object Decrement : CounterIntent()
    data class SetCount(val value: Int) : CounterIntent()
}
```


```kotlin
//处理意图、更新状态，并通过LiveData暴露状态：
class CounterViewModel : ViewModel() {
    //用 LiveData 承载 State 并通知 UI 更新
    private val _state = MutableLiveData(CounterState())
    val state: LiveData<CounterState> = _state

    fun processIntent(intent: CounterIntent) {
        when (intent) {
            is CounterIntent.Increment -> updateState { it.copy(count = it.count + 1) }
            is CounterIntent.Decrement -> updateState { it.copy(count = it.count - 1) }
            is CounterIntent.SetCount -> updateState { it.copy(count = intent.value) }
        }
    }

    private fun updateState(transform: (CounterState) -> CounterState) {
        val currentState = _state.value ?: return
        _state.value = transform(currentState)
    }


    // ViewModel中暴露属性级LiveData  结合Transformations.map或switchMap实现属性级观察，减少不必要的UI更新。
val countLiveData = Transformations.map(state) { it.count }
val isLoadingLiveData = Transformations.map(state) { it.isLoading }

}
```

```kotlin
//观察LiveData状态并更新UI，同时将用户事件转化为Intent：
class CounterActivity : AppCompatActivity() {
    private val viewModel: CounterViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_counter)

        // 观察状态变化
        viewModel.state.observe(this) { state ->
            countTextView.text = state.count.toString()
            progressBar.isVisible = state.isLoading
            errorMessageTextView.text = state.errorMessage ?: ""
        }

        // 绑定用户事件到Intent
        incrementButton.setOnClickListener {
            viewModel.processIntent(CounterIntent.Increment)
        }
        decrementButton.setOnClickListener {
            viewModel.processIntent(CounterIntent.Decrement)
        }


        // View中分别观察
       
viewModel.countLiveData.observe(this) { count ->
    countTextView.text = count.toString()
}
viewModel.isLoadingLiveData.observe(this) { isLoading ->
    progressBar.isVisible = isLoading
}

    }
}
```




```kotlin
// Intent
sealed class DemoIntent {
    object LoadData : DemoIntent()
}

// 状态
通过 ViewState 封装所有 UI 状态，对外暴露一个 LiveData
通过 LiveData 的个别属性监听（如 distinctUntilChanged）实现局部 UI 更新。
data class DemoState(
    val data: List<String> = emptyList(),
    val loading: Boolean = false
)

data class MainViewState(
    val fetchStatus: FetchStatus = FetchStatus.NotFetched,
    val newsList: List<NewsItem> = emptyList()
)

// ViewModel
class DemoViewModel : ViewModel() {
    private val _state = MutableLiveData(DemoState())
    val state: LiveData<DemoState> = _state

    fun dispatch(intent: DemoIntent) {
        when (intent) {
            is DemoIntent.LoadData -> loadData()
        }
    }

    private fun loadData() {
        _state.value = _state.value?.copy(loading = true)
        viewModelScope.launch {
            val result = repository.getData()
            _state.value = DemoState(data = result, loading = false)
        }
    }
}

class MainViewModel : ViewModel() {
    private val _viewState = MutableLiveData<MainViewState>(MainViewState())
    val viewState = _viewState

    fun fetchNews() {
        // 更新状态为加载中
        _viewState.value = _viewState.value?.copy(fetchStatus = FetchStatus.Fetching)
        
        // 模拟网络请求
        viewModelScope.launch {
            when (val result = repository.getMockApiResponse()) {
                is Success -> {
                    _viewState.value = _viewState.value?.copy(
                        fetchStatus = FetchStatus.Success,
                        newsList = result.data
                    )
                }
                is Error -> {
                    _viewState.value = _viewState.value?.copy(
                        fetchStatus = FetchStatus.Error
                    )
                }
            }
        }
    }
}


class MainFragment : Fragment() {
    private val viewModel: MainViewModel by viewModels()
    private val binding by viewBinding(FragmentMainBinding::inflate)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // 观察 ViewState
        viewModel.viewState.observe(viewLifecycleOwner) { state ->
            binding.progressBar.isVisible = state.fetchStatus == FetchStatus.Fetching
            binding.errorText.isVisible = state.fetchStatus == FetchStatus.Error
            binding.recyclerView.isVisible = state.fetchStatus == FetchStatus.Success
            binding.recyclerView.adapter = NewsAdapter(state.newsList)
        }

        // 用户交互触发 Intent
        binding.refreshButton.setOnClickListener {
            viewModel.fetchNews()
        }
    }
}

//局部刷新方案
fun <T, A> LiveData<T>.observeState(
    lifecycleOwner: LifecycleOwner,
    prop: KProperty1<T, A>,
    action: (A) -> Unit
) {
    this.map { prop.get(it) }
        .distinctUntilChanged() // 防抖
        .observe(lifecycleOwner, Observer(action))
}

// 使用示例
viewModel.viewState.observeState(
    viewLifecycleOwner,
    MainViewState::newsList,
    { newsList -> binding.recyclerView.adapter = NewsAdapter(newsList) }
)
```



 通过 distinctUntilChanged 防止重复渲染。

```kotlin
（1）定义 Intent（用户意图）

// 密封类表示所有可能的用户意图
sealed class UserIntent {
    object LoadUser : UserIntent() // 加载用户数据的意图
    data class UpdateUserName(val newName: String) : UserIntent() // 更新用户名的意图
}

（2）定义 State（UI 状态）
// 密封类表示所有可能的UI状态（不可变）
//UIState 推荐采用不可变 data class 数据类（可以使用其 copy 方法创建新对象后更新状态）封装UI所需的所有状态,任何状态变更均通过生成新对象实现，避免意外修改，也可以使用 sealed class，自行处理创建新对象
sealed class UserState {
    object Loading : UserState() // 加载中状态
    data class Success(val user: User) : UserState() // 数据成功状态
    data class Error(val message: String) : UserState() // 错误状态
}

// 数据模型
data class User(val id: String, val name: String)

（3）ViewModel（处理 Intent，通过 LiveData 暴露 State）
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    // 私有LiveData用于内部更新状态
    private val _state = MutableLiveData<UserState>(UserState.Loading)
    // 暴露给View的不可变LiveData（只允许观察，不允许修改）
    val state: LiveData<UserState> = _state

    // 接收View发送的Intent
    fun handleIntent(intent: UserIntent) {
        when (intent) {
            is UserIntent.LoadUser -> loadUser()
            is UserIntent.UpdateUserName -> updateUserName(intent.newName)
        }
    }

    private fun loadUser() {
        viewModelScope.launch {
            _state.value = UserState.Loading
            try {
                val user = repository.getUser() // 调用仓库层获取数据
                _state.value = UserState.Success(user)
            } catch (e: Exception) {
                _state.value = UserState.Error(e.message ?: "加载失败")
            }
        }
    }

    private fun updateUserName(newName: String) {
        // 类似处理更新逻辑...
    }
}
（4）View（观察 State，发送 Intent）
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)

        // 观察State变化，更新UI
        viewModel.state.observe(this) { state ->
            when (state) {
                is UserState.Loading -> showLoading()
                is UserState.Success -> showUser(state.user)
                is UserState.Error -> showError(state.message)
            }
        }

        // 点击按钮发送"加载数据"的Intent
        btnLoad.setOnClickListener {
            viewModel.handleIntent(UserIntent.LoadUser)
        }
    }

    // UI更新方法...
    private fun showLoading() { /* 显示加载框 */ }
    private fun showUser(user: User) { /* 展示用户数据 */ }
    private fun showError(message: String) { /* 显示错误提示 */ }
}

```