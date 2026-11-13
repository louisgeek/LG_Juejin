# Flutter 架构理论参考

## 一、通用理论

| # | 来源 | 规定 | 依据 |
|---|------|------|------|
| 1 | Fowler catalog 中 PoEAA 的 Repository 条目；微软 .NET 微服务 DDD 指南（持久化层设计）；FFCA domain 页 | Repository 数据仓库模式：接口放 domain、实现放 data | Fowler 原文只讲集合语义与单向依赖（"Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects."）——业务代码只面对领域对象，不感知数据存储方式。「接口在 domain、实现在 data」Fowler 未明文规定：该安排出自微软指南（"it's recommended that you define and place the repository interfaces in the domain model layer..."）与 FFCA domain 页（"In the domain layer, these classes are `abstract interface class` definitions."），是 DDD/Clean/FFCA 的常见依赖安排，可保证 domain 不依赖具体实现、便于替换与测试。 |
| 2 | Evans《领域驱动设计》ch.6（Repository 模式）；微软 .NET 微服务 DDD 指南（持久化层设计） | Repository 围绕领域对象集合提供访问语义，不按表／API 机械划分 | Evans 定义 Repository 为领域对象集合的访问入口（本文引据 domainlanguage.com 的《DDD Reference 2015-03》第 17 节 Repositories，系作者官网对原书的摘录摘要、非 ch.6 逐字；"For each type of aggregate that needs global access, create a service that can provide the illusion of an in-memory collection of all objects of that aggregate's root type."）；微软指南反对每张数据库表都建一个、建议按聚合根组织；「每类数据一个 Repository」在 Android（"You should create a repository class for each different type of data you handle in your app."）与 Flutter（"There should be a repository class for each different type of data handled in your app."）有明文，微软指南未作此表述；「不按 API(endpoint) 机械划分」这一否定式仍属项目推论。项目通常只取「不按表／API 机械划分」，不将「每聚合根一个 Repository」立为项目规则。|
| 3 | Cockburn 六边形架构原文（2005 HaT 技术报告） | port／adapter 架构模式把内外分离、依赖反转落地：领域不依赖 adapter，port 宜少量 | 通过 port 定义应用与外部能力的边界、adapter 提供可替换实现，原文措辞为 application 对 adapter 无感知（"The application has a semantically sound interaction with the adapters on all sides of it, without actually knowing the nature of the things on the other side of the adapters."），原文无 `domain` 一词——外部设施（数据库、UI）可替换而不动核心逻辑。port 数量原文有整节 "How Many Ports?" 明确规定："Neither extreme appears optimal."（「每 use case 一个 port」与「全部并成一个」两个极端均非最优），且 "My selection tends to favor a small number, two, three or four ports"——倾向 2–4 个、未设强制数量，port 宜少量即据此。 |
| 4 | Seemann《Composition Root》（2011 博客） | 组合根装配模式：全部应用代码只依赖构造函数注入（或其他注入模式），绝不在代码内部就地组合；全应用只在入口一处创建并组装对象，DI 容器只被组合根引用 | 组合根是应用内组装全部模块的唯一位置，应贴近入口——分散在各模块自行组装会导致依赖关系失控、难以整体替换。其余模块不得触碰 DI 容器；原文措辞为 "A Composition Root is a (**preferably**) unique location in an application where modules are composed together" 与 "Only applications should have Composition Roots. Libraries and frameworks shouldn't."——即一个应用对应一个（最好唯一的）组合根，原文未对复杂应用另作数量断言。Flutter 项目可用 Riverpod 容器（ProviderScope + overrides）承担该角色——但注意容器引用范围比原文宽（Notifier 经 ref 取依赖），偏离「容器只被组合根引用」的原文口径。 |
| 5 | Martin《Screaming Architecture》（2011 博客） | 目录按业务／use case 命名，框架与 Web 属可替换细节（后来社区所称 feature-first 思路的重要思想来源） | 主张架构的第一要务是让读者从目录结构和包名上一眼看清领域与用例：顶层目录应体现业务用途而非框架或技术分层，看目录即知系统做什么。框架只是实现工具，Web、数据库、交付方式都属可推迟、可更换的外围细节，业务核心不随它们变，围绕 use case 组织才不绑死交付方式，且可脱离框架与数据库测试。注：原文反复以 use cases 为结构目标，但未给出 `use_cases/` 目录或 feature-first 目录布局，该目录化表述属文档判读。 |
| 6 | Martin 的 Clean Architecture（2012 博客） | 依赖规则：源码依赖只指向内层；层数可裁剪、规则不可裁剪 | 源码依赖只能指向内层，内层不知道外层存在——业务核心不依赖 UI、数据库等细节，才能独立测试与替换。四圈只是示意，层数与命名可裁剪，但依赖规则本身不可裁剪。 |
| 7 | Android 官方架构指南与社区实践 | 官方术语为 data source（两词；类名后缀才是 DataSource）；DTO 为社区常用术语；Repository 可持 0 至多个 data source | 持有数量出自 Android 官方指南；该指南还允许复杂场景下 Repository 依赖其他 Repository。DTO 可在数据层内部承载外部数据，经 Mapper 转为领域模型。而 Flutter 官方指南则明确 "Repositories should never be aware of each other."，多 Repository 的数据组合放在 ViewModel 或 domain 层。 |
- DDD：Domain-Driven Design 领域驱动设计，Evans 提出 Repository 是 DDD 标准构件之一
- FFCA：Feature-First Clean Architecture，VGV 提出的分包体系
- 聚合根：aggregate root，DDD 中聚合（aggregate，共享一致性边界的一组对象）的唯一入口
- 组合根：Composition Root，应用组装全部模块的唯一位置
- DataSource：data 层外部访问组件（Flutter 官方称 Service）
- DTO：Data Transfer Object，社区常用术语；是否需要独立 DTO 与 domain model 取决于边界和模型差异

## 二、Flutter / Dart 官方

