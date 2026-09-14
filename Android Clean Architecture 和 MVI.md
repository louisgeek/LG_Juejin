
# Android Clean Architecture 和 MVI
- Feature-first 项目结构
- Repository Pattern 仓库模式，抽象数据层（ViewModel/UseCase 不直接调数据源）
- Result Pattern 结果模式：一收敛成 Domain 能理解的"成功/失败"两种状态，异常产生在哪层，就在哪层被 try/catch 并转换成 Result（通常在 Data 层的 Repository 实现中）
- UseCase Pattern：UseCase 可选，有业务逻辑再加（多步骤组合、校验逻辑）；Data 层没有多数据源时可以先不引入 DataSource 相关抽象
- Mapper Pattern：数据模型转换器

```
依赖层级 Presentation → Domain ← Data

DTO 属于远程协议；Entity 属于本地存储；User 属于业务领域。      

MVI                          Clean Architecture 项目架构
State / Intent / Effect      Presentation（UI） / Domain / Data
ViewModel                    UseCase
                             Repository 接口
                             Repository 实现
                             DTO / Mapper
                             API / Database

UDF 单向数据流（一种数据流约束），是一种原则 ，而 MVI 是实现 UDF 的一种较严格的风格


UDF / Google 风格： XxxAction + onAction()
严格 MVI 风格： XxxIntent + handleIntent()
其实 onAction/handleIntent 可以简单的看作做了统一和集中管理各个方法  dispatch
```

Model 是对各层对象模型的泛称（比如只管映射 JSON、只管映射数据库、只管描述 UI 展示状态），通常不作为常规类名后缀
类名采用反映边界与职责的具体后缀：

| 类型 | 命名 | 备用 | 说明 |
|---|---|---|---|
| **API 入参** | `XxxRequest` |  | API 请求体 |
| **API 出参** | `ApiResponse<XxxDto>` | `BaseResponse<XxxDto>` | 带 code/msg/data 等通用结构，如果是非通用的顶层响应类可用 `XxxResponse` |
| **网络数据模型** | `XxxDto` |  | Remote/Data 层网络传输对象，不关心业务规则,附带 `XxxDtoMapper` |
| **本地 DB 模型** | `XxxEntity` |  | Room/缓存持久化对象（本地缓存记录），附带 `XxxEntityMapper` |
| **Domain 领域模型** | `Xxx`（无后缀） |  | 代表业务核心，不加通用后缀 |
| **UI 状态** | `XxxUiState` |  | 页面或功能UI的完整可渲染状态（由 ViewModel 负责组装），局部用 `XxxState`、`XxxItemState` |
| **UI 事件** | `XxxIntent` | `XxxAction` | 意图，UI 到 ViewModel 的输入 |
| **UI 事件（一次性）** | `XxxEffect` | `XxxEvent` | 副作用，ViewModel 到 UI 的一次性指令 |
| **ViewModel** | `XxxViewModel` |  |  调 Domain 层的 UseCase 或 Repository 接口 |
| **UseCase** | `XxxUseCase` |  | 明确业务动作或业务编排（可选） |
| **Repository 接口（Domain 层）** | `XxxRepository` |  | Repository 出入参 Xxx（领域对象） |
| **Repository 实现（Data 层）** | `XxxRepositoryImpl` |  | Data 层实现 Domain 层接口（Data 依赖 Domain） |
| **远程数据源** | `XxxRemoteDataSource` |  | 封装 API 调用 |
| **本地数据源** | `XxxLocalDataSource` |  | 封装 Dao / DataStore / 文件等 |
| **DI 模块** | `XxxModule` |  |  |
| **页面** | `XxxScreen` | `XxxPage`+`XxxBody` | 可以考虑 Stateful Screen + Stateless Content 分开 |
| **网络模型 → Domain** | `XxxDto#toDomain` |  | 领域层后缀是什么就叫 to 什么，如果无后缀就叫 toDomain |
| **本地 DB 模型 → Domain** | `XxxEntity#toDomain` |  | 领域层后缀是什么就叫 to 什么，如果无后缀就叫 toDomain |
| **Domain → 网络模型** | `Xxx#toDto` |  |  |
| **Domain → 本地 DB 模型** | `Xxx#toEntity` |  |  |
| **Domain → UI状态** | `Xxx#toUiState` |  |  |



跨功能模块复用时，Feature 或 Domain 可以依赖对方公开的 Domain API（UseCase、Repository 接口等）；Data 的 RepositoryImpl 不应该去依赖对方 Feature 的 UseCase（操作应该上移到自身的 UseCase）

```
Feature A 想用 Feature B 的东西
        ↓
不要直接引用 Feature B
        ↓
把真正需要的能力提取到 B 的 API / Domain 模块
        ↓
Feature A 引用依赖这个 API / Domain 模块
```



用 Flutter 官方推荐的 MVVM+Repository 起步,项目变复杂后再按需引入 Domain 层,逐步向 Clean Architecture 靠拢


乐观状态(Optimistic state)—— 提升响应速度的用户体感
离线优先(Offline-first support)—— 基于 Repository 模式实现离线优先
Result 对象错误处理—— 用 Result 对象统一各层的错误处理



	Flutter 官方架构	     Clean Architecture(经典 Uncle Bob 版)
层	UI层、数据层,领域层（可选）	 Presentation / Domain（强推） / Data
UI 侧	View + ViewModel(MVVM)	    Presentation:View + Presenter/ViewModel
中间层	Use-case(可选,按需添加)	    Domain 层是核心:Entity + UseCase + Repository 接口
数据侧	Repository + Service	   Data 层:RepositoryImpl + DataSource(remote/local)




```
1. 用户在 XxxPage 操作 UI。

2. XxxPage 创建 XxxIntent，
   调用 XxxNotifier.dispatchIntent(intent)。

3. XxxNotifier 根据 Intent 类型，
   分发并执行对应的 handleXxxIntent 方法。

4. handleXxxIntent 按需调用 XxxUseCase，
   获取业务成功或失败结果。

5. XxxNotifier 使用 state.copyWith(...) 更新 XxxState，
   用于描述页面可持续渲染的状态。

6. XxxNotifier 按需发送 XxxEffect，
   用于描述仅执行一次的 UI 动作。

7. XxxPage 监听 XxxState：
   - ref.watch：根据 State 重建 UI。

8. XxxPage 监听 XxxEffect：
   - Stream.listen：执行 SnackBar、Dialog、导航等副作用。
```