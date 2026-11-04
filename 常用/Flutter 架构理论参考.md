# Flutter 架构理论参考

> 状态标注：✅ 已对照官方核实（2026-11）；⚠️ 部分核实或需降级引用；❌ 未采用。
> 包版本号以 pub.dev 为准，本文标注的为核实时版本。

## 一、Flutter / Dart 官方

| # | 来源 | 用途 | 状态 |
|---|------|------|------|
| 1 | Flutter 架构指南（docs.flutter.dev/app-architecture/guide） | UI 层 + Data 层，Domain 层可选；Views、ViewModels、Repositories、Services | ✅ |
| 2 | Architecture recommendations 页 | 三档优先级。抽象 Repository 为 Strongly recommend（便于按 dev/staging 切换实现）；API model 与 domain model 分离为 Conditional；go_router 为 Recommend | ✅ |
| 3 | Architecture case study 系列（Data layer、Communicating between layers、Dependency injection） | 依赖方向：View 只认一个 ViewModel，ViewModel 持 Repository，Repository 持 Service，Service 不知上层。DI 示例用 provider，只借构造函数注入 | ✅ |
| 4 | Common architecture concepts | 分层与单向数据流 | ✅ |
| 5 | Compass 案例（flutter/samples 的 compass_app） | 目录、main_development/main_staging 入口、分层对照。用 Provider + ChangeNotifier，不用 Riverpod，只借结构，不借状态管理写法 | ⚠️ 降级引用。仓库未归档、仍在维护，但 README 称大多数情况不再新增 sample |
| 6 | Cookbook 设计模式（Result、Command、Optimistic state、Offline-first） | core 的 Result/Exception 依据 | ✅ |
| 7 | Cookbook 持久化（Key-value、SQL） | Repository + Service 范例 | ✅ |
| 8 | Flavors 文档、--dart-define | Flavor 入口与环境配置 | ⚠️ --dart-define-from-file 无官方独立文档页。flutter/flutter#138793：3.16 起其值不再传入原生构建（Gradle/环境变量），仅供 Dart 编译期；原生侧需自行同步配置 |
| 9 | Flutter 测试文档 | unit、widget、integration | ✅ |
| 10 | 国际化文档 | flutter_localizations、intl、l10n.yaml、gen-l10n，默认 lib/l10n。3.32 起需 pubspec 配 `flutter: generate: true`，生成代码进源码目录而非 synthetic package | ✅ |
| 11 | 错误处理文档 | FlutterError.onError、PlatformDispatcher.instance.onError，放 bootstrap | ✅ |
| 12 | go_router | routing 挂载点。官方 Roadmap 明确 feature-complete，主打修 bug 与稳定性，新功能靠社区贡献 | ✅ |
| 13 | Dart 3 sealed class 与模式匹配 | 实现 Result，穷尽检查 | ✅ |
| 14 | Effective Dart、flutter_lints | 风格、基础 lint | ✅ |
| 15 | Pub workspaces（dart.dev/tools/pub/workspaces） | 多包依据，Dart 3.6 起，glob 需 3.11。单包分层可不加 | ✅ |

## 二、Riverpod 3

入口是 https://riverpod.dev/。/docs/introduction/why、/docs/what_is_riverpod 这类旧路径会 404，以新站导航为准。

