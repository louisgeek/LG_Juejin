## Fragment 生命周期
> 官方文档 https://developer.android.google.cn/guide/fragments/lifecycle

## 核心回调
- 11 个
```java
onAttach —— onCreate —— onCreateView —— onViewCreated —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestroyView —— onDestroy —— onDetach
```

![Fragment 生命周期](https://android-docs.cn/static/images/guide/fragments/fragment-view-lifecycle.png "https://android-docs.cn/static/images/guide/fragments/fragment-view-lifecycle.png")


## 经典场景
1 第一次启动 Fragment A
```java
//Fragment A
onAttach -> onCreate -> onCreateView -> onViewCreated -> onStart -> onResume
```

2 在 Fragment A 上启动 Fragment B（replace 替换的方式）
```java
//Fragment A（启用 addToBackStack 返回栈，不会继续执行 onDestroy 和 onDetach，实例保留在内存中）
onPause -> onStop -> onDestroyView
//Fragment B
onAttach -> onCreate -> onCreateView -> onViewCreated -> onStart -> onResume
```

3 从 Fragment B 返回到 Fragment A
```java
//Fragment B
onPause -> onStop -> onDestroyView -> onDestroy -> onDetach
//Fragment A
onCreateView -> onViewCreated -> onStart -> onResume
```
4 点击 HOME 键、来电
```java
//点击 Home 键，来电
onPause -> onStop
//回到 App
onStart -> onResume
```

5 锁屏与解锁
```java
//锁屏
onPause -> onStop
//解锁
onStart -> onResume
```


## 生命周期推荐做的事
```java
onAttach 与 Activity 首次关联时调用（仅执行一次），可以获取 Context
onCreate 当 Fragment 创建调用时（仅执行一次），可以初始化非视图相关的数据（比如初始化数据集合等）
onCreateView 创建视图时（会被多次调用），需要创建并返回 Fragment 的视图，避免耗时操作
onViewCreated 视图创建完成后（会被多次调用），初始化控件逻辑（比如设置点击事件等）
//
onStart 变为可见时，适合启动后台任务或注册监听器
onResume 有焦点可交互时，恢复 UI 交互（比如开始动画、视频的播放）
onPause 失去焦点时，保存临时数据，释放资源（比如停止动画、视频的播放）
onStop 完全不可见时，释放更多资源或取消注册监听器
//
onDestroyView 视图销毁时，释放视图相关资源
onDestroy 当 Fragment 被销毁时，释放所有资源
onDetach 与 Activity 解绑时，解除关联，清理全局引用
```

## Activity 和 Fragment 生命周期的关系
- 比如 Activity 的 onPause 会触发所有可见的 Fragment 走 onPause
- Activity 开始创建带动 Fragment
- Fragment 开始销毁带动 Activity

