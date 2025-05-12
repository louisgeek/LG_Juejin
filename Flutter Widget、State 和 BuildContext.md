# Flutter Widget、State 和 BuildContext
- Widget 是描述 UI 界面的不可变配置，定义了 UI 界面的结构、布局、样式和交互逻辑，分为 StatelessWidget 无状态的静态组件和 StatefulWidget 需要动态更新的组件
- State 是 StatefulWidget 中用于存储和管理组件的可变状态，每个 StatefulWidget 对应一个 State（State 的生命周期长于 Widget，Widget 可能因重建被销毁，但 State 会保留）
- BuildContext 是 Widget 在树中的位置标识，是用于定位当前 Widget 在 Widget 树中位置的上下文对象，提供了访问父级或祖先 Widget 和全局资源访问的能力

## Widget 组件
- Widget 一旦创建，属性不能修改，需通过重建来更新 UI（每个 Widget 仅代表 UI 某一时刻的一个快照）
- Widget 树由多个 Widget 嵌套组合形成，描述了 UI 的层级关系
- 通过声明式构建 UI，驱动界面渲染

## State 状态
- StatefulWidget 通过 createState 创建一个 State 对象
- State 管理可变状态，通过 setState 触发 UI 重建
- StatefulWidget 本身是不可变的，而 State 是可变的

## BuildContext
- BuildContext 是抽象类，Element 实现了 BuildContext（Element 是 Widget 树在引擎中的实例化对象，是 UI 树的真实节点）
- 每个 Widget 的 build 方法接收父级传递的 Context，形成树状结构
- 通过 context 可向上访问父级或祖先 Widget（比如 context.findAncestorWidgetOfExactType），也可向下传递数据（比如通过 InheritedWidget、Provider）
- 通过 Theme.of(context) 获取当前主题，通过 Navigator.of(context) 获取导航器

## 总结
- Widget：描述 UI 的配置（不可变），每个 Widget 只能通过生成新实例来更新
- State：存储 UI 的状态（可变），StatefulWidget 依赖 State 对象管理状态
- BuildContext：提供上下文环境与资源访问，提供访问当前 Widget 树信息和全局资源的能力