| # | 来源 | 用途 | 状态 |
|---|------|------|------|
| 16 | 官方文档（Notifier、AsyncNotifier、StreamNotifier） | ViewModel 实现 | ✅ |
| 17 | What's new in 3.0（/docs/whats_new）、DO/DON'T | 新特性总览、用法约定 | ✅ |
| 18 | 3.0 迁移指南 | 行为变化 | ✅ |
| 19 | Overrides 与测试文档 | bootstrap 装配、Flavor、测试。ProviderContainer.test 自动销毁；mock Notifier 一般不建议，非要 mock 须继承 Notifier 基类；优先 mock Repository。NotifierProvider.overrideWithBuild 可只 mock build | ✅ |
| 20 | Automatic retry | 初始化失败默认指数退避重试（200ms 起翻倍到 6.4s，默认最多 10 次；Error 与 ProviderException 不重试）。影响 Repository 错误处理与 SessionNotifier 恢复策略，可用 retry 关闭或定制 | ✅ |
| 21 | Mutations（实验性） | 副作用状态，对应 Command 模式。隔离在 feature 内。官方明示 API 可能在不升大版本时破坏性变更；run 回调写法以所装版本为准 | ✅ |
| 22 | Offline persistence（实验性） | Storage 接口 + Notifier 内 persist，官方配套 riverpod_sqflite。不替代 Repository | ✅ |
| 23 | riverpod_generator | 仅用注解写法时需要 | ✅ |
| 24 | riverpod_lint | 3.1.0 起用 analysis_server_plugin（analysis_options.yaml 顶层 plugins:），不再基于 custom_lint；SDK 要求高（3.1.9 需 Dart 3.13） | ✅ |

3.0 写法约定：
- 用 Notifier、AsyncNotifier、StreamNotifier。
- StateProvider、StateNotifierProvider、ChangeNotifierProvider 是 legacy，新代码不用。
- AsyncValue 是 sealed，valueOrNull 改名 value。
- Provider 失败会被包成 ProviderException。
- 销毁后使用 Ref 会抛异常（UnmountedRefException），异步代码 await 之后必须用 ref.mounted 判断。

## 三、社区与公司实践

| # | 来源 | 用途 | 状态 |
|---|------|------|------|
| 25 | Bizzotto 的 Riverpod 架构文章 | 4 层：data、domain、application、presentation。个人做法，正文 2023 年，只取思路，代码未逐段核 3.0。他的 application 层是 feature 内部的 Service 中介层，不等于本项目的 app/ 组合根，不要对等 | ✅ |
| 26 | Bizzotto 项目结构篇（codewithandrea.com/articles/flutter-project-structure/） | feature 按"用户做什么"划分（auth、cart、checkout、orders）；顶层 shared 为 common_widgets、constants、exceptions、localization、routing、utils；test/ 镜像 lib/ | ✅ |
| 27 | VGV FFCA 标准站（engineering.verygood.ventures/architecture/ffca/overview/，含 project_structure、faq） | 三段 apps/features/shared；{feature}_domain/_data/_presentation 分包；shared 判定为"能否发 pub.dev 给陌生人原样用"；auth、analytics、user_profile 属 headless feature；_presentation 不依赖任何 _data；shared 只依赖外部包；shared 需要 feature 概念时声明 port，由 app 用 adapter 满足。用 Bloc，只借依赖规则 | ✅ 分包 monorepo 实践。单包项目为降级引用，约束力只剩 lint |
| 28 | VGV Scooter 生产复盘（2026-09，16 feature、48 feature 包、共 88 包） | {Feature}Handlers 替代多回调；{feature}_data_{backend} 后缀；shared 永不依赖 features；跨 feature Query/Command 在组合根装配 | ⚠️ 内容未逐条复核 |
| 29 | Very Good Architecture（2022 博客，Layered：data/domain/business-logic/presentation）、very_good_analysis | 分层参考、lint。与 FFCA 是两套，别混用 | ✅ |
| 30 | freezed、json_serializable | Model、DTO | ✅ 版本以 pub.dev 为准 |
| 31 | very_good_cli 项目模板 | 模板参考，未采用 | ❌ |

## 四、通用理论（只借依赖规则）

| # | 来源 | 用途 | 状态 |
|---|------|------|------|
| 32 | Cockburn 六边形架构原文 | ports 与 adapters，无 core/ports 目录约定 | ✅ |
| 33 | Martin 的 Clean Architecture（2012 博客） | 源码依赖只指向内层，四圈只是示意 | ✅ |
| 34 | Fowler 的 Repository 模式 | 介于 domain 与 data mapping 之间 | ✅ |
| 35 | Evans《领域驱动设计》 | Repository 另一出处 | ⚠️ 仅经 Fowler 页间接确认 |
| 36 | Android 官方架构指南 | DataSource 术语来源 | ✅ |

