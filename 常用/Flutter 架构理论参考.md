# Flutter 架构理论参考

> 标注约定：来源未加括注的，均已对照官方核实（2026-11）；来源后括注表示降级引用、部分核实、社区经验等非官方立场。包版本号以 pub.dev 当日为准。
> 通用工程清单（测试、性能、安全、可观测性、存储、工程化）已拆至同目录 [Flutter 工程实践清单](Flutter%20工程实践清单.md)。

## 一、Flutter / Dart 官方

| # | 来源 | 要点 |
|---|------|------|
| 1 | Flutter 架构指南（docs.flutter.dev/app-architecture/guide） | UI 层（View + ViewModel，MVVM）+ Data 层（Repository + Service）；Domain 层可选，只放 UseCase。Repository 把原始数据加工成 domain model 供 ViewModel 消费；ViewModel 可直接访问 Repository，也可按需经过 UseCase，官方示例两种并存。 |
| 2 | Architecture recommendations 页 | 建议分 Strongly recommend / Recommend / Conditional 三档。主干：抽象 Repository（Strongly，便于 dev/staging 切换实现）；API model 与 domain model 分离（Conditional，大项目再分）；go_router（Recommend）。其余条目：① UI/Data 分层、data 层 Repository 模式、UI 层 View+ViewModel 的 MVVM（均 Strongly）；② 逻辑不进 widget（Strongly：View 只保留简单 if 显隐、动画逻辑、基于设备的布局逻辑、简单路由逻辑，其余归 ViewModel）；③ 单向数据流（Strongly：数据 data→UI 流动，UI 交互发送到 data 层处理）；④ 不可变数据模型（Strongly：字段改动只能新建对象替换，防止数据在 domain/data 之外被修改）+ freezed 或 built_value（Recommend：生成 ==、copyWith、JSON 序列化等，注意代码生成增加构建耗时）；⑤ DI（Strongly：避免全局可达对象，官方示例用 provider 包）；⑥ Command 封装用户交互事件（Recommend：防止渲染错误、标准化 UI→data 的事件通道）；⑦ ChangeNotifier/Listenable 驱动 widget 更新（Conditional：只是 SDK 自带的便利选项，状态管理可自选，即本项目用 Riverpod 不冲突）；⑧ domain 层（Conditional：仅当复杂逻辑挤占 ViewModel、或在多个 ViewModel 间重复时启用，多数 app 属不必要开销）；⑨ 统一命名约定（Recommend：按架构角色命名，如 HomeViewModel、HomeScreen、UserRepository、ClientApiService；共享组件目录建议叫 ui/core 而非 widgets，避免与 Flutter SDK 概念混淆）。 |
| 3 | Architecture case study 系列（Data layer、Communicating between layers、Dependency injection） | 依赖方向 View → ViewModel → Repository → Service，Service 不知上层；View 只认一个 ViewModel，ViewModel 可持多个 Repository。DI 示例用 provider，只借其构造函数注入思想；另有 Testing each layer 页（见《Flutter 工程实践清单》测试一节）。 |
| 4 | Common architecture concepts | 单一数据源（Repository 是该类数据的唯一 SSOT 与唯一修改点）、单向数据流（数据 data→UI，交互 UI→data）、关注点分离等概念定义。 |
| 5 | Compass 案例（降级引用：仓库未归档、仍在维护，但官方称多数情况不再新增 sample） | flutter/samples 的架构官方案例：目录结构、main_development / main_staging 入口、dev 用 assets 本地数据 / staging 用 HTTP server 的双环境、高测试覆盖率。用 Provider + ChangeNotifier，只借结构，不借状态管理写法。 |
| 6 | Cookbook 设计模式（Result、Command、Optimistic state、Offline-first） | Result 用 sealed 表达成功/失败并在层间传递；Command 封装一次用户操作（携带 running/error 状态，防重复执行与渲染错误）；是本项目 core 的 Result/Exception 结构与"提交状态"设计的依据。 |
| 7 | Cookbook 持久化（Key-value、SQL） | 两个官方范例均为 Repository + Service 分层：Key-value 用 shared_preferences，SQL 用 sqflite；可对照本项目 data 层"Repository 持 Service"的写法。 |
| 8 | Flavors 文档、--dart-define（--dart-define-from-file 无官方独立文档页） | main_* 入口 + flutter run --flavor 组织环境。dart-define 值仅供 Dart 编译期读取；from-file 在 **3.16.x 及之前**还会把值传入 Gradle 属性与原生环境变量，**3.19 起该行为被移除**（PR #136865；issue #138793、#143633 明确"3.16.5 可用、3.19.0 起失效"），原生侧（Manifest、plist）需自行同步配置。在原生构建里解码 DART_DEFINES 的 base64 属未文档化实现细节，官方明确不保证，勿依赖。值会编译进二进制，不能放秘密。 |
| 9 | Flutter 测试文档 | unit / widget / integration 三类的定义、适用场景与官方 recipes，详见《Flutter 工程实践清单》测试一节。 |
| 10 | 国际化文档 | flutter_localizations + intl；l10n.yaml 配 arb-dir 与输出位置，默认 lib/l10n。synthetic package:flutter_gen **已弃用**：3.32 起默认生成到源码目录、pubspec 必须写 flutter: generate: true，后续稳定版移除 flutter_gen 支持；synthetic-package 仍是 l10n.yaml 合法选项，仅作迁移过渡，按所装 SDK 文档为准。 |
| 11 | 错误处理文档 | FlutterError.onError 捕获框架层错误，PlatformDispatcher.instance.onError 捕获未进入框架的异步错误（如 isolate 回调）；本项目统一在 bootstrap 注册，接崩溃上报见《Flutter 工程实践清单》可观测性一节。 |
| 12 | go_router | 声明式路由：redirect 支撑登录态守卫，ShellRoute/StatefulShellRoute 支撑底部导航多栈。官方 Roadmap 明确 feature-complete：团队以修 bug 与稳定性为主，新功能靠社区贡献；不是弃维护。 |
| 13 | Dart 3 sealed class 与模式匹配 | sealed 类型层级 + switch 穷尽检查，漏分支编译期报错；用于本项目的 Result、SessionState、AppError，避免 if-else 链漏处理。 |
| 14 | Effective Dart、flutter_lints | 官方风格指南与默认 lint 集，作为 analysis_options.yaml 基础，团队规则在其上增量，不另起炉灶。 |
| 15 | Pub workspaces（dart.dev/tools/pub/workspaces） | 根 pubspec 的 workspace 列表 + 成员包 resolution: workspace：全仓统一依赖解析、单一 pubspec.lock、分析器共享上下文降低内存。workspace 列表自 Dart 3.6 可用；glob 通配写法（packages/*）需 3.11。单包分层项目不需要。 |

## 二、Riverpod 3

入口是 https://riverpod.dev/。旧路径（/docs/introduction/why 等）会 404，以新站导航为准。

| # | 来源 | 要点 |
|---|------|------|
| 16 | 官方文档（Notifier、AsyncNotifier、StreamNotifier） | 分别对应同步、异步、流式状态。ViewModel 以 Notifier 实现：build() 产出初始状态，公开方法承载用户操作，UI 用 ref.watch 读状态、ref.read 调方法。 |
| 17 | What's new in 3.0（/docs/whats_new）、DO/DON'T | 3.0 新特性总览与官方用法约定入口，写新代码前先过一遍。 |
| 18 | Overrides 与测试文档 | ProviderScope.overrides 在 bootstrap 注入预创建实例（存储、配置）；ProviderContainer.test 创建容器并在测试结束自动销毁；优先 mock Repository 而非 Notifier；NotifierProvider.overrideWithBuild 只替换 build 的初始状态，保留真实方法逻辑。 |
| 19 | Automatic retry | provider 初始化抛异常默认自动重试：200ms 起翻倍、封顶 6.4s、最多 10 次，累计约 38 秒（0.2+0.4+0.8+1.6+3.2+5×6.4）；Error 与 ProviderException 不重试。可在 ProviderScope(retry:) 全局或单 provider 定制，返回 null 即不重试。retry 只在初始化抛异常时触发，build() 内 catch 后返回正常状态不会触发——要么让它抛、要么关掉，别半抛半吞。await provider.future 会等重试结束才抛错，因此 SessionNotifier 恢复失败默认会让 splash 被拉长近 40 秒：建议该 provider 设 retry: (_, __) => null（或仅对网络类错误重试一次），并给 splash 加超时兜底。 |
| 20 | Mutations（实验性：隔离在 feature 内，不进 core） | Mutation 对象跟踪一次副作用的 idle/pending/error/success，UI 用 ref.watch 驱动提交按钮与提示；同时解决 onPressed + ref.read + autoDispose 导致副作用进行中 provider 被销毁的问题。官方明示 API 可能不升大版本破坏性变更。 |
| 21 | Offline persistence（实验性：不进 core） | Storage 接口 + Notifier 内 persist() 把 provider 状态落盘，默认缓存 2 天（可配），key 须全局唯一且含 family 参数；官方配套 riverpod_sqflite，测试用 Storage.inMemory。定位为数据库的简单包装，不替代 Repository 与迁移管理。 |
| 22 | riverpod_generator | @riverpod 注解 + build_runner 生成 provider 与 family；手写与注解两种风格按项目二选一，部分 lint 只对注解写法生效。 |
| 23 | riverpod_lint | 3.1.0 起改用 analysis_server_plugin（analysis_options.yaml 顶层 plugins: 声明），不再基于 custom_lint，dart analyze 直接出警告与 quick fix；3.1.9 要求 Dart 3.13。 |

3.0 写法约定：
- 用 Notifier、AsyncNotifier、StreamNotifier。
- StateProvider、StateNotifierProvider、ChangeNotifierProvider 是 legacy（移到 legacy.dart），新代码不用。
- AsyncValue 是 sealed，valueOrNull 改名 value，出错时 value 为 null。
- Provider 失败会被包成 ProviderException，重试时不重复追究（见 #19）。
- 销毁后使用 Ref 抛 UnmountedRefException：异步代码 await 之后必须先判 ref.mounted 再碰 ref/state。

## 三、社区与公司实践

| # | 来源 | 要点 |
|---|------|------|
| 24 | Bizzotto 的 Riverpod 架构文章（个人实践，2023 年正文，只取思路，代码未逐段核 3.0） | data / domain / application / presentation 四层。注意他的 application 层是 feature 内部的 Service 中介层（协调多个 repository、承载跨数据逻辑），不等于本项目的 app/ 组合根，不要对等。 |
| 25 | Bizzotto 项目结构篇（codewithandrea.com/articles/flutter-project-structure/） | feature 按"用户做什么"划分（auth、cart、checkout、orders），而非按技术类型；顶层 shared 只放 common_widgets、constants、exceptions、localization、routing、utils；test/ 目录镜像 lib/ 结构。 |
| 26 | VGV FFCA 标准站（分包 monorepo 实践；单包项目降级引用，约束力只剩 lint，不要宣称"遵循 FFCA"） | engineering.verygood.ventures/architecture/ffca/：apps / features / shared 三段；feature 拆 {feature}_domain/_data/_presentation 包，domain 纯 Dart 放模型、业务规则与 repository 接口，data 实现接口并管 DTO 映射，presentation 不依赖任何 _data；shared 判据"能否发 pub.dev 给陌生人原样用"且只依赖外部包；auth、analytics、user_profile 属 headless feature（无 presentation）；use_cases 目录内类以 Command（写）/ Query（读）命名；shared 需要 feature 概念时声明 port，由 app 用 adapter 满足。用 Bloc，只借依赖规则。 |
| 27 | VGV Scooter 生产复盘（2026-09，内容未逐条复核） | 16 个 feature、48 个 feature 包、共 88 包的生产经验：{Feature}Handlers 对象替代多回调参数；{feature}_data_{backend} 后缀区分后端实现；shared 永不依赖 features；跨 feature 的 Query/Command 在组合根装配。 |
| 28 | Very Good Architecture（2022 博客）、very_good_analysis | data / domain(business-logic) / presentation 分层思路与严格 lint 集；与 FFCA 是两套体系，概念和目录不要混用。 |
| 29 | freezed、json_serializable | freezed 生成不可变模型（copyWith、==、sealed 联合），json_serializable 生成 DTO 的 fromJson/toJson；DTO 只在 data 层内部使用，经 mapper 转 domain model 后出层。 |

## 四、通用理论（只借依赖规则）

| # | 来源 | 要点 |
|---|------|------|
| 30 | Cockburn 六边形架构原文 | ports & adapters：领域只定义 port（接口），与外界的交互由可替换的 adapter 实现，替换实现不影响领域。本项目只借这一机制。 |
| 31 | Martin 的 Clean Architecture（2012 博客） | 源码依赖只能指向内层，内层不知道外层存在；四圈只是示意，层数与命名按需裁剪。 |
| 32 | Fowler 的 Repository 模式 | Repository 介于 domain 与 data mapping 之间，对上层屏蔽存储细节；"Repository 接口放 domain 层"的出处之一（Flutter / Android 官方只要求抽象 Repository，没规定放哪层）。 |
| 33 | Evans《领域驱动设计》（仅经 Fowler 页间接确认） | Repository 概念的另一出处。 |
| 34 | Android 官方架构指南 | DataSource 术语来源；官方只有"API model 与 domain model 分离"的说法，DTO、DataSource 是社区惯用词，引用时别当成 Flutter 官方概念。 |

## 五、依赖规则强制工具

| # | 工具 | 要点 |
|---|------|------|
| 35 | import_lint（首选） | analysis_options.yaml 里用 target / from / except 写导入约束，可把 app→features→shared→core 落成可执行规则。2.0.0 起要求 Dart 3.10+ / Flutter 3.38，analysis_server_plugin 机制。与 riverpod_lint 同机制可共存，但 SDK 下限被顶到 riverpod_lint 的要求（3.1.9 为 Dart 3.13），analyzer 区间冲突时表现为 pub version solving failed 而非运行期报错。注意：CLI（dart run import_lint）里 severity: error 退出码 1 可接 CI；IDE 内严重级别固定为 warning，想显红需另配 analyzer: errors: import_lint: error。 |
| 36 | dependency_validator | 依据代码 import 判定 missing / under-promoted / over-promoted / unused 四类依赖声明问题（声明了没用、用了没声明、层级放错等）；不做分层与路径约束，多包项目作补充。 |
| 37 | DCM（商业工具；规则名与免费档覆盖以 dcm.dev 当前文档或实测为准） | avoid-banned-imports 可按路径 deny 导入、强制层级依赖；该规则名源自已停更的开源 dart_code_metrics，商业版是否沿用同名规则、免费档是否覆盖需实测。另有数百条 lint 与代码健康指标。 |

## 六、本项目自定

> 原则：**官方或社区已有明确做法的一律遵循，不自造规则**；本节只保留"官方/社区未规定"或"需在多个并存选项中取舍"的决定。以下为 2026-11 约定，随代码演进复核。
> 目录命名不单独算作自定：理论或社区已有名称的沿用（features、shared、data、domain、presentation 等），没有现成名称的给个推荐名即可（如 core/ports + app/adapters，机制出自六边形与 FFCA），无需逐条立项。

- **依赖方向**：app → features → shared → core 单向、不被反向依赖，遵循 Clean 与 FFCA。feature 内部**遵循 Flutter 官方 MVVM**：ViewModel 直接持 data 层 Repository（官方 case study 即此结构），禁止 import dto / sources / mappers；不采用 FFCA 更严的"presentation 不依赖任何 data"（该规则以 domain 接口为前提，属可选的更严方案，不拿来评审本项目代码）。
- **core 依赖口径**：官方/社区无硬性规定，本项目从宽：不依赖 Flutter UI（widgets/material），允许纯 Dart 包与必要的 Flutter 插件（dio、flutter_secure_storage、shared_preferences 等可留 core）。
- **shared/**：遵循 FFCA 判据"能否发 pub.dev 给陌生人原样用"，只放真共享，防垃圾场。
- **Repository 抽象**：遵循官方 Strongly recommend——**默认定义抽象 Repository + 实现类**，支持 dev/staging/Fake 替换。接口位置官方不规定，从简：默认与实现同放 data 层（Andrea 风格）；需跨 feature 共享契约时再放 domain（Fowler/Clean/FFCA 出处）。mocktail 对抽象与具体类都可 mock，测试不构成是否抽象的决策依据。
- **UseCase**：遵循官方 Conditional，默认不建。添加触发器（满足其一即建）：单个 ViewModel 依赖 Repository 超过 3 个；同一段编排逻辑在两个 Notifier 中重复出现；需要组合多个 Repository 的数据供一个界面使用。
- **SDK 约束**：不在正文钉死版本号，定版本前用 dart pub outdated 与 pub.dev 各包页面核对 SDK 下限，以当日为准。
- **core/logging**：官方无强制日志方案，按需建统一 AppLogger 接口（社区实践：脱敏 + error 级转发崩溃平台，见《Flutter 工程实践清单》可观测性一节）。

### 技术选型与否决

- **Riverpod 3 而非 Provider**：Provider 依赖 widget 树、类型与生命周期错误在运行时才暴露、测试需构造 widget 环境；Riverpod 同作者继任，provider 脱离 BuildContext、可 override 测试、生命周期（autoDispose）与 DI 一体。官方 Compass 用 Provider 仅借其结构，不构成选型依据。
- **Riverpod 3 而非 Bloc**：Bloc 事件/状态分层带来大量模板代码与概念成本，中小团队收益不成比例；FFCA、VGV 用 Bloc 只借其依赖规则，不借状态管理。Riverpod 的 Notifier 直接对应官方 MVVM 的 ViewModel，测试用 ProviderContainer 即可。
- **import_lint 而非 custom_lint 系**：custom_lint 上游（invertase/dart_custom_lint）已于 2026-03 归档，官方推荐 analysis_server_plugin，新机制只有这一条路。
