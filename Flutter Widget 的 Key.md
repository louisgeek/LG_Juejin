# Flutter Widget 的 Key
- Key 作为每个 Widget ​的唯一标识符，帮助框架在 Widget 树重建时区分相同类型的组件（比如在动态视图中，若未指定 Key，交换两个同类 Widget 的位置可能会导致状态错乱，因为框架比较 runtimeType 一致后认为是可复用的不需要刷新，所以状态错乱了）
- 避免状态丢失：当 Widget 的位置或层级发生变化时，通过 Key 可以确保其关联的 State 对象被正确复用（比如输入框需通过 Key 保留输入内容，避免因重建导致状态丢失）
- ​渲染性能​优化：在 Widget 树重建时，能够通过 Key 匹配新旧 Widget 来决定是复用现有 Element（从而复用 RenderObject），避免不必要的重建，减少性能开销
- 避免滥用 Key：通常情况下，框架会自动管理 Key，仅在动态列表、状态保持等场景下才需要显式使用 Key
- Widget 的 Key 主要分为 GlobalKey 和 LocalKey 两大类

```dart
//Widget#canUpdate
//类型相同且 Key 相同：复用旧的 Element（仅更新数据）
//类型相同但 Key 不同：销毁旧的 Element，创建新 Element
//类型不同：直接创建新的 Element（无论 Key 是否相同）
static bool canUpdate(Widget oldWidget, Widget newWidget) {
    //若未指定 Key，仅通过类型匹配，可能会导致状态错乱
  return oldWidget.runtimeType == newWidget.runtimeType 
      && oldWidget.key == newWidget.key;
}
```

## GlobalKey 全局唯一
- 在整个应用中唯一标识一个 Widget
- 通过 GlobalKey 可以实现跨 Widget 树访问目标 Widget 的信息（比如 Widget、State 和 Context），可以实现复杂的交互逻辑
```dart
_globalKey.currentWidget
_globalKey.currentState
_globalKey.currentContext
```

## LocalKey 本地唯一
- 在同一父节点下的同类型 Widget 中唯一
- LocalKey 又分为 ValueKey、ObjectKey 和 UniqueKey
- ValueKey​​：基于值（比如数字、字符串等）标识 Widget，适合动态列表项（比如使用 ValueKey 基于 item.id，可确保列表顺序变化时，列表项的输入框内容能正确保留）
- ​​ObjectKey​​：基于对象引用标识 Widget，适用于需唯一性但值可能重复的对象（ValueKey​ 先比较类型再比较值是否相等，而 ​​ObjectKey​​ 先比较类型再比较内存地址是否一致）
- ​​UniqueKey​​：每次刷新都生成唯一值，强制 Widget 重建，所以过度使用会导致性能问题

## 总结
- Widget 的 Key 是一个唯一性标识符，有助于在 Widget 树发生变化时正确地识别、追踪和管理 Widget（防止状态错乱）
- 通过 Key 可以优化状态管理，防止状态丢失
- 通过 Key 可以精确控制 Widget 的复用逻辑，避免不必要的重建，实现 Widget 树的更新的性能​优化
- 谨慎使用 GlobalKey，GlobalKey 会增加性能开销，仅在必要时使用（比如跨 Widget 访问状态）
- 列表项必须使用 Key（比如 ValueKey 或 ObjectKey），尤其是动态列表，否则位置变化时可能导致状态错位（比如输入框内容错误关联）
- UniqueKey 适用于动态生成的 Widget（比如拖拽排序列表），强制销毁和重建