## 五、依赖规则强制工具

| # | 工具 | 能否强制层级依赖 | 状态 |
|---|------|------------------|------|
| 37 | import_lint | 能（target、from、except）。2.0.0 起 Dart 3.10+/Flutter 3.38，analysis_server_plugin 机制（analysis_options.yaml 顶层 plugins:），与 riverpod_lint 同机制。首选 | ✅ 与 riverpod_lint 实际共存未验证 |
| 38 | import_guard_custom_lint | 能，但属旧 custom_lint 机制，与新版 riverpod_lint 不同路。仅旧 SDK 备选 | ✅ |
| 39 | custom_lint | 旧机制底层 | ✅ |
| 40 | dependency_validator | 不能，只查 pubspec，多包时补充 | ✅ |
| 41 | DCM | 能，avoid-banned-imports 支持按路径 deny 导入，可强制层级。商业工具 | ✅ |

## 六、常用包

| # | 包 | 说明 | 状态 |
|---|----|------|------|
| 42 | mocktail | 无需代码生成 | ✅ |
| 43 | shared_preferences 2.5.6 | 要求 Flutter 3.41/Dart 3.11。旧 SharedPreferences 为 legacy 将废弃，新代码用 SharedPreferencesAsync 或 SharedPreferencesWithCache；官方 Key-value 范例仍用旧 API | ✅ |
| 44 | dio | 网络层实现，官方未指定网络包，属项目自选 | ⚠️ 版本以 pub.dev 为准 |
| 45 | flutter_secure_storage 11.2.0 | 要求 Dart 3.8。Windows、Linux、Web 安全程度不同，细节未查 | ⚠️ |
| 46 | drift | 包存在（原 moor），未评估，不纳入 | ❌ |
| 47 | 日志方案（logger、talker） | 官方无强制日志方案，core/logging 属项目自定 | ❌ 暂不建 |

## 七、本项目自定

- core/ports + app/adapters 的目录名自定。机制有出处（FFCA 的 port 与 adapter、六边形架构）。
- app -> features -> shared -> core 链路：方向规则有 Clean 和 FFCA 出处（shared 不依赖 features、presentation 不依赖 data），目录名自定。
- core 禁依赖 Flutter：FFCA 的 shared 含 ui_kit、localizations 等 Flutter 包，不一致，属本项目收紧。注意与 core/storage 使用 Flutter 插件的矛盾，落地时二选一：收紧口径为"不依赖 Flutter UI"，或将存储实现上移。
- shared/ 防垃圾场规则：可借 FFCA 的"能否发 pub.dev"判据，具体条文自定。

## 八、引用时的注意点

1. Repository 接口放 domain 层：出处是 Fowler、Clean、FFCA。Flutter 和 Android 官方只要求抽象 Repository，没规定放哪层。本项目 data 层用具体类、不设接口，属有意简化，与官方 Strongly recommend 相左，如需多实现或便利测试再补抽象。
2. 官方没有 DTO 和 DataSource 这两个词，只有"API model 与 domain model 分离"。
3. UseCase 按需添加，符合官方态度。
4. Riverpod 做 ViewModel 的依据：Compass 案例写明 riverpod 等也可遵守这些规则。
5. 自动重试会影响 Repository 错误处理设计；SessionNotifier 恢复失败默认重试最多 10 次，需显式决定 retry 策略。
6. 实验性功能（Mutations、persistence）不进 core。
7. 单包项目不要宣称"遵循 FFCA"，只能说借其依赖规则，由 lint 约束。
8. SDK 约束要统一：riverpod_lint 新版（Dart 3.13）、import_lint（Dart 3.10）、shared_preferences 2.5.6（Dart 3.11）要求偏高，定版本前对照。
