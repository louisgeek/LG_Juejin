# Android MVP、MVVM 和 MVI

## MVP
- MVC 中 Controller 过于臃肿：Controller（Activity/Fragment）承担了过多的逻辑，导致代码难以维护
- MVC 中 View 和 Model 耦合度高：View 直接依赖 Model（View 直接操作 Model，Model 直接更新 View）
- MVC 中单元测试比较困难： UI 逻辑与业务逻辑耦合，难以进行单元测试
- MVP 通过引入 Presenter 作为中间层解耦 View 和 Model（Presenter 负责协调 View 和 Model 的交互，仅通过接口与 View 通信），将业务逻辑从 Activity/Fragment 中分离出来
- MVP 通过接口隔离，使 Presenter 可独立于 UI 进行单元测试，提升开发效率

## MVVM
- MVP 中 View 的接口数量庞大
- MVP 中手动更新 UI 数据比较繁琐
- MVVM 引入 ViewModel 层，通过 DataBinding 等实现 View 与 ViewModel 的双向数据绑定，实现 Model 数据变化自动同步到 View（数据驱动 UI）
- MVVM 通过 ViewModel 持有数据，避免 Activity/Fragment 旋转时数据丢失

## MVI
- MVVM 中双向绑定可能导致状态不可控（状态管理混乱、可预测性差）
- MVI 是一种基于响应式编程的架构模式，引入单向数据流、不可变状态和单一可信数据源等概念确保状态更新的可预测性
- Model 不仅是数据模型，更代表应用的 State 状态（比如加载中、数据成功、错误等），通常是不可变对象
- View 负责展示 UI 并将用户操作（比如按钮点击、输入等）转换为 Intent 意图发送给 ViewModel
- Intent 用户操作的抽象表示（比如 LoadDataIntent 加载数据和提交表单等），是 View 到 ViewModel 的唯一数据输入渠道
- ViewModel 接收 Intent 意图，处理业务逻辑（比如调用 Repository 仓库层进行网络请求或数据库操作等），将结果转换为新的 State 状态，最终通过数据流暴露给 View
- 单向数据流：View -> UIIntent 封装用户操作 -> ViewModel 处理 UIIntent 逻辑 -> 更新 UIState 状态变化 -> 驱动 View 渲染刷新，MVI 架构强制所有状态变更都通过 UIIntent 触发，使得状态管理更加清晰和可控，降低调试难度
- 不可变状态：每次状态变化会创建新的对象，避免多线程竞争问题，减少并发问题
- 单一（唯一）可信数据源：所有状态集中在一个 UIState 里通过 ViewModel 进行管理（比如避免多个 LiveData 造成状态分散），统一管理 UI 状态，简化状态管理，便于追踪和调试（不过复杂页面建议拆分 State，避免单个 State 过于庞大）

## 总结
- MVVM 侧重双向数据绑定（View <-> ViewModel），双向绑定通过 DataBinding 等实现，不过大部分开发者都会使用 LiveData、Flow 来观察数据变化，通常为了保证数据流的单向流动性（非必须，不强制），会向外暴露不可变的 LiveData，所以 ViewModel 里一个 State 状态会存在两个 LiveData，一个可变的一个不可变的，如果状态很多，LiveData 数量就会大大地增多
- MVI 是 MVVM 的升级方案，侧重单向数据流（View -> UIIntent -> ViewModel -> UIState -> View）约束，强调不可变状态（State 实例是不可变的，每次状态更新时都会创建新的 State 对象）和单一（唯一）可信数据源，MVI 通过将状态变化和用户意图分离，确保状态的更新的可预测性，MVI 里的 Model 侧重指 UIState 状态，UIState 用于封装页面状态和数据（比如页面加载状态、控件位置等），对 State 进行了集中状态管理（这样一来 LiveData 就只需要一份了，一个可变的一个不可变的），UIIntent 用于包装用户的操作意向（意图）发送给 ViewModel 进行数据请求，另外可以额外引入 UIEvent 来处理一次性的事件
- UIState​ 不可变对象，封装 UI 所有状态，作为唯一数据源，通常可以采用密闭类（密封类），也可以是数据类（设计使用 val，可以避免 View 直接修改状态）
- UIEvent 处理一次性操作（比如 Toast、Snackbar），避免状态残留，通常可以采用密闭类（密封类）
- UIIntent 用户交互事件，通过传递 UiIntent 至 ViewModel，通常可以采用密闭类（密封类）
- ViewModel 通过 LiveData 暴露 UIState 给 UI 观察订阅，ViewModel 里可以提供集中统一处理 UIIntent 的入口方法
- Activity/Fragment 里观察 UiState 状态，然后通过 when 判断处理对应的更新 UI 逻辑（比如展示数据、显示错误提示）
- MVI 中所有 UIState 状态都可以通过一个 LiveData 来管理，也随之而来引出新的问题，就是无法直接支持局部刷新
- 由于 UIState 状态是不变的，因此每当 UIState 需要更新时都要创建新对象替代老对象，这会带来一定内存开销