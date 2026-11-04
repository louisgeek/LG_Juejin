# Flutter 工程实践清单

> 最后核对：2026-11。本文是通用工程清单，与具体项目决策无关、最易过期，升级 Flutter / 依赖后需重扫。
> 来源后括注为社区经验等非官方内容；社区条目为公开资料综述，未逐条附链接与官方核实，版本相关结论以 pub.dev / 官方文档当日为准。
> 架构分层与选型依据见同目录 [Flutter 架构理论参考](Flutter%20架构理论参考.md)。

## 一、测试策略

| # | 来源 | 要点 |
|---|------|------|
| 1 | Flutter 官方 testing/overview | 三类测试置信度、维护成本、依赖数量、速度逐级递进；unit/widget 用 flutter test 跑在开发机，integration 跑真机或模拟器。integration_test 无法操作原生 UI（权限弹窗、通知、platform view），需要原生交互时用 patrol 扩展。 |
| 2 | 架构 case study 的 Testing each layer 页 | 可测试性是架构是否健康的判据：View / ViewModel 测试只需 mock/fake Repository，测 Repository 时 mock 其依赖的 Service；ViewModel 用不依赖 Flutter 框架的纯 Dart 单测。分层正确的项目不需要为测试额外改造。 |
| 3 | 社区测试金字塔（社区经验比例，非官方指标） | unit 60-80%：业务规则、校验、mapper、UseCase；widget 15-30%：加载/错误/空态、导航、表单交互；golden 5-15%：视觉回归；integration 1-5%：只保登录、支付、深链、推送等收入关键路径与原生插件边界，不是每个页面都测。 |
| 4 | Golden 测试实践（社区经验） | 只用于设计系统组件与稳定页面。必须固定：打包测试字体、surface 尺寸、pixel ratio、locale、主题、输入 fixture；CI 单一 OS 并锁定 Flutter 版本。含地图、视频、实时数据等动态内容的页面不做 golden，否则像素 diff 导致 flaky。 |
| 5 | CI 分层跑测试（社区经验） | PR 跑 analyze + unit + widget + golden（目标 <10 分钟），integration 放 nightly 与 release 分支。交互元素用 find.byKey 定位，共享 pumpApp()/fixture 辅助，每个测试前重置本地存储；flaky 测试立即修，覆盖率作参考不追数字。 |

## 二、性能

| # | 来源 | 要点 |
|---|------|------|
| 6 | 官方 perf/best-practices | 控制 build() 成本：不在 build 里做重活；超大 widget 按变化频率拆分；setState 局部化到真正变化的子树；复用 UI 用 StatelessWidget 而非返回 Widget 的函数；长列表用 ListView.builder；动画避免 Opacity/clipping；警惕隐式 saveLayer；不要给 Widget 重写 ==（导致 O(N²) 退化）。60Hz 一帧 16ms，120Hz 8ms。 |
| 7 | 官方 Impeller 页、rendering-performance 页 | Impeller 在构建期预编译 shader，解决首次运行动画的 jank，Android API 29+ 默认启用，低版本或不支持 Vulkan 回退 OpenGL。性能测量一律用 profile mode（debug 数据无参考价值），用 DevTools 查看 rebuild 范围与帧耗时。 |
| 8 | Riverpod 侧性能配合（社区经验） | ref.watch(provider.select(...)) 只订阅需要的字段，缩小 rebuild 范围；页面级 provider 用 autoDispose 及时释放；全局常驻只留 session、主题等少数；长列表把状态下沉到 item 级 provider，避免整页重建。 |

## 三、安全

| # | 来源 | 要点 |
|---|------|------|
| 9 | OWASP MASVS、Mobile Security Cheat Sheet | 用 MASVS 八组控制项（存储/加密/认证/网络/平台/代码/韧性/隐私）当发布前清单。全量 HTTPS、禁止为自签证书关闭校验；证书 pinning 属可选项，一旦启用必须内置备用 pin 并设计轮换流程，否则证书轮换即把老版本用户锁在门外。 |
| 10 | 官方 obfuscate 页 | flutter build --obfuscate --split-debug-info=<目录>，逐版本备份符号表，崩溃堆栈用 flutter symbolize 还原；仅 release 构建有效，web 用 minify 替代。混淆只重命名符号，不加密资源、不防逆向；运行时依赖类名/函数名字符串的代码会因此失效，需自查。 |
| 11 | OWASP MASTG-TECH-0109（Flutter 专项） | Dart 使用自己的证书库且不感知系统代理，系统安装的证书被忽略：常规系统代理抓包直接失效，需要额外手段；但 hook/patch 仍可绕过，所以这只是 Flutter 的特殊性，不构成安全边界。 |
| 12 | flutter_secure_storage 文档 + Android 备份文档（算法默认值随大版本变化，以所装版本 README 为准） | iOS/macOS 走 Keychain；Android 自 v10 起默认 RSA OAEP + AES-GCM（11.2.0 核实时，minSdk 23），更早版本默认算法不同。iOS 可访问性级别需在"后台可读"与"最严格"间权衡。Android Auto Backup 默认包含 SharedPreferences 与内部文件，使用安全存储时应评估 allowBackup="false" 或配置排除规则，避免备份恢复导致密钥失效或数据泄露。 |

