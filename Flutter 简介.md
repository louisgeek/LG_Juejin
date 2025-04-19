# Flutter 简介
- Apache Cordova：使用 HTML、CSS 和 JavaScript 开发，通过 WebView 进行渲染（性能相对较弱），利用插件调用原生功能
- Facebook ReactNative：使用 JavaScript 和 React 框架开发，通过原生控件渲染（通过 JavaScript Bridge 将组件映射为原生控件，比如 View 对应 ViewGroup，Image 对应 ImageView 等）
- Google Flutter：使用 Dart 开发，通过自绘引擎 Skia 进行渲染（目前 Chrome 和 Android 均采用 Skia 作为 2D 绘图引擎），避免了桥接开销，可以保证 Android、IOS 等平台上 UI 的一致性

## 跨平台
- 通过共享同一份代码，可以同时为多个平台构建应用，一次编写，到处运行，可以大大减少开发和维护的成本
- 通过自绘引擎进行渲染，规避原生控件差异，确保各平台 UI 一致性

## 高性能
- 使用高性能自绘引擎 Skia 来绘制 Widget 组件，Skia 可以直接与底层操作系统的图形 API 进行交互（比如 OpenGL），意味着应用程序的性能更高，响应更快，尤其在滑动、过渡效果和动画场景表现突出，，规避原生控件差异
- 采用 Dart 语言开发，Dart 在 JIT 即时编译（Just-In-Time）模式下，执行速度与 JavaScript 基本持平，而 Dart 支持 AOT 提前编译（Ahead-Of-Time），当在 AOT 模式下 JavaScript 就远远追不上了，从而提高应用的启动速度和运行性能
- Flutter 在开发阶段采用 JIT 模式，可以避免了每次改动都要进行编译，极大地节省了开发时间
- Flutter 在发布时采用 AOT 模式生成高效的本地机器码以保证应用性能

## 类型安全和空安全
- Dart 是强类型的语言（JavaScript 是弱类型语言），支持静态类型检测保证类型安全
- Dart 空安全特性在编译期检查变量是否可能为 null，从而减少运行时错误

## 热重载
- 支持 Hot Reload 功能，支持实时查看代码修改效果，提高开发效率（无需等待重新编译）

## 灵活的 UI 设计理念
- Flutter 采用 “一切皆是 Widget” 的设计理念，UI 元素（包括布局、动画、交互）均以 Widget 形式描述，可以通过自由组合和嵌套实现复杂动效和个性化界面
- Flutter 的响应式编程设计（声明式 UI）允许开发者专注于状态与 UI 结构的映射，框架通过组件状态自动处理 UI 更新，简化复杂逻辑的开发，使得 UI 代码更加直观和易于维护