| # | 来源 | 规定 | 依据 |
|---|------|------|------|
| 1 | Flutter 架构指南 | MVVM 分层规定：UI 层（View 和 ViewModel 1 对 1）+ Data 层（Repository + Service）；Domain 层条件引入；ViewModel 经 Repository 或 UseCase 取数据 | 官方分层口径：UI 层承担界面与交互逻辑，Data 层持有数据源与加工，View 与 ViewModel 严格一对一（View 是 widget 集合）；Domain 层是可选层（官方 Conditional 项为 "Use a domain layer."，guide 写 "This layer is optional"）——合并多个 Repository、逻辑极复杂或被多个 ViewModel 复用，满足其一即可引入（"meets one or more of the following conditions"），并非三条同时满足才引入。Repository 把原始数据加工成 domain model 供 ViewModel 消费，可直接访问也可经 UseCase。按 Flutter guide 的 UI-layer feature 定义，feature 是一对 View + ViewModel；该指南未讨论 FFCA 意义上的 headless feature，两者分类维度不同。Repository 之间不互调（官方原话 "Repositories should never be aware of each other."，比「不互调」更强）；跨 Repository 数据合并位置官方给 ViewModel 与 domain 层两个选项，跨 feature 编排场景选 domain 层是常见做法。 |
| 2 | Architecture recommendations 页 | 官方建议分三档：Strongly recommend（分层、Repository 模式、MVVM、抽象 Repository、逻辑不进 widget、单向数据流、不可变模型、DI、分层测试、写 Fake）、Recommend（用 freezed/built_value 代码生成不可变模型、Command、命名约定、go_router）、Conditional（ChangeNotifier、domain 层、API/domain model 分离） | 三档是官方置信度的表达——Strongly recommend 档为通用正确的做法（逻辑不进 widget 使 View 可简化测试、不可变模型防止数据在层外被改、DI 避免全局可变状态）；Conditional 档需按项目规模裁剪（API/domain 分离适用于大型 app、状态管理方案可自选故用 Riverpod 不冲突）；抽象 Repository 便于 dev/staging 切换实现。官方开篇即声明是建议非硬性规则。 |
| 3 | Architecture case study 系列（ui-layer、data-layer、dependency-injection、testing 四页） | 组件依赖规则：View 只依赖一个 ViewModel；ViewModel 可依赖一个或多个 Repository；Repository 可依赖多个 Service；Repository 与 Service 不反向依赖上层 | 官方 case study 明确 ViewModel 通过构造函数接收 Repository，Repository 通过构造函数接收 Service；它不是“只能依赖相邻层”的普遍规则。ViewModel 可直接组合多个 Repository，也可按需经 UseCase 编排；Repository 之间不互调（官方原话见 #2.1）。DI 官方推荐 `package:provider`（构造函数注入；ViewModel 单测把 Fake 经构造函数注入，widget 测试用 `Provider.value` / `ChangeNotifierProvider.value` 注入 Fake，`Provider(create:)` 在官方案例中用于创建真实依赖）；也有项目改用 Riverpod overrides 承担同等职责。 |
| 4 | Common architecture concepts | 核心概念规定：单一数据源；单向数据流（state: data→logic→UI；events 反向）；关注点分离；每类数据一个 Repository | Repository 是该类数据的可信来源与集中修改入口——多处可写会使状态来源难以判定；单向数据流让数据变化可追溯；Flutter 官方原句 "There is typically one repository class for each type of data in your app."（concepts 页）与 DDD 中避免按表机械划分方向相容，但不是严格等价规则；实际划分需结合业务边界与数据职责。 |
| 5 | Compass 案例（flutter/samples 架构官方案例） | 官方案例结构：目录结构、三个 main 入口（dev/staging/production，官方口径不完全一致）、dev/staging 双数据环境、高测试覆盖率 | 官方完整示例的参考价值：多环境入口 + 双数据环境（dev 用 assets 本地数据、staging 用 HTTP server）演示了环境装配的完整做法；示例用 Provider + ChangeNotifier 与 `package:provider` DI。入口文件官方实际命名为 `main.dart` / `main_development.dart` / `main_staging.dart`（README 运行命令 `--target lib/main_development.dart`）。注意官方口径自相矛盾：case study 称三个 main 文件对应 dev/staging/production，README 运行段只列 Development / Staging、首段却写 "development and production environments"，且 `main.dart` 默认转调 `development.main()`——以 README 运行命令为准。本文第七节的 `main_dev.dart` 属项目自定命名、非沿用官方拼写。落地示例：采用其 UI/Data 分层与多环境结构，状态管理换 Riverpod 3、DI 换 ProviderScope.overrides；ViewModel 不直接构造注入 data 层 Repository，取 concepts 口径由组合根装配。 |
| 6 | 官方设计模式系列（Result、Command、Optimistic state、Offline-first） | Result 以 sealed 类型表达成功／失败；Command 封装 ViewModel action 及执行状态；Optimistic state 先更新界面、失败时回滚；Offline-first 按业务需要组合本地数据、远端数据与同步 | Result 是异常处理的可选替代方式，令可预期失败成为可穷尽检查的值；不是 Dart 内置 API，也不要求所有层间调用都返回 Result。Command 是官方 Recommend，用于标准化 ViewModel 对用户操作的处理，并可携带 running/complete/error 状态（官方："If the `running` state is `true`, the command cannot begin executing again."）。Optimistic state 与 Offline-first 是可按场景采用的模式，并非唯一强制流程；将 Result 用作 Repository/UseCase 公共操作的失败值约定、将 Command 用于提交状态管理，均属项目取舍。 |
| 7 | Persistent storage architecture: Key-value data / SQL | 两个官方架构范例均为 Repository + Service 分层（Key-value：ThemeRepository + SharedPreferencesService；SQL：TodoRepository + DatabaseService） | 与「Repository 持 DataSource」职责划分一致——Repository 管领域语义、Service 管具体存储组件，仅数据访问组件命名不同（官方称 Service，Flutter 社区常称 DataSource）。SQL 范例采用 sqflite + sqflite_common_ffi（Web 另有实验性 sqflite_common_ffi_web），这说明该范例的实现选择，不构成 Flutter 官方指定的存储基线；cookbook persistence 两页只演示插件基本用法、无分层，不作为架构参考。 |
| 8 | Flavors 文档、--dart-define、testing/integration-tests 页、Flutter 1.17 / 3.19 release notes | 环境组织方式：原生 flavor 与 Dart 入口（`-t`）分别选择；`--dart-define` 编译期进入 Dart，官方亦文档化 Android 侧可经 Gradle `-Pdart-defines` 取得（本项目不依赖该通道） | `--flavor` 选择原生 flavor，不会自动选择 `main_*` Dart 入口（现行 flavors 英文页未出现 `--target`/`-t`，此句属工具行为而非官方逐字规定）；如 `flutter run --flavor dev -t lib/main_dev.dart`。`--dart-define` 在编译期进入 Dart 代码，官方亦已文档化原生侧通道：integration-tests 页给出 `-Pdart-defines="{base64 (key=value)}[, ...]"`（Gradle 属性、值 base64 编码）；3.19 release notes 写 "do not include entries from `--dart-define-from-file` files in the gradle config or environment during build"（反证普通 `--dart-define` 条目会进 gradle config/environment）；1.17 release notes 有 "Allow providing dart-defines to Android, iOS, macOS builds"；官方 CD 页甚至推荐用 `--dart-define` 传 CI 变量。但官方正文未出现大写 `DART_DEFINES`，也未提供 Gradle 侧 base64 解码范例——这部分属未文档化实现细节，本项目不依赖该通道、原生侧配置仍独立维护。官方 CD 文档有通过 `--dart-define` 传递 CI 变量的示例，但这不意味着该参数能保护秘密；官方 deployment/obfuscate 明确指出 "It is a poor security practice to store secrets in an app."，因此 `--dart-define` 只用于非秘密编译期配置，不作为秘密保管机制。现行 flavors 页另有 `appFlavor` 常量与 `default-flavor` 配置可读取当前 flavor 名。 |
| 9 | Flutter 测试文档 | 三类测试分工：unit / widget / integration 三类测试的定义、适用场景与官方 recipes | 三类各测一个层面——unit 测逻辑、widget 测组件、integration 测流程，是分层测试策略的官方依据。 |
| 10 | 国际化文档 | 本地化配置：flutter_localizations + intl；l10n.yaml 配 arb-dir 与输出位置（默认 lib/l10n）；Flutter 3.32+ 生成到源码目录，pubspec 必须写 `flutter: generate: true` | Flutter 3.32 起不再生成 `package:flutter_gen` synthetic package，使用生成的 l10n 源码时必须配置 `flutter: generate: true`；迁移指南建议将 `synthetic-package` 设为 false。现行国际化指南选项表仍写 `synthetic-package` 默认为 true，并描述其生成 synthetic package，与迁移指南对 3.32+ 行为的说明存在已核实的文档口径冲突。项目配置显式写 `synthetic-package: false`，将生成文件导入源码路径，并按目标 Flutter SDK 实测确认；不把冲突中的默认值当成可靠依据。 |
| 11 | 错误处理文档 | 错误捕获入口：FlutterError.onError 处理 Flutter framework 捕获的错误，PlatformDispatcher.instance.onError 处理未被 framework 捕获的相关异步错误 | 两个入口用于处理互补的错误通道，但不保证覆盖所有错误来源；手动 spawn 的 isolate 错误需经 isolate 自身错误监听（所引 Flutter 官方 testing/errors 页未涉及此点，属项目补充；Dart API 有 `Isolate.addErrorListener`），已被局部捕获或原生侧错误也须按各自机制处理。两个入口通常统一在 bootstrap 注册。 |
| 12 | go_router、go_router_builder | 声明式路由（go_router）：redirect 支撑登录态守卫，ShellRoute 支持嵌套 Navigator，StatefulShellRoute 支撑保留各分支状态的多栈导航（如底部导航）；类型安全路由由 go_router_builder 生成（`@TypedGoRoute` 与 `GoRouteData`） | 声明式 redirect 让守卫逻辑集中可测而非散落命令式跳转中；类型安全路由经 build_runner 生成、不是字符串路由的默认能力——字符串路径拼接不具类型安全。go_router 为 feature-complete 状态（Flutter 团队当前重点是 bug 修复与稳定性，未计划主动开发新功能，但接受社区贡献）；SDK 约束以 pub.dev 当日数据和 CI 解析为准。 |
| 13 | Dart 3 sealed class 与模式匹配（现行 class-modifiers 与 branches 页） | sealed 穷尽检查：sealed 类型层级 + switch 穷尽检查，漏分支编译期报错 | 典型应用如 Result、SessionState、AppError 等状态类型——穷尽检查把「漏处理一个分支」从运行期问题变成编译期问题，避免 if-else 链漏处理。 |
| 14 | Effective Dart、flutter_lints | lint 基线：官方风格指南与推荐 lint 集（`flutter_lints`，`flutter create` 新项目默认启用），作为 analysis_options.yaml 基础，团队规则在其上增量 | 不另起炉灶——官方集是团队共识基线，增量规则只加团队特有约束，避免与官方规则冲突。 |
| 15 | Pub workspaces | Pub workspaces 配置：根 pubspec 的 workspace 列表 + 成员包 resolution: workspace：全仓统一依赖解析、单一 pubspec.lock、单一共享 `.dart_tool/package_config.json` 并降低分析内存占用 | 特性自 Dart 3.6.0 引入（成员包须声明 `^3.6.0` 或更高，glob 通配写法需 Dart 3.11+）。单一共享依赖解析增加冲突风险，但官方定性为 useful feature——"It forces you to resolve incompatibilities between your packages when they arise, rather than when you start using the packages."，引用时不宜只取前半句。单包分层项目不需要；分包会放大该风险。 |
| 16 | Native interop（Pigeon（flutter.dev 发布）、FFIgen（tools.dart.dev 发布）、JNIgen） | 官方互操作工具：Pigeon 默认生成类型安全的平台通道代码；Pigeon 另提供 direct native interop 选项（iOS/macOS 用 FFIgen、Android 用 JNIgen），目前为 preview；FFIgen 可独立生成 Dart FFI 绑定，支持 C 与 Objective-C；Swift 可经 Objective-C 兼容 API 暴露后互操作；自动化纯 Swift 绑定工具 Swiftgen 仍属实验性；JNIgen 为 Java/Kotlin JNI 互操作提供绑定生成 | Preview 明确指 Pigeon 的 direct native interop 选项，不泛指 FFIgen/JNIgen 所有独立 direct-binding 用法。FFIgen 解析 C 与 Objective-C headers（包括带 `@objc` 的 Swift API）；纯 Swift 特性不受支持，`package:swiftgen` 仍为实验性。Flutter 现行 Native interop 文档将 JNIgen 列作 direct-binding 工具；Dart Java interop 页对相关能力仍有 experimental 标注，采用前需按目标 SDK 与工具版本核验。互操作属实现细节层组件，放 shared 或 data 层的 DataSource，不入 domain。 |

