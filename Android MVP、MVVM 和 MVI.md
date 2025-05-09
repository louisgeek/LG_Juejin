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
- MVI 引入单向数据流、状态不可变概念确保状态更新的可预测性
- MVI 单向数据流：View -> UIIntent 封装用户操作 -> ViewModel 处理逻辑 -> 变更 UIState -> View 更新，MVI 强制所有状态变更通过 UIIntent 触发，使得状态管理更加清晰和可控
- MVI 不可变状态：每次状态变化会创建新的对象，避免多线程竞争问题
- MVI 单一可信数据源：所有状态集中在 ViewModel，便于管理和调试

## 总结
- MVVM 侧重双向数据绑定（View <-> ViewModel），双向绑定通过 DataBinding 等实现，不过大部分开发者都会使用 LiveData、Flow 来观察数据变化，通常为了保证数据流的单向流动性（非必须，不强制），会向外暴露不可变的 LiveData，所以 ViewModel 里一个 State 状态会存在两个 LiveData，一个可变的一个不可变的，如果状态很多，LiveData 数量就会大大地增多
- MVI 是 MVVM 的升级方案，侧重单向数据流（View -> UIIntent -> ViewModel -> UIState -> View）约束，强调不可变状态（State 实例是不可变的，每次状态更新时都会创建新的 State 对象）和单一（唯一可信）数据源，MVI 通过将状态变化和用户意图分离，确保状态的更新的可预测性，MVI 里的 Model 侧重指 UIState 状态，UIState 用于封装页面状态和数据（比如页面加载状态、控件位置等），对 State 进行了集中状态管理（这样一来 LiveData 就只需要一份了，一个可变的一个不可变的），UIIntent 用于包装用户的操作意向（意图）发送给 ViewModel 进行数据请求，另外可以额外引入 UIEvent 来处理一次性的事件
- UIState、UIIntent 通常都可以采用密闭类（密封类）
- Activity/Fragment 里观察 UiState 状态，然后通过 when 判断处理对应逻辑
- ViewModel 里可以提供集中统一处理 UIIntent 的入口方法
- MVI 中所有 UIState 状态都可以通过一个 LiveData 来管理，也随之而来引出新的问题，就是页面不支持局部刷新
- 由于 UIState 状态是不变的，因此每当 UIState 需要更新时都要创建新对象替代老对象，这会带来一定内存开销