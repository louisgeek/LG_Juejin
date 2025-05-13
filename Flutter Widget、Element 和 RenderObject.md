# Flutter Widget、Element 和 RenderObject
- Widget、Element 和 RenderObject 的三者通过分工（描述、管理和渲染）和协作，实现高效、灵活的 UI 渲染和更新机制

## Widget 组件
- Widget 用于声明式地（声明式 UI）描述界面元素的外观和行为，不直接处理渲染绘制
- Widget 是不可变的，无法被直接更新，当 Widget 的属性配置改变时，需要通过重建 Widget 来反映这种变化（每次保持在一帧，如果发生改变是通过 State 实现跨帧状态保存，而 State 保存在 Element 中）
- Widget 树由多个 Widget 嵌套组合形成，描述了 UI 的层级关系

## Element 元素
- Element 是 Widget 树在引擎中的实例化对象，是 UI 树的真实节点，负责管理 Widget 的生命周期和状态，并创建对应的 RenderObject，将 Widget 的配置参数（描述信息）转化为具体的渲染指令
- Element 协调 Widget 和 RenderObject 之间的绑定关系（可以理解为是一个粘合剂，Element 中持有 Widget 和 RenderObject），当 Widget 更新时，Element 会对比新旧 Widget 的 Key 和类型，决定是否复用 Element，避免重复创建 Element 和 RenderObject
- 编写的 Widget 和生成的 Element 是一对多的关系（比如 Container 设置 color 实际会生成 Container 对应的 Element 和 ColoredBox 对应的 Element），通过调用 Widget 的 createElement 方法生成对应的 Element 对象

## RenderObject 渲染对象
- RenderObject 是实际负责布局和绘制的对象，是渲染树中的节点，直接操作底层渲染引擎（比如 Skia）
- 一个 Element 不一定对应一个 RenderObject‌，并非所有 Element 都会创建 RenderObject（比如 StatelessWidget 和 StatefulWidget 都没有对应的 RenderObject），部分 Widget 的 Element 本身不直接创建 RenderObject，而是通过其子 Widget 的 Element 来创建 RenderObject

## 总结
- 因为 Widget 不涉及渲染，所以更新代价很小，而 RenderObject 负责 UI 渲染，所以更新代价极大，于是 Element 就负责将 Widget 树的变更以最小的代价映射到 RenderObject 树上，可以通过复用 Element 来减少频繁创建和销毁 RenderObject
- Widget 和 Element 是一对多的关系，Element 和 RenderObject 是一一对应的关系（除去 Element 不存在 RenderObject 的情况）