- MVVM：View + ViewModel 的 UI 分层模式，Flutter 官方架构指南采用
- SSOT：Single Source of Truth，单一数据源，数据的唯一可信与可修改处

## 三、Riverpod 3

| # | 来源 | 规定 | 依据 |
|---|------|------|------|
| 1 | 官方文档（Notifier、AsyncNotifier、StreamNotifier） | Notifier 写法约定：三种 Notifier 分别对应同步、异步、流式状态；ViewModel 以 Notifier 实现：build() 产出初始状态、公开方法承载用户操作、UI 用 ref.watch 读状态 ref.read 调方法 | build 的约定决定写法：依赖在 build 内用 ref.watch 声明——依赖变化时 provider 被 recomputed、旧状态销毁并新建（官方 "The provider is recomputed, such as with `ref.watch`. In that case, the previous state is disposed, and a new state is created."）；初始状态通常由 build 返回值表达，官方措辞是 "DON'T perform side effects during the initialization of a provider"（do_dont 页）。异步状态类型可在 build 中设置 `AsyncLoading(progress:)` 报告进度（whats_new 页 "An optional `progress` property is available on `AsyncLoading`."）；「build 可多次执行」为据 rebuild/recomputed 与自动重试推出的结论，官方无该逐字句。 |
| 2 | What's new in 3.0、DO/DON'T、迁移指南 | 3.0 升级入口：特性总览、用法约定与 2.x→3.0 breaking changes 清单 | 官方警示本版含生命周期变更、可能以隐蔽方式破坏应用，升级须谨慎；下方「3.0 写法约定」即从这些页收拢，升级执行以迁移页当日清单为准。 |
| 3 | Overrides 与测试文档 | overrides 装配机制：ProviderScope.overrides 在 bootstrap 注入实例或工厂（overrideWithValue 注预创建实例，overrideWith 注创建工厂）；测试用 ProviderContainer.test | overrides 是组合根装配的机制（见 #1.4）；官方不建议 mock Notifier、优先 mock Repository——官方逐字理由是 "Notifiers cannot be instantiated on their own, and only work when used as part of a Provider."；「掩盖真实状态机逻辑」为本项目推论，非官方原文。 |
| 4 | Automatic retry | 自动重试行为：计算抛异常默认重试，200ms 起翻倍、封顶 6.4s、最多 10 次重试（不含首次计算）；Error 与 ProviderException 不重试；可全局或单 provider 定制（`retry:` 参数在三处均可配：provider 构造函数、ProviderScope/ProviderContainer、`@Riverpod` 注解） | build() 内 catch 后返回正常状态不触发重试，此为由官方条件 "A retry is attempted when an exception is thrown during the provider's computation." 推出的结论（非逐字原句；FAQ 亦建议 "make the computation total: return `null`... instead of throwing"）；另需注意 `migration/from_state_notifier` 页反而建议 "Removing any `catch`/`try` blocks in initialization or in side effects methods"，取向不同，按项目取舍——故只对可恢复的临时错误配置有限重试；await provider.future 会等重试结束才抛错。默认十次重试的纯退避等待合计约 38.2 秒，不含每次实际计算或网络请求耗时；splash 出现该量级延迟需同时满足：build() 抛可预期 domain error、异常非 Error/ProviderException、用 await future 等待。可预期业务失败可转换为 Result/状态值以避免重试；若选择在 build() 中抛出，则须通过 provider 或全局 retry 配置跳过永久性业务错误——官方并未一律禁止在 build() 中抛出业务失败。 |
| 5 | Mutations、Offline persistence（均实验性） | Mutations：Mutation 对象跟踪副作用的 `MutationIdle`/`MutationPending`/`MutationError`/`MutationSuccess` 状态；Offline persistence：Storage 接口 + persist() 持久化 provider 状态、默认缓存 2 天 | Mutations 解决副作用进行中 provider 被销毁的问题、按钮可依状态禁用；Offline 的 key 须全局唯一、跨重启稳定、含 family 参数（重复 key 会触发断言；具体持久化行为见官方说明）；官方提供 riverpod_sqflite 作为 SQLite Storage adapter，不据此推断跨进程/isolate 安全，测试可用 Storage.inMemory。Offline persistence 是数据库交互的简单封装，不替代复杂迁移或手写数据库操作；两项 API 均属实验性，隔离在 feature 内、不进 core。 |
| 6 | riverpod_generator | 代码生成写法：@riverpod 注解 + build_runner 生成 provider 与 family；手写与注解两种写法项目统一选一种 | 官方口径为完全可选、仅在已为其他用途用代码生成时才用（在用 freezed/json_serializable 即满足）；同页亦写明 Riverpod 总体拥抱并推荐代码生成，两句话适用范围不同——不宜只取前半句。autoDispose 默认值随写法而变（注解默认启用、keepAlive: true 退出；手写 3.0 用 isAutoDispose: true，修饰符仍有效、移除的只是接口类），统一以现行页写法为准；“(riverpod_generator only)” 的 lint 仅对注解写法生效。 |
| 7 | riverpod_lint | Riverpod lint 工具：analysis_server_plugin 系插件，analysis_options.yaml 顶层 `plugins:` 安装，`dart analyze` 可运行并提供诊断与 quick fix | 工具定位属稳定口径；版本号、analyzer/SDK 区间、与 import_lint 的相容冲突及复核触发条件属版本快照，统一登记在 #6.7「依赖方向校验」决策行（2026-11-08 核验口径），此处不重复维护。 |
| 8 | Scoping providers | scoping 局部覆盖：在应用局部（页面/子树）改变 provider 行为，方式是在非根 ProviderScope 上用 overrides 覆盖；provider 须显式声明 dependencies 才可被 scope | 官方页首警告该特性高度复杂、未来可能重做；provider 必须显式声明 `dependencies` 才可被 scope，依赖 scoped provider 的其他 provider 也须在自己的 `dependencies` 中列出该 scoped provider。显式 opt-in 与依赖声明是官方设置的使用成本——这两点正是多数项目不用它的论据。 |
| 9 | ProviderObserver | 观察者挂 ProviderScope.observers，监听 provider 生命周期：四个主要回调——didAddProvider（初始化）、didUpdateProvider（更新）、didDisposeProvider（销毁）、providerDidFail（发出错误；API 另有容器创建／销毁、provider unmount 与 mutation 生命周期等回调，共 11 个 Observer 回调） | 是 3.0 挂日志/监控/崩溃上报的正规入口——回调覆盖容器内全部 provider、不侵入业务代码；providerDidFail 承接 error 级转发，转发前须跳过 `ProviderException`（依赖链上游失败会让下游 provider 重复触发，同一根因只上报一次，见 #5.24）。3.0 起用单个 `ProviderObserverContext` 取代原 `ProviderBase` + `ProviderContainer` 两个参数（context 内含 container、provider 及 mutation 等附加信息）；各回调仍保留 value / previousValue,newValue / error,stackTrace，如 `providerDidFail(ProviderObserverContext context, Object error, StackTrace stackTrace)`——只有 `didDisposeProvider(context)` 是单参数，实现 #5.24 的 observer 时按新签名编写。另：直接失败的 provider 的 observer 收到的是未包装的原始 error，跳过 `ProviderException` 只针对依赖链上游失败。Repository provider 的生命周期规则见 #5.5、#5.9。 |
| 10 | Pause/Resume | 所有 ref.listen 的订阅可手动暂停/恢复；widget 子树不可见时相关监听自动暂停（基于 TickerMode）、provider rebuild 期间订阅暂停、provider 暂停时其订阅一并暂停 | 与 autoDispose 是两个机制（官方分别描述两种行为、未用 "two mechanisms" 逐字表述，此归纳属推论）：autoDispose 决定销毁，pause 决定监听是否暂时接收更新——不可见 Consumer 的相关监听会暂停；provider 是否暂停取决于其监听关系，其他未暂停的监听者仍可能继续使用它。autoDispose 不替代 pause。 |

3.0 写法约定：
- 用 Notifier、AsyncNotifier、StreamNotifier。
- StateProvider、StateNotifierProvider、ChangeNotifierProvider 属于 legacy（`package:riverpod/legacy.dart`），新代码不用。
- AsyncValue 是 sealed；error/loading 状态下 value 返回 previous value，无 previous value 时为 null（3.0 的 `value` 即 2.x 的 `valueOrNull`，旧 `value` 已移除，勿与 2.x 语义混淆）。
- 依赖失败会被包成 ProviderException：默认重试不重试该异常，由真正抛错的源头 provider 承担重试（见 #3.4）。
- 销毁后调用 Ref / Notifier 的方法（除 mounted 外）抛 UnmountedRefException；稳妥写法是不依赖该类型名做捕获——异步代码 await 之后先用 ref.mounted 判活，再碰 ref/state。