## 四、可观测性（日志与崩溃监控）

| # | 来源 | 要点 |
|---|------|------|
| 13 | Firebase Crashlytics Flutter 官方指南 | bootstrap 中 FlutterError.onError = recordFlutterFatalError 接框架错误；Zone / PlatformDispatcher 捕获的异常用 recordError(fatal: true)；非致命错误用 recordError 上报；可用 custom keys 附 flavor、环境、用户散列等维度。 |
| 14 | Sentry vs Crashlytics（社区对比，2026） | 已在 Firebase 生态、只要崩溃上报选 Crashlytics（接入最快）；需要错误 + 性能 trace + release 追踪一体、更强的 breadcrumb、问题分组与告警工作流选 Sentry；高事件量下注意 Sentry 成本。 |
| 15 | 结构化日志实践（社区经验） | 全工程统一 AppLogger 接口，实现可换 logger / talker；按 flavor 控制级别（dev=debug，prod=warn/error）；输出前脱敏 Authorization、令牌、PII、请求体；error 级日志转发崩溃平台，并附 requestId、endpoint、statusCode、durationMs、appVersion 等关联字段。talker 自带 Dio / Riverpod / Bloc 集成与 observer 转发钩子。 |

## 五、本地存储选型

| # | 来源 | 要点 |
|---|------|------|
| 16 | 选型分界（社区共识） | 设置/标志位用 shared_preferences（Async / WithCache），令牌用 flutter_secure_storage，两者都不是数据库；只有需要过滤、排序、关联、聚合、schema 迁移的结构化数据才上数据库，别把数据库当万能缓存。 |
| 17 | drift（社区主流默认，非官方推荐） | 类型安全查询（Dart 或 SQL，构建期校验）；任意查询可转 Stream，直接接 StreamBuilder / Riverpod；schemaVersion + 迁移步骤与迁移测试；web 支持稳定；基于 sqlite3（FFI），与 sqflite 无封装关系。注意：**isolate 支持需显式启用**——默认在调用方 isolate 同步执行 SQLite，会阻塞调用方；用 NativeDatabase.createInBackground 或 DriftIsolate 显式启用后，重查询才不占 UI isolate。 |
| 18 | sqflite | 官方 cookbook 持久化范例所用：手写 SQL、自行映射 row、onCreate/onUpgrade 管迁移；支持 Android/iOS/macOS，桌面走 sqflite_common_ffi，web 为独立实验包。表少、不想引入代码生成、团队熟 SQL 时够用。 |
| 19 | hive / isar 维护状态（社区观察） | 原 hive 停更于 2022，简单 box 需求用社区分支 hive_ce（含 web/WASM、isolate 支持）；isar 3 停更，isar_community 以 v3 修 bug 为主（无 web），isar_plus 较新但采用度低。新项目不引入原版；存量迁移前用真实用户数据验证兼容性。 |

## 六、工程化（DI、monorepo、CI/CD）

| # | 来源 | 要点 |
|---|------|------|
| 20 | DI 选型（社区共识；官方仅 Strongly recommend DI 本身，示例用 provider） | 已用 Riverpod 做状态就同时用它做 DI：provider 即依赖图，组合根装配，类内部一律构造函数注入，测试用 override 替换，避免两套心智模型。get_it 适合不用 Riverpod 或有大量非 UI 代码的场景：默认 registerLazySingleton（避免启动时构造拖慢冷启动），getIt() 只允许在 main、路由构建等组合点调用。底线：业务类应能脱离任何框架直接 new 出来跑纯 Dart 测试。 |
| 21 | melos 官方文档 + dart.dev workspaces | 两者互补：pub workspaces 管统一依赖解析与本地链接（3.6+）；melos 管跨包脚本编排、按变更过滤执行（只测改动的包）、版本与 changelog、发布。melos 新版直接读 workspace 配置，bootstrap 不再必需但可挂共享依赖与钩子。单包项目都不需要。 |
| 22 | 官方 deployment/cd 页（流水线分工为社区经验） | 官方提供 fastlane 与 GitHub Actions / Codemagic 等的集成指南。常见分工：GitHub Actions 在 Linux 跑 PR 快检（analyze + 测试，<10 分钟）；iOS 构建用 macOS runner 或 Codemagic；fastlane（match 管证书）负责签名与商店发布。构建号必须唯一（可用 CI run number），main 分支保护 + 发布人工审批 + 分阶段放量。 |
| 23 | Shorebird code push（商业服务，上架合规需自行评估） | Dart 代码热更新：release 与 patch 分离，patch 必须用 --release-version 对齐已发布版本，可经 fastlane 插件接入现有流水线；只覆盖 Dart 层变更，原生代码改动不能热更。App Store / Play 对热更的政策限制需自行评估后再采用。 |
