# Flutter Visibility 和 Offstage 的区别

## Visibility
```dart
//默认行为：隐藏，不占位，动画状态会被销毁，不保留 State
Visibility(
  visible: false,
  child: const MyWidget(),
)
//显式写出 replacement 配置
Visibility(
  visible: false,
  //replacement 默认就是 SizedBox.shrink()
  replacement: const SizedBox.shrink(),
  child: const MyWidget(),
)
//相当于就是 MyWidget 被默认的 SizedBox.shrink() 替换效果
//...
const SizedBox.shrink()
//...
//三元条件渲染（visible=false）
visible
    ? const MyWidget()
    : const SizedBox.shrink();
```

## Visibility 的关键参数
```dart
//隐藏，不占位，动画状态会被销毁，不保留 State
Visibility(
  visible: false,
  child: const MyWidget(),
)
//就是
Visibility(
  visible: false,
  maintainState: false,
  maintainAnimation: false,
  maintainSize: false,
  child: const MyWidget(),
)

//依赖关系（只有设为 true 时才会触发依赖关系）
//maintainSize == true
  //→ maintainAnimation 必须为 true
    //→ maintainState 必须为 true

//隐藏但占位，动画仍在后台运行，保留 State
maintainSize: true  maintainAnimation: true  maintainState: true
//隐藏，不占位，动画仍在后台运行，保留 State
maintainSize: false  maintainAnimation: true  maintainState: true
//隐藏，不占位， 动画 ticker 暂停，进度停在当前值，保留 State
maintainSize: false  maintainAnimation: false  maintainState: true
```


## Offstage 幕后
```dart
//隐藏，不占位，动画仍在后台运行，保留 State
Offstage(
  offstage: true,
  child: const MyWidget(),
)
//近似于（该状态下构建时底层就是返回了 Offstage）
Visibility(
  visible: false,
  maintainState: true,
  maintainAnimation: true,
  maintainSize: false,
  child: const MyWidget(),
)
```

## 总结
- 完全移除组件（不保留状态）：如骨架屏、空状态和列表切换；用 Visibility 的默认 参数或三元表达式，隐藏、不占位、State 销毁
- 保留状态但暂停动画（节省资源）：如临时隐藏的加载动画、轮播图、卡片动画或倒计时动画；用 `maintainSize: false, maintainAnimation: false, maintainState: true`，隐藏后 ticker 暂停，保留 State
- 临时隐藏但保留状态：如 Tab 页面切换、多步骤表单前后切换、收起再展开的筛选面板；用 `maintainSize: false, maintainAnimation: true, maintainState: true` 或 `Offstage(offstage: true)`，隐藏、不占位、保留 State
- 隐藏但保留布局位置（避免 UI 跳动）：如表单错误提示、按钮操作区；用 `maintainSize: true, maintainAnimation: true, maintainState: true`，隐藏但仍占位