## 四、社区与公司实践

| # | 来源 | 规定 | 依据 |
|---|------|------|------|
| 1 | Bizzotto 的 Riverpod 架构文章（2023-09 更新） | 四层结构（Bizzotto 口径）：data / domain / application / presentation 四层 | 其 application 层是 feature 内部、**可选**（官方 "we can introduce a new, **optional** layer called the application layer"）的 Service 中介层（"service classes, which act as the middle-man between the controllers... and the repositories"，协调多个 repository、承载跨数据逻辑），不等于 app/ 组合根（DI 装配入口）——两词同形不同义，引用时须辨析。 |
| 2 | Bizzotto 项目结构篇（2022-03-23） | feature 划分方式：feature 按「用户做什么」划分（auth、cart、checkout、orders）而非按技术类型；单包内分目录、最终结构树无顶层 shared/（非禁令） | 按用户做什么划分与 feature-first 同源（业务可读性优先）；其 common_widgets、constants 等是 lib/src 下与 features/ 并列的目录（勿与 FFCA 的 shared 段混同，两套体系）；test/ 目录镜像 lib/ 结构（原文 "it makes a lot of sense for the `test` folder to follow the same project structure as the `lib` folder"）。属单包内分目录，不是物理分包。注：最终结构树中无顶层 `shared/`，但原文并未禁止，反而写 "there is no right or wrong answer, and you have to use your best judgement on a case-by-case basis"，并提醒 shared/common/utils 易成 dumping ground。 |
| 3 | VGV FFCA 标准站（overview / domain / data / presentation / project_structure 子页）与 Scooter 生产复盘——同一标准的先后两版 | FFCA 分包体系：apps / features / shared 三段；feature 按需采用 domain、data、presentation 包；支持完整、headless 与 presentation-only feature | package 边界使依赖关系显式化：未声明的直接依赖可由 analyzer 诊断，规则仍需 lint/CI 执行；domain 纯 Dart；data 实现 domain 接口并处理 DTO 映射；跨包只经公共 barrel API。详细规则、演进与规模成本见表后 #4.3 要点。 |
| 4 | Marcos Sevilla 分层架构指南（2022-04-12，VGV 官方博客署名文章、非个人博客）与 VGV 官方站 Layered Architecture 页 | 两套层清单并存（VGV 口径）：博客口径 data / domain / business logic / presentation（business logic 层用 flutter_bloc）；官方站口径 data / repository / business logic / presentation（无 domain 层、单列 repository 层，data/repository 层以独立 package 放 packages/ 目录） | 两套清单的层名不能互换，且与 FFCA 也是不同体系——概念和目录不混用。官方站定位 Layered 为多数项目的正确起点（多 app 共享 feature 时改用 FFCA）；博客指出小项目用 packages 可能过重。落地示例：借用 “Layered Architecture” 仅取「不物理分包」含义、分层与目录仍按 FFCA 结构退化到单包执行。`very_good_analysis` 是 VGV 单独发布的开源 lint 包（"an open source package that contains lint rules for Dart and Flutter used internally at Very Good Ventures"）；「可选使用、非该架构组成要求」为项目判读，官方无该逐字表述。 |
| 5 | freezed、json_serializable | 不可变模型与序列化代码生成：Freezed 可生成不可变模型（copyWith、==），按 union 定义生成 sealed 联合；json_serializable 可生成 JSON 序列化代码；仅在 API/domain model 确有分离需求时，DTO 留在 data 层内部并经 mapper 转为 domain model | Flutter 官方将 freezed 列为不可变模型代码生成的 Recommend 选项，并提示模型量大时代码生成会增加构建耗时；json_serializable 是项目选择的序列化工具，不代表 Flutter 官方指定。小模型量下构建开销是否可接受由项目实测决定。「多包项目里每个使用方都要放自己的 build_runner 依赖」是多包项目的推论、非 freezed 文档原文，落地规则见 #5.21「依赖声明」（代码生成依赖放在实际使用它的包），生成物入库与 CI 漂移校验见 #5.20「生成物与 CI」。 |

