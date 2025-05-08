# MVP、MVVM 和 MVI 分别解决了什么痛点

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
- MVI 单向数据流：Intent 用户交互 -> ViewModel 处理 -> 更新 UIState -> View 根据 State 渲染！！！  用户操作（Intent）触发状态更新，模型（Model）存储不可变状态，视图（View）根据状态渲染  Intent → Model → State → View
- MVI 不可变状态：每次状态变化生成新对象，避免多线程竞争问题
- MVI 单一可信数据源：所有状态集中在 ViewModel，便于管理和调试
- MVI通过将状态变化和用户意图分离，确保状态的更新是单向的和可预测的。用户操作被转换为Intent，Intent通过处理逻辑生成新的状态，然后状态被传递到视图进行更新。这种单向数据流使得状态管理更加清晰和可控    MVI强制所有状态变更通过Intent触发，由Model生成不可变的新状态，避免副作用