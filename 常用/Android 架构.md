# Android 架构
- 使用 ViewModel 而非 AndroidViewModel：不建议使用 AndroidViewModel，不应该在 ViewModel 中使用 Application 类，应该将依赖项移至界面层或数据层
- 如果是要共享数据的话，应该在业务层或者repo层就做了，不应该在vm这层来做。


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

Android 最新的架构规范中，引入了 Domain Layer（译为领域层or网域层），建议大家使用 UseCase 来封装一些复杂的业务逻辑。 --多个 ViewModel 可以复用同一个 UseCase，每个UseCase专注一个业务场景（比如GetTopHeadlinesUseCase、GetTopNewsUseCase 、SearchNewsUseCase、SaveImageUseCase  动词命名）
UseCase的目的是成为 ViewModels 和 Repository 之间的中介。--调用一个或多个 Repository，对数据做进一步处理（过滤、排序、转换、组合等）。，比如数据转换、组合多个 Repository 调用或添加业务规则。--ViewModel负责 UI 状态管理，UseCase封装业务逻辑
UseCase 用来封装可复用的单一业务逻辑。这里的两个关键词一个是单一、一个是业务逻辑、不持有状态。--最好不要沦为只是一个 Repository 的包装器
UseCase 不应该依赖任何 Android 平台相关对象，甚至 Context、UseCase 内部不应该包含 mutable 的数据
- UseCase 应该是 Main-safe 的，即可以在主线程安全的调用，其中的耗时处理应该自动切换到后台线程
单一职责的 UseCase 应该只有一个方法或一类重载方法，而且方法最好是纯函数逻辑，不依赖 UseCase 对象的任何状态，因此从这个角度讲，UseCase 可以是一个单例，直接使用 object 定义---所以不应该把 UseCase 搞成抽象的