#4.3 补充要点：
- data 包可带 backend 后缀（`{feature}_data_{backend}`，如 `auth_data_firebase`）；domain 放模型、业务规则与 repository 接口，data 实现接口并管 DTO 映射，DTO 不出 data 包；presentation 不依赖 data，shared 不依赖 feature。
- shared 判据「能否发 pub.dev 给陌生人原样用」是经验法则；VGV 将 Layered Architecture 的适用情形作为选型建议，条件不要求同时成立。
- FFCA 建议避免 “use case” 术语、类改用 Command/Query（目录名 `use_cases/` 保留）；从单包切换到 FFCA 分包时，相关类需同步改名，这是迁移成本之一。
- 3 / 10 feature 在首发文（"That works well when there are three features. At ten or more..."）中是举例、非阈值；但 FFCA 标准站 overview 页另有 "fewer than three features" 作为选型条件，两者口径不同，引用时勿混（见 #6.1）。
- 演进时间线：2026-08-10 首发文 → 2026-09-17 Scooter 生产复盘（16 features 实证，变更含 {Feature}Handlers、一 domain 多 data 包含 backend 后缀、跨 feature 读写归 Queries/Commands、shared-feature 边界规则成文——其中具体变更项未逐条与英文原文核对，仅时间线、16/48/88 与组合根表述已逐字核实）→ 文档记载变更已并入现行标准站（复盘对 presentation-only 一项有逐字依据："That variant was not in the August post. It was added to the standard after this project showed the need for it."）。复盘规模记载为 16 features、48 个 feature packages、总计 88 包、每 feature 约 8–9 个样板文件，组合根成为需独立测试的真实代码；这些数字可由 [Scooter 生产复盘英文原文](https://verygood.ventures/blog/feature-first-clean-architecture-in-production/) 直接核实，但仅属单一项目的规模实证，不得当作 FFCA 的普遍要求或分包阈值。
- 细粒度校验口径已扩展（属项目内部论断，本次未单独以英文官方资料核实）：首发文 analyzer 钩子可选说明与标准站依赖表脚本校验是两个命题；落地示例沿用分包与依赖规则、状态管理换 Riverpod 3。

- barrel：package 对外公开的聚合入口文件，跨包只经它访问
- core 段（apps / features / shared / core 中的第四段）：非 FFCA 原生——FFCA 标准结构只有 apps / features / shared 三段；core 段取自 Android 官方模块化指南（common modules 又称 core modules）与 Now in Android（`core/*` 承载 common、network、designsystem 等公共模块），业内对 core/shared 边界无统一定义，本项目在 FFCA 框架上增设 core 段（承载与业务语义无关的技术能力，非纯 Dart 契约）属本项目取舍，职责划见 #5.16、#5.17。

## 五、项目最佳实践

| # | 主题 | 推荐做法 | 理由与代价 |
|---|------|----------|-----------|
| 1 | 目录形态 | 默认单包、不物理分包、内按 feature 分目录；经评审确认多 app 复用、多团队按 feature 所有权并行或大量 coding agent 并行中的至少一项，且收益足以抵消成本时，才启用 FFCA 式分包 | 分包成本真实——Scooter 88 包的规模实证（每 feature 约 8–9 个样板文件、组合根成为需独立测试的真实代码）；VGV 所列 FFCA 适用情形是选型建议，不是硬门槛，feature 数量（3／10）只是评审提示线、不是分包开关（3 与 10 出自 FFCA 首发文示例，见 #4.3）；现行 overview 另提到采用 AI 工具时小项目也可考虑 FFCA |
| 2 | feature 分包结构 | 按需采用 `{feature}_domain` / `_data` / `_presentation` 包（data 可带 backend 后缀，如 auth_data_firebase），跨包只经公共 barrel；支持 headless 和 presentation-only feature | package 边界使依赖显式化；未声明的直接依赖可由 analyzer 诊断，边界规则仍需 lint/CI 执行。backend 后缀使替换后端时无需改变上游领域接口与分层结构，但仍需调整依赖声明和组合根装配；多后端并存时另设 fake 包供测试替换 |
| 3 | 模块依赖 | app → features → shared → core 为主依赖方向、不得成环；feature 间默认只经公共 domain contract 协作，不得依赖对方 data，复用 UI 仅依赖对方公开的 presentation API。有明确业务 owner 的跨 feature 编排可由 owner 的 domain 定向依赖其他 feature 的 domain contract（见 #5.8），不得反向依赖或成环；共享且无业务语义的契约可上提 core，有业务语义的协作由 owner feature 或 app 层编排 | 反向依赖会成环、破坏可替换性；跨 feature 走 domain contract 可保持各 feature 独立演进 |
| 4 | feature 内依赖 | domain 位于底层，只依赖 core 的纯 Dart 契约（Result、无业务语义的错误基类等），不依赖 core 的技术实现模块（network/storage/logging/git 等）、data 与 presentation；默认不依赖 shared。仅当有明确业务 owner 的跨 feature 编排确需其他 feature 的公开 domain contract 时，允许该 domain 定向依赖对方 domain；不得形成循环依赖。data 与 presentation 都依赖本 feature domain；presentation 不依赖任何 data 包 | 官方 "Guide to app architecture" / "Common architecture concepts" 建议 UI 与 data 分离；Compass 的具体 MVVM 案例则把 Repository 注入 ViewModel。两者展示的边界不完全一致，本项目选择更严格地让 presentation 不直接依赖 data，由组合根装配；domain 以 core 的纯 Dart 契约为基础依赖，并仅在有明确业务 owner 的跨 feature 编排时定向依赖其他 feature 的公开 domain contract；这是本项目比 FFCA 更严格的边界安排（FFCA 允许 domain 依赖 shared，本项目把可复用纯 Dart 契约收敛到 core 的契约部分，避免 domain 被 Flutter 依赖污染）——core 扩展为技术能力桶后，「依赖 core」不再自动保证 domain 纯 Dart，须限定只取其纯 Dart 契约 |
| 5 | 组合根装配 | app/bootstrap 负责 ProviderScope、环境 overrides 与实现装配；Repository provider 以 domain 抽象类型声明在消费它的 presentation，app 用 data 实现 override | 官方 DI 推荐 `package:provider`（见 #2.3），本项目改用 Riverpod overrides；代价是团队需理解容器作用域与 override 装配。domain 不因此引入 Riverpod，presentation 也不为装配而 import data；Repository/DataSource 优先构造函数注入，Notifier 经 provider/ref 取依赖、保持私有；同一 Repository 被多个 feature 消费时，app 须以同一实例 override 各消费方（overrideWithValue，或 app 层单一 provider），不得分别创建，避免多实例破坏 SSOT（#5.9）。消费方声明的 provider 不提供默认实现；未 override 时在该 provider 首次被读取并求值时显式失败，而非静默产生第二个实例。若要求启动阶段发现漏装配，bootstrap 必须主动读取并校验所有必需 provider；未执行该校验时不得称为启动期失败。本项目统一采用注解写法，故 Repository provider 的默认生命周期**是 autoDispose**（手写 provider 默认非 autoDispose，见 #3.6）：Repository 通常只持 `final` 依赖、不保存可变数据，重建即重新绑定引用；缓存、串行锁这类状态由它依赖的组件持有，因此应在那些依赖的 provider 上设 keepAlive，而不是在 Repository 上。仅当 Repository 自身引入内存缓存字段时，才改为 `@Riverpod(keepAlive: true)`（手写 provider 对应写法为不设 `isAutoDispose: true`，两种写法不得混用，见 #3.6）。这不保证 provider 重算或容器重建后仍是同一实例。 |
| 6 | Repository 抽象 | presentation 只依赖 domain 接口，由 app 注入 dev/staging/Fake 等实现 | 官方 Strongly recommend 抽象 Repository 类（"Use abstract repository classes"，便于 dev/staging 切换实现）；同档还有 Repository 模式、SSOT、DI 等，不宜写「只要求抽象」。官方未明文规定接口的文件位置（Compass 案例放在 `data/repositories/`）——「接口在 domain、实现在 data」是 DDD/Clean/FFCA 常见安排；mocktail 对抽象与具体类均可 mock，不构成决策依据 |
| 7 | UseCase | 默认不建，需要时放 domain；仅在逻辑复杂、被多个 ViewModel 复用或需组合多个 Repository 时引入 | 官方 Conditional 项为 "Use a domain layer."，use-case 只在说明中出现（"In very large apps, use-cases are useful, but in most apps they add unnecessary overhead."）；单个 ViewModel 依赖超过 3 个 Repository 只作复杂度评审信号；Repository 之间不互相调用（官方原话 "Repositories should never be aware of each other."，见 #2.1）（FFCA 分包形态下类名用 Command/Query、避免 "use case" 术语，见 #4.3） |
| 8 | 跨 feature 编排 | 有明确业务所有者的放该 owner feature 的 domain，只依赖其他 feature 的公共 domain contract；app 专属且无单一 owner 的编排逻辑放 `app/application/use_cases` | 避免 domain 循环依赖；组合根只负责装配、不含业务规则；该 app 路径是本项目存放约定，不是 VGV 的规定 |
| 9 | 状态所有权 | Repository 是可共享应用／领域数据的 SSOT 与唯一修改入口；Notifier/ViewModel 只管页面、表单、选择与提交等 UI 状态 | 多处可写会使状态来源不可判定；共享数据不归某个页面私有，派生数据用 provider/select 计算而非重复保存；Repository 持有共享可变数据时，其 provider 正是 #5.5「自身持有可变状态才 keepAlive」的例外场景，生命周期细则见 #5.5 |
| 10 | Provider 生命周期 | 页面级、可重建的 provider 按 autoDispose 配置；跨页面共享状态由长生命周期 Repository/provider 持有 | 不用 keepAlive 掩盖错误的状态所有权；autoDispose 只决定销毁，监听是否暂停按 Riverpod 3 的 Pause/Resume 规则处理（见 #3.10）；keepAlive 的判定看**该 provider 自身是否持有可变状态**，不按「是不是 Repository」一刀切，细则见 #5.5。 |
| 11 | 模型与 DTO | domain model 与 ViewModel state 用不可变对象（优先 freezed）；当 API 数据结构与业务模型有差异时，DTO 与 mapper 留在 data 内部，Repository 对外暴露 domain model；简单同构数据可不另建 DTO | 仅在确有边界隔离收益时增加映射，避免为同构简单模型制造无收益转换；不可变模型降低跨层意外修改；若采用独立 domain model，不附加基础设施序列化注解 |
| 12 | 单向数据流 | UI event → Notifier 方法 → Repository/UseCase → 新 state → UI；build 中 ref.watch 订阅、ref.listen 处理一次性副作用、事件回调用 ref.read | Widget 不直接改数据或调 Repository/DataSource；await 后先用 ref.mounted 判活再碰 ref/state |
| 13 | 异步与提交状态 | 读取：优先 `AsyncNotifier` + `AsyncValue`；提交：同步 Notifier + 不可变 `SubmitState<T>`（idle/running/success/error），类型放 `core/submit_state.dart`；`LoadState<T>`（loading/data/error）仅用于既需同步状态、又不需 AsyncValue 语义的读取场景，不作全局默认 | 本项目默认将可预期业务失败转换为 Result/状态值，避免 provider build() 触发默认重试；若选择在 build() 中抛出，则须通过 provider 或全局 retry 配置跳过永久性业务错误（见 #3.4）。读取走 AsyncNotifier 才能用上 AsyncValue 的 sealed 穷尽与 previous value 语义（见 #3 写法约定）；提交态与读取态分开建模，避免一个泛型同时承担两种语义。`LoadState` 仅用于特殊场景，非全局默认，提交态请使用 `SubmitState<T>`。 |
| 14 | Result 边界 | Repository/UseCase 的公共查询或命令存在调用方需处理的可预期失败时，统一返回 `Result<T>` 并 switch 穷尽处理 | 业内无「所有层间调用必须返回 Result」的统一标准；无失败语义的方法、纯领域计算不机械包 Result；Result 不承诺方法绝不抛异常 |
| 15 | 异常映射 | DataSource 可抛 HTTP／数据库／平台异常；Repository 在 data 边界转成 domain error，可预期失败走 `Result.error` | 不让 DioException、数据库异常等基础设施类型穿透到 domain/presentation；`Error` 与程序员缺陷不包装成业务 Result，由全局错误处理上报 |
| 16 | core 与 shared | core 是与业务语义无关的技术能力：网络封装（Dio client、拦截器、统一错误处理）、数据库/本地存储封装、异常/Failure 类型、路由路径常量与路由基础设施（不含具体页面）、常量与环境配置、Result 等通用类型。shared 是不属于任何单一 feature、但带业务表达的复用资产：通用 UI 组件、跨 feature 复用的业务规则（格式化、通用校验）、l10n 文案管线。 | 判据来源须分级标注——三条判据（不得依赖 feature／是否带业务语义／多消费方门槛）的完整依据、来源分级与 NIA/FFCA 口径差异，见表后 #5.16 补充要点；判定顺序与口诀见 #5.17。core/shared 是目录（单包）或独立 package（分包），职责与依赖规则相同 |
| 17 | core/shared 判定顺序 | ①有明确业务 owner 的留该 feature，app 专属留 app；②与业务语义无关的技术能力与纯契约放 core（network/storage/logging/config/error/navigation/Result）；③不属于单一 feature、但带业务表达的复用资产放 shared（通用 UI 组件、跨 feature 业务规则、l10n）；④有替换需求才拆 port/实现 | ①–④为本项目为统一落地而定、非 Android 或 FFCA 原文；口诀「core 管地基与词汇，shared 管可复用的业务表达」；判定依据见 #5.16 三条（不得依赖 feature / 是否带业务语义 / 多消费方门槛）；无法证明有两个消费方时先留在 feature，防垃圾场 |
| 18 | 导航与路由 | app 拥有 go_router 全局路由表、登录守卫与 go_router_builder 类型安全路由；feature 只表达导航意图，不 import app router | 由 app 通过回调/adapter 注入导航能力，避免路由造成 feature 反向依赖或循环依赖 |
| 19 | 测试归属 | domain 测业务规则与 UseCase；data 用 Fake/Mock DataSource 测 Repository 与映射；presentation 用 Repository Fake + ProviderContainer.test 测 Notifier、widget test 测 View；app 测装配、导航与集成流程 | Flutter 官方建议分层测试并测试组件组合；各 package 优先测自己的公共行为，避免重复覆盖其他层的内部实现 |
| 20 | 生成物与 CI | 本项目选择将需要随源码发布的生成物纳入版本控制、不手改、随源码同提交；生成器升级单独提交一次全量重生成；CI 在干净检出并解析依赖后，按仓库配置运行所有适用的生成器，再运行 analyze/test，并检查生成后工作区无未提交改动（`git status --porcelain` 为空） | 先重生成再检查工作区，才能验证生成物与源码一致；具体生成命令随项目选用的生成器配置确定，未启用代码生成时跳过该步骤；是否入库须结合生成器的 build_to 配置、package 发布方式与消费者是否运行生成器决定，并非 Dart/Flutter 对所有生成物的统一要求；代价是构建时间增加、生成器版本需固定，并承担合并冲突与噪声 |
| 21 | 依赖声明 | 每个 package 直接声明自己 import 的依赖，代码生成依赖放在实际使用它的包；CI 从 workspace 根运行（或逐包运行）dependency_validator 检查 missing/unused | 不依赖 app 或传递依赖「顺便带入」的包，避免隐式耦合与升级意外；该工具检查依赖声明，不负责 import 方向或包边界校验 |
| 22 | 依赖方向校验现状 | 单包形态无 package 边界，依赖方向只由 review 保障；dependency_validator 仅校验依赖声明（missing/unused），不校验 import 方向（见 #5.21）；分包形态使依赖关系显式化，未声明的直接依赖可由 analyzer 诊断，但完整边界规则仍需 lint/CI 校验；import_lint 待与 riverpod_lint analyzer 区间相容后启用 | 细粒度路径级约束（feature 不 import 他 feature 的 data、barrel 不 import lib/src）暂未接入自动校验；两 lint 区间当前不相交、同时启用会在 pub get 失败 |
| 23 | 版本与 SDK | 文档不指定包版本、不维护版本清单；具体项目须在 `pubspec.yaml` 的 `environment` 与 CI Flutter/Dart 工具链中明确并保持一致的 SDK 基线；依赖解析用全新 `pub get` + analyze/test 验证，依赖下限用 `pub downgrade` 验证 | 版本号只作为论证相容性的事实证据、随 pub.dev 数据漂移；本文是架构参考而非具体项目配置，SDK 基线须由项目仓库声明，不得从本文暗示的特性版本推定 |
| 24 | core/logging | AppLogger 接口与 Riverpod 侧实现（ProviderObserver）都放 `core/logging/`；ProviderObserver 挂 ProviderScope.observers（providerDidFail 转发 error 级，但先跳过 `ProviderException`——依赖链上游失败会让下游 provider 重复触发，同一根因只上报一次），Flutter 侧在 bootstrap 注册 FlutterError.onError / PlatformDispatcher.instance.onError | 脱敏 + error 级转发崩溃平台属社区实践口径；三处共同构成错误上报链路 |
| 25 | 磁盘缓存 | 缓存组件封装在 core（`core/storage/`，Sembast 核心及平台适配；Flutter 原生文件路径由 path_provider 提供，Web 使用 sembast_web/IndexedDB），由 data 层 DataSource 调用；缓存读写不进 domain，Repository 只暴露结果 | Sembast 核心无原生平台插件有助于降低平台集成差异、简化部分测试，但不保证全平台行为完全一致，目标平台仍需验证；单文件文档型契合「按 key 缓存 JSON」；明确 store 与 key 的命名范围，确保缓存记录不会意外覆盖；Riverpod 离线持久化另有其 key 全局唯一、跨重启稳定且包含 family 参数的要求（见 #3.5），两种机制不可混为同一规则 |

#5.16 补充要点：
- 三条判据为本项目为统一落地而定，**不是任何外部权威的规定**（§4.3 已声明「业内对 core/shared 边界无统一定义」）。
- ①**不得依赖 feature**（有外部依据）——Android 官方 common modules 页只定义「contain code other modules frequently use; reduce redundancy; don't represent any specific layer in an app's architecture」，**未规定反向依赖**；「core 不得依赖 feature/app」出自 NIA 示例项目的工程约定（`docs/ModularizationLearningJourney.md` 原文 “they shouldn’t depend on feature nor app modules”（原文用弯引号，且同句允许 core → core）；`core/*` README 中无 "Core Module Principles" 一节（`core/README.md` 返回 404，各子模块 README 仅含标题与依赖图），不引作依据），非架构指南原文。本项目采纳该约定，并据 Android 官方的 Low Coupling 原则推及 shared。
- ②**是否带业务语义**（本项目取舍，与官方做法不同）——**Android 官方把 UI module 列为 common/core 的例子**（理由：封装 widget 集供复用、集中主题以便换品牌时少重构），即官方允许带品牌表达的组件进 core。本项目因另有 shared 段，改为「无领域概念的进 core、带业务表达的进 shared」；这是本项目在 FFCA 框架上增设 core 段后的划分取舍，不据官方背书。
- ③**多消费方门槛**（本项目约定）——进 shared 需至少两个 feature 实际在用，只有一个用就留在该 feature；与 #5.17 末句一致。
- 与 NIA 的差异：NIA 只有 core 一个跨模块桶（以下为举例、非全量：`core:network`/`core:database`/`core:designsystem`/`core:ui`；当前 main 的 `core/` 共 17 个模块，另有 `core:domain`/`core:model`/`core:common`/`core:data` 等，以及 feature 的 api/impl 拆分与 sync/benchmark/test/app-nia-catalog 杂项模块），没有 shared，因此其归类**无法用来裁决** core/shared 二分；按本项目判据，NIA 的 `core:designsystem`、`core:ui` 应归 shared。
- FFCA 的 shared 是**包边界**概念、本项目 core/shared 是单包内的目录概念，两者不同层，仅「不得依赖 feature」这一条可类比。
- FFCA 原文称 domain「pure Dart」，本项目据此把 domain 收敛为不依赖 Flutter/shared、不带序列化注解——core 扩展为技术能力桶后，该收敛须限定为「domain 以 core 的纯 Dart 契约为基础依赖、不依赖 core 的技术实现模块；有明确业务 owner 的跨 feature 编排可定向依赖其他 feature 的公开 domain contract」（见 #5.4、#5.8）。

## 六、技术选型与否决

> 决策行按表内顺序编号，供全文引用：#6.1 状态管理、#6.2 路由、#6.3 网络层、#6.4 磁盘缓存、#6.5 本地化、#6.6 实验性 API、#6.7 依赖方向校验。

> **选型原则**：① 官方明确推荐优先采纳（recommendations 档位引用给出处）；② 维护状态作为风险登记（feature-complete、实验性 API 须配复核触发条件）；③ 与既有架构约束兼容（FFCA 分包体系、Riverpod、组合根装配）；④ 每条决策显式登记**接受的代价**与**复核触发条件**，不藏成本。

| 决策 | 未采纳方案（含暂缓评估项） | 理由 | 接受的代价 | 复核触发条件 |
|---|---|---|---|---|
| 状态管理 = Riverpod 3 | Provider / Bloc / Signals | ProviderContainer 脱离 BuildContext 的容器能力：overrides、autoDispose、ProviderContainer.test 与 DI 一体（Provider 包本身亦覆盖状态管理——官方以 ChangeNotifier/Listenable 驱动 widget 更新属 Conditional，且 “the decision comes down to personal preference” 出自 recommendations 页该 Conditional 行内（非 case study），见 #2.2；测试注入 Fake 用构造函数注入与 `Provider.value` / `ChangeNotifierProvider.value`，见 #2.3——真差异在 Riverpod 的容器能力，非「Provider 不覆盖」）；官方推荐 package:provider 的语境即 DI（“teams at Google recommend using package:provider”，见 #2.3），本项目偏离该 Strongly 推荐改用 Riverpod overrides，已在 #5.5「组合根装配」中标明——按选型原则①，此处登记该偏离；Notifier 对应官方 MVVM 的 ViewModel | 学习曲线；默认 retry 会把确定性失败拖成重试等待（见 #3.4，须按条目配置 retry）；autoDispose 默认值随手写/注解写法浮动（见 #3.6）；放弃 Bloc 的显式事件流（可日志、时序可测、事件组合表达力强，VGV 官方口径 "our preferred solution" 即 Bloc（出自 `vgv.dev/blog/why-we-use-flutter-bloc`，非 Layered Architecture 页）） | 团队极小（项目自定启发、无官方出处——勿与 FFCA overview "fewer than three features" 的 feature 数口径混同）、以 Bloc 存量为主、或出现多来源事件竞态 / 回放（时间旅行）需求时，重评 Bloc |
| 路由 = go_router + go_router_builder | auto_route（类型安全路由主流替代，**暂缓评估**、未深度评估——按原则③④须补评估后再登记为被否，现阶段不构成已否决策） | 官方 Recommend 档（原文：“Go_router is the preferred way to write 90% of Flutter applications.”，见 recommendations 页）；该句后紧跟例外限定 "There are some specific use-cases that go_router doesn't solve, in which case you can use the Flutter Navigator API directly or try other packages found on pub.dev."，不宜读成硬性规则 | go_router 已 feature-complete（Flutter 团队以 bug 修复和稳定性为首要重点，主动新功能开发不在当前计划中，但接受社区功能贡献，见 #2.12）；类型安全路由需 build_runner 生成、生成物入库与 CI 漂移检查成本（#5.20「生成物与 CI」已列，按原则④登记） | 出现长期未修的阻断 bug，或社区分叉活跃度反超 |
| 网络层 = Dio | http（Dart 团队维护的官方 pub 包）/ chopper | 提供拦截器、超时、取消等通用 HTTP 封装能力（#5.16「core 与 shared」将网络封装归入 core，DioClient 随之放 core/network）；非官方推荐项，属项目自选——选型依据是上述工程能力，而非官方背书 | 第三方依赖（非 Flutter SDK 自带）；DioException 作为基础设施异常须在 data 边界映射为 domain error（#5.15「异常映射」已约束其不穿透）；放弃 `package:http` 相对简单、轻量且由 Dart 团队维护的优势（它同样需经 pub 引入，并非 SDK 内置或零依赖） | 出现长期未修的阻断 bug，或拦截器/超时需求消失且团队要求最小依赖时，重评官方 http |
| 磁盘缓存 = sembast | sqflite / drift / isar / hive（结构化持久化选型未定，不在本文决策范围；本行只决策缓存场景） | Sembast 核心为 100% Dart、无原生平台插件；原生文件实现面向单进程 IO，Web 通过 `sembast_web` 使用 IndexedDB，需分别配置平台适配；按 Sembast API 编写的访问逻辑可复用，但底层存储与行为不应视为完全相同；支持事务与用户自定义 codec；与缓存场景契合——按 key 或查询缓存 JSON 文档，不必为缓存引入关系型 schema 与迁移链 | 原生文件实现打开时加载数据进内存，大缓存量占内存；`sembast_sqflite` 提供基于 SQLite 的跨进程安全存储，但须从 main isolate 使用，并非跨-isolate 并发方案；非关系型，不适合复杂关联查询与 schema 迁移——故只承担缓存，不替代业务数据库 | 缓存数据量增长到内存不可接受、确需多 isolate 并发写、或需求转为关联查询时，重新评估存储方案 |
| 本地化 = flutter_localizations + intl（官方方案） | slang + slang_flutter（社区替代，**暂缓评估**、未深度评估，现阶段不构成已否决策） | 官方文档路径（#2.10：l10n.yaml + arb-dir、生成到源码目录、pubspec `generate: true`，Flutter 3.32+ 口径）；按选型原则①官方优先 | 官方生成流程与 ARB 工作流；生成物入库策略统一适用；l10n 产物归属随架构形态：单包放 shared/l10n、分包移作 shared/localizations（项目约定的包命名；FFCA project_structure 页的 Folder layout 未在本次英文官方核对中逐字确认该路径，不作官方依据）——按 #5.16 归 shared（带业务表达的复用资产，见 #5.17③） | slang 等社区方案在官方工具链停滞或团队痛点显著时，补深度评估；Slang Flutter 集成需一并评估 slang_flutter 与生成器配置 |
| Riverpod 实验性 API（Mutations / Offline persistence）= **不进生产默认** | Mutations / Offline persistence 直接进生产 | 均为官方实验性（Mutations 概念页逐字 “the API may change in a breaking way without a major version bump”，见 #3.5；whats_new 页的 Offline persistence 与 Mutations 两个 experimental 小节各有一句复数写法 “in breaking ways”，Offline persistence 概念页仅标题标 experimental、无该句，同见 #3.5），破坏性变更可不升大版本发生 | 实验性 API 若发生破坏性变更，须跟随版本调整；API 稳定前试点实现可能返工，且需额外承担版本监控与迁移验证成本 | 官方宣布转稳定后，重评是否进 core 并补正式决策行 |
| 依赖方向校验 = **待定（未启用）** | 候选方案（均未启用）：import_lint / custom_lint | import_lint 2.0.0（2026-04-18）为 analysis_server_plugin 系，支持路径级规则（target/from/except glob；`severity: "error"` 仅对 `dart run import_lint` 生效、exit code 1 可接 CI，IDE 中插件固定 warning）；版本快照：2026-11-08 核验、2026-11-11 复核，riverpod_lint 最新 3.2.0 要求 analyzer `>=13.0.0 <15.0.0`，import_lint 2.0.0 要求 `^12.1.0`，区间不相交、同时启用无法解析。不用 custom_lint 的理由（仅对它成立）：其规则需用 `dart run custom_lint` 执行，`dart analyze` 不会拾取；当前 0.8.1 使用旧 analyzer_plugin 体系，与 analysis_server_plugin 并行会增加工具链复杂度——这是项目取舍，不据此断言其已废弃。 | 单包形态无 package 边界，依赖方向仅由 review 保障（dependency_validator 只查依赖声明，见 #5.21）；分包形态使依赖显式化，未声明的直接依赖可由 analyzer 诊断，但完整边界规则仍需 lint/CI 校验；路径级约束尚未接入自动校验（见 #5.22） | VGV 依据链：FFCA 标准站 project_structure 页 “This table is the whole rule, and it is the version a validation script implements”（依赖表即校验脚本实现，见 #4.3）、复盘提及校验钩子已打包为 FFCA 的 Claude Code 插件（原文注明当时尚未公开；截至 2026-11-11 VGV 已公开列出 `vgv-ffca-plugin`，其 open-source packages 页标注 2026-11-09 更新），是否可获取不影响本项决策；可参照的是标准站公开的依赖表与校验脚本口径。触发条件（任一）：选用 analyzer 12 兼容组合（riverpod_lint 3.1.4 要求 analyzer ^12.0.0、同属 analysis_server_plugin 系；属旧版折衷而非当前首选）；等待 import_lint 支持 analyzer 13+；或 riverpod_lint 放宽区间——均以 CI 全新解析验证 |

## 七、实践目录结构

依据 #5.1「目录形态」：默认单包、不物理分包、lib 内按 feature 分目录。FFCA 原生为 apps / features / shared 三段（#4.3），本结构在其上按 #5.16、#5.17 **只增设 core 段**，并细化 core / shared 的职责与判定顺序（core 非 FFCA 原生，见第四节术语表），故 lib 顶层为 app / features / core / shared 四段（依赖方向另见下方约定：app → features → shared → core）。l10n 资源（.arb 源 + 生成物，见 #6.5）按 #5.16 归 shared：单包形态直接放 `shared/l10n/`（`l10n.yaml` 显式设置 `synthetic-package: false`，`arb-dir`/`output-dir` 指向此处，Flutter 3.32+ 生成到源码目录），分包形态移作 `shared/localizations` 包。它不再是顶层独立段，也不再需要「物理摆放例外」——直接由 #5.16 判据得出。

条件性目录（下表统一标〔按需〕）：#2.2 把 domain 层与 API/domain model 分离列为 Conditional，本项目按 #5.4 在 feature 内常设 domain 目录（属项目取舍），但 DTO 与 mapper 按 #4.5、#5.11 仅在模型确有差异时建立，是否采用独立 domain model 亦属同一取舍；#5.7、#5.8 规定 UseCase 默认不建；#6.6 的实验性 API 不进默认结构，按需在 feature 内试点、不进 core。〔按需〕目录不属于默认结构，不应照抄。

典型数据调用链：`Widget/Page → ViewModel（Riverpod Notifier）→ UseCase（按需）→ Repository → DataSource/Service → Dio、MQTT、存储或 Platform Channel`。UseCase 可省略；部分简单场景可由 Repository 直接提供 ViewModel 所需的数据。箭头表达职责调用方向，不代表每类功能都必须经过全部节点；跨 Repository 编排按 #5.7、#5.8 处理，Repository 不互相调用。

```text
project/
├── pubspec.yaml
├── analysis_options.yaml          # flutter_lints 基线 + plugins 装 riverpod_lint（#2.14、#3.7）
├── l10n.yaml                      # synthetic-package: false；arb-dir 指向 lib/shared/l10n/arb；pubspec 开 generate: true（#2.10）
├── build.yaml                     # 〔按需〕需要自定义 builder 配置时使用；生成器本身不要求此文件（#4.5、#3.6）
├── assets/
│   └── dev/                       # dev 环境本地数据（staging 走 HTTP server）（#2.5）
├── android/                       # 原生侧 flavor / Gradle 属性的独立配置落点；官方另有 `-Pdart-defines` 通道，本项目不依赖（#2.8）
├── ios/                           # 原生侧 flavor / scheme / xcconfig 的独立配置落点；dart-defines 亦可传至 iOS/macOS 构建，本项目不依赖（#2.8）
├── .github/workflows/ci.yml       # 明确 Flutter/Dart SDK 基线；全新 pub get 后运行适用生成器、analyze/test，再查 git status --porcelain；运行 dependency_validator、pub downgrade 验下限（#3.7、#5.20、#5.21、#5.23、#6.7）
├── lib/
│   ├── main.dart                  # 生产入口
│   ├── main_dev.dart              # Dart 入口；与原生 flavor 分别选择（如 flutter run --flavor dev -t lib/main_dev.dart）；--dart-define 供编译期配置，原生侧通道见 #2.8（#2.5、#2.8）
│   ├── main_staging.dart
│   ├── app/                       # app 层：bootstrap 是组合根，另含路由与 app 专属编排（#5.5、#5.8）
│   │   ├── app.dart               # MaterialApp.router、主题与本地化挂载；不承载异步启动初始化
│   │   ├── bootstrap.dart         # 组合根：日志、全局错误入口、预创建存储实例、ProviderScope 与环境 overrides（#5.5、#5.24、#2.11）；不在此同步恢复登录令牌
│   │   ├── router/                # go_router 路由表、登录守卫、go_router_builder；session 变化触发 redirect 刷新（#5.18、#6.2）
│   │   ├── shell/                 # 〔按需〕splash、底部导航 shell 与 tab 定义；session 恢复中显示 splash，不提前跳转
│   │   └── application/           # 名称易混：Bizzotto 的 application 层是 feature 内中介层，非组合根（#4.1）
│   │       └── use_cases/         # 〔按需〕无单一 owner 的 app 专属编排（#5.8 原文路径）；分包形态类名用 Command/Query（#4.3）
│   ├── features/                  # 按「用户做什么」划分，不按技术类型（#4.2、#5.1）
│   │   └── auth/                  # 示例 feature；其他 feature 按实际职责裁剪
│   │       ├── auth.dart          # 约定 barrel：仅暴露 domain contract 与公开 presentation API；单包无编译期强制（#5.3、#4.3、#5.22）
│   │       ├── domain/            # 纯 Dart：模型、规则、Repository 接口；有 owner 的跨 feature 编排也在此（#5.4、#5.8）
│   │       │   ├── models/        # 〔按需〕采用独立 domain model 时用不可变对象（优先 freezed）、不附加序列化注解（#5.11、#4.5）
│   │       │   ├── repositories/  # 抽象接口；实现在 data（#5.6）
│   │       │   ├── validators.dart # 纯业务校验返回错误类型／枚举；界面负责本地化文案
│   │       │   └── use_cases/     # 〔按需〕默认不建；复杂/复用/多 Repository 才加（#5.7）
│   │       ├── data/              # 实现层；DTO 与 mapper 按需建立、不出 data（#5.11、#4.5）
│   │       │   ├── data_sources/  # 只执行外部访问并返回原始数据／DTO，不承载业务编排（#2.7）
│   │       │   ├── dto/           # 〔按需〕仅 API/domain model 有差异时建；请求与响应 DTO 可分别配置序列化方向（#5.11；序列化方向不属 #2.2 的 API/domain model 分离条款）
│   │       │   ├── mappers/       # 〔按需〕DTO 到 domain model 的转换（#2.2、#5.11）
│   │       │   └── repositories/  # 实现 domain 接口；令牌持久化与基础设施异常映射在此协调（#5.15、#5.25；#5.14 为 Result 边界）
│   │       └── presentation/
│   │           ├── login/         # 页面、autoDispose Notifier 与不可变表单状态；Notifier await 后检查 ref.mounted，成功后通知 session
│   │           ├── register/      # 页面、autoDispose Notifier 与状态；注册成功后由 View 监听并导航，不改变登录态
│   │           ├── widgets/       # 〔按需〕登录／注册共用组件；没有实际复用时不建
│   │           ├── session_notifier.dart # 应用级存活的认证状态恢复与 signedIn/signedOut；属于 auth、不是无业务语义的 core
│   │           └── auth_providers.dart # 声明 domain Repository 抽象类型 provider；不提供第二套默认实现，具体实现由 app override
│   ├── core/                      # 与业务语义无关的技术能力，不得 import features（#5.16、#5.17）
│   │   ├── result.dart            # Result<T>（#5.14）
│   │   ├── load_state.dart        # 不可变 LoadState<T>，仅用于无需 AsyncValue 语义的读取场景（#5.13）
│   │   ├── submit_state.dart      # 不可变 SubmitState<T>（idle/running/success/error），提交默认用它（#5.13）
│   │   ├── errors/                # 仅无业务语义的错误基类/接口；具体业务错误归 feature/domain（#5.15、#5.16、#5.17①、#2.13）
│   │   ├── config/                # AppConfig：API baseUrl、缓存目录名、查询上限等常量（#5.16）
│   │   ├── network/               # Dio 工厂与拦截器：认证、错误映射、脱敏日志，顺序 logging→auth→error（#6.3）
│   │   ├── navigation/            # 路由路径常量；具体页面与守卫仍在 app/router（#5.18）
│   │   ├── storage/               # 与业务语义无关的本地存储封装：安全 KV（令牌等敏感值的读写与清除，不含认证判断）与 Sembast 缓存；缓存仅由 DataSource 调用（#5.25、#6.4）
│   │   ├── logging/               # Logger 装配 + ProviderObserver 实现（#5.24、#3.9）
│   │   ├── git/                   # 本地 git CLI 设施：runner/resolver/url（无 tag/commit 对比语义；若出现 git 业务规则，下沉对应 feature 的 domain）
│   │   ├── utils/                 # 〔按需〕纯 Dart、无业务语义的小工具、扩展与生命周期 mixin
│   │   └── ports/                 # 〔按需〕Clock 等 port 接口；有替换需求才拆（#5.17④、#5.24；AppLogger 接口现与实现同放 core/logging，见 #5.24）
│   ├── shared/                    # 不属于单一 feature、但带业务表达的复用资产（#5.16）
│   │   ├── l10n/                  # arb + 生成物 + AppLang/文案访问；业务 AppError 到文案的映射留在对应 feature presentation（#2.10、#6.5；映射归属为项目约定）
│   │   ├── widgets/               # 〔按需〕跨 feature 通用组件（≥2 个 feature 在用）
│   │   ├── mixins/                # 〔按需〕涉及 Flutter 生命周期、且带业务判断的防抖/提交保护；无业务语义的生命周期 mixin 归 core/utils
│   │   └── design_system/         # 设计系统与品牌化组件（#5.16）
├── test/                          # 大体镜像 lib/ 结构（#4.2）；shared/l10n 不镜像（.arb 源 + 生成物，生成物不做单测）；fakes/ 为测试专用
│   ├── app/                       # 装配、导航（#5.19）
│   ├── features/auth/...          # domain 规则 / data 映射 / Notifier + widget 分层测试（#5.19）
│   ├── core/
│   ├── shared/
│   └── fakes/                     # Repository/DataSource Fake，供 app 注入与 ProviderContainer.test（#5.6、#5.19、#3.3）
└── integration_test/              # 集成流程测试（#2.9、#5.19）
```

三条依赖约定（当前由 review 保障，路径级自动校验待 #5.22 / #6.7 启用）：

- 主依赖方向：app 层（bootstrap 为组合根）→ features → shared → core，默认单向且不得成环（#5.3）；shared/l10n 同属 shared 层，无额外例外。feature 内 domain 只依赖 core 的纯 Dart 契约，不依赖 core 技术实现模块；有 owner 的跨 feature 编排仅按 #5.8 定向依赖对方公开 domain contract；data 与 presentation 都依赖本 feature domain，presentation 不 import data，实现装配只发生在组合根（#5.4、#5.5）。
- 公开面最小化：feature 间只经 barrel 暴露的 domain contract 与公开 presentation API 协作（#5.3）。单包形态没有 package 边界；分包使依赖显式化，但未声明依赖由 analyzer 诊断、边界规则仍需 lint/CI 执行（#4.3）。「barrel 不 import lib/src」「feature 不 import 他 feature 的 data」均为约定；#5.22 已记明路径级约束尚未自动校验、import_lint 与 riverpod_lint 的 analyzer 区间不相交，故不得称其为编译期不变量。
- 归属口诀（#5.17）：有业务 owner 的留 feature，app 专属留 app；与业务语义无关的技术能力（network/storage/logging/config/error/navigation）与纯契约进 core；不属于单一 feature、但带业务表达的 UI 组件/业务规则/l10n 进 shared；证明不了第二个消费方就先留在 feature，防垃圾场。core 与 shared 的共同硬约束是**不得依赖任何 feature**。

若经评审满足 #5.1 的分包条件（多 app 复用 / feature 所有权并行 / 大量 coding agent 并行），此树按目录→包同构平移：`features/auth/{domain,data,presentation}` → `auth_domain` / `auth_data[_backend]` / `auth_presentation`，`app/` → `apps/`（每个 app 各有自己的 bootstrap 组合根，#1.4），`core/`、`shared/` → 独立包（`shared/l10n` → `shared/localizations`，见 #6.5），跨包只经公共 barrel（#5.2、#4.3），依赖规则不变。分包形态下类名按 #4.3 用 Command/Query（`use_cases/` 目录名保留），`application/` 的同形异义辨析见上（#4.1）。

平移不只是目录改名，还需同步配置：根 pubspec.yaml 增加 `workspace:` 列表、各成员包声明 `resolution: workspace`、全仓收敛为单一 pubspec.lock，成员包 SDK 下限须 ≥ Dart 3.6.0（#2.15 已提示分包放大依赖冲突风险）；`build.yaml`、生成物与 `test/` 随各包自带（#5.19、#5.20、#5.21），CI 从 workspace 根运行（或逐包运行）dependency_validator（#5.21）。
