# Flutter 架构

> Flutter 架构理论参考.md


## 目录结构
Widget/Page → ViewModel（Notifier） → UseCase（可选） → Repository → DataSource → Dio/MQTT/存储/Channel

```
lib/

├── main.dart                          # 应用入口:初始化 Flutter 环境(WidgetsFlutterBinding),然后调用 bootstrap
│
├── app/                               # 组合根:可依赖所有层,唯一同时认识 features 和 core 的地方
│   ├── app.dart                       # 根组件 MaterialApp.router:挂载路由、主题、多语言，只放根 Widget（MaterialApp.router + 主题/多语言/路由），与初始化解耦
│   ├── bootstrap.dart                 # 启动流程，放异步初始化（日志、错误捕获、存储、DI/ProviderScope），返回结果对象
│   │                                  #   1. 初始化日志(包由项目自选)
│   │                                  #   2. 注册全局错误捕获:FlutterError.onError、PlatformDispatcher.onError
│   │                                  #   3. 提前创建存储实例(SecureStorage、SharedPreferences)
│   │                                  #   4. 通过 ProviderScope.overrides 注入这些实例
│   │                                  #   不读取令牌(交给 SessionNotifier 异步恢复)
│   │                                  #   baseUrl 等环境值暂用 --dart-define,仅到 Dart 侧
│   ├── router/
│   │   ├── app_router.dart            # GoRouter provider、StatefulShellRoute、登录态 redirect
│   │   ├── router_refresh.dart        # 把 session 变化转成 refreshListenable,触发重新 redirect
│   │   └── deeplink.dart              # 外部链接 → 内部路由
│   ├── shell/
│   │   ├── splash_page.dart           # session 为 Unknown(恢复中)时显示,此时不做任何跳转
│   │   ├── main_shell_page.dart       # 底部导航栏 + 内容区容器
│   │   └── main_tab.dart              # Tab 枚举:图标、标题、路径
│   └── theme/
│       ├── app_theme.dart             # 亮/暗主题:配色、字体、组件样式
│       └── theme_mode_notifier.dart   # 主题模式(跟随系统/亮/暗),全局常驻,不 autoDispose
│
├── features/                          # 按"用户做什么"划分;feature 之间不互相 import
│   └── auth/                          # 登录与注册
│       ├── domain/                    # 纯 Dart;不依赖本 feature 的 data 和 presentation
│       │   ├── user.dart              # 业务模型:用户信息
│       │   └── validators.dart        # 表单校验:邮箱格式、密码长度、两次密码一致
│       │                              #   只返回错误枚举,文案由界面决定
│       ├── data/
│       │   ├── dto/                   # json_serializable;DTO 不出 data 层
│       │   │   ├── auth_dto.dart      # LoginRequest、RegisterRequest:发给后端,只生成 toJson
│       │   │   │                      #   AuthDto:后端返回(令牌 + UserDto),只生成 fromJson
│       │   │   └── user_dto.dart      # UserDto:后端返回的用户数据,只生成 fromJson
│       │   ├── sources/
│       │   │   └── auth_remote_source.dart  # 登录、注册接口,只做 Dio 调用,返回 DTO,不处理业务
│       │   ├── mappers/
│       │   │   └── user_mapper.dart   # extension UserDtoX on UserDto { User toDomain() }
│       │   ├── auth_repository.dart   # 抽象 AuthRepository + AuthRepositoryImpl(同文件;遵循官方 Strongly recommend 抽象);不依赖、不通知 session
│       │   │                          #   登录:构造 LoginRequest → 调 source → 存令牌 → 返回 Result<User>
│       │   │                          #   注册:构造 RegisterRequest → 只提交 → 返回 Result<void>
│       │   │                          #   退出:清令牌 → 返回 Result<void>
│       │   │                          #   错误:拦截器已把 AppError 放进 DioException.error,
│       │   │                          #         此处取出并包成 Failure,不再二次转换
│       │   └── auth_providers.dart    # authRepositoryProvider 声明为抽象类型,构造 AuthRepositoryImpl(面向接口装配)
│       └── presentation/              # 可 import data 里的 auth_repository 和 auth_providers
│           │                          #   不得 import dto、sources、mappers
│           ├── login/                 # 登录页,自成一个目录
│           │   ├── login_page.dart    # 登录页界面
│           │   ├── login_notifier.dart  # autoDispose,页面离开即销毁
│           │   │                        #   提交邮箱、密码给 repository
│           │   │                        #   await 之后用 ref.mounted 判断是否已销毁
│           │   │                        #   成功后调用 SessionNotifier.signedIn()(全工程唯一调用点)
│           │   ├── login_state.dart   # freezed:表单字段、是否显示密码、提交状态
│           │   │                      #   密码不进 toString(可能需手写覆盖,需实测)
│           ├── register/              # 注册页,自成一个目录
│           │   ├── register_page.dart # 注册页界面;ref.listen 监听成功后跳转登录页
│           │   ├── register_notifier.dart  # autoDispose;只管提交状态,不碰登录态
│           │   ├── register_state.dart     # freezed:注册界面状态
│           └── widgets/               # 登录、注册两页共用的组件(输入框、提交按钮、协议勾选)
│                                      #   没有共用就不建;login/ 和 register/ 互相不 import
│
├── shared/                            # 跨 feature 复用,可依赖 Flutter;只依赖 core 和外部包,不依赖 features、app
│   ├── l10n/
│   │   ├── arb/                       # app_zh.arb、app_en.arb(l10n.yaml 需显式配置 arb 目录和输出位置)
│   │   ├── gen/                       # gen-l10n 生成的代码,不手改
│   │   ├── locale_providers.dart      # 当前语言、切换语言
│   │   ├── locale_storage.dart        # 保存用户选择的语言(依赖 core/storage)
│   │   ├── l10n_extensions.dart       # extension on BuildContext:context.l10n
│   │   └── app_error_message.dart     # extension on AppError:转成用户可读文案
│   ├── mixins/                        # 带状态/生命周期的通用 mixin,与业务无关;用到再建
│   │   ├── debounce_mixin.dart        # on State<T>:防抖/节流,dispose 时取消 Timer
│   │   ├── submit_guard_mixin.dart    # 防重复提交
│   │   └── focus_mixin.dart           # on State<T>:FocusNode 创建与释放
│   ├── widgets/                       # 跨 feature 通用组件;用到再建
│   └── utils/
│       └── context_extensions.dart    # 屏幕尺寸、主题快捷访问等(依赖 BuildContext,所以不放 core)
│
└── core/                              # 跨功能基础设施;不依赖 UI 库(widgets、material),不依赖 shared、features、app
    ├── network/
    │   ├── dio_factory.dart           # 构建 Dio:baseUrl、超时、默认头
    │   ├── network_providers.dart     # dioProvider;按顺序挂载拦截器;注入 onUnauthorized 回调
    │   └── interceptors/              # 挂载顺序:logging → auth → unauthorized → error(需测试验证顺序)
    │       ├── logging_interceptor.dart       # 仅 debug 打印,脱敏 Authorization 和密码
    │       ├── auth_interceptor.dart          # 注入令牌
    │       ├── unauthorized_interceptor.dart  # 识别 401,调用回调;不直接依赖 session
    │       └── error_interceptor.dart         # 最后一环:DioException → AppError,放入 DioException.error
    ├── storage/
    │   ├── storage_providers.dart     # SecureStorage、SharedPreferences 的 provider,实例由 bootstrap 注入
    │   └── token_storage.dart         # 令牌读写(flutter_secure_storage),拦截器、session、repository 共用
    ├── session/
    │   ├── session_state.dart         # sealed:Unknown(恢复中)/ Authenticated / Unauthenticated
    │   └── session_notifier.dart      # 异步恢复令牌;提供 signedIn() / signedOut()
    │                                  #   只存登录态,不持有 User,不放登录页和注册页
    │                                  #   注意 Riverpod 3 的自动重试(retry),恢复失败时要决定是否关闭
    ├── navigation/
    │   └── routes.dart                # 路径常量(含 login、register),全工程唯一一份
    ├── error/
    │   ├── app_error.dart             # sealed:Network / Timeout / Unauthorized / Server / Unknown
    │   └── result.dart                # sealed Result<T>:Success / Failure;用模式匹配即可
    ├── cache/                         # 按需:没有具体需求不要建
    └── utils/                         # 纯 Dart 工具,无状态,克制使用
        ├── string_extensions.dart     # 字符串判空、截断等
        └── datetime_extensions.dart   # 日期格式化等
```