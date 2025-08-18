
# Android RecyclerView


## NestedScrollView 和 RecyclerView 滑动卡顿
- RecyclerView 设置 android:nestedScrollingEnabled="false"，意味着放弃 RecyclerView 自己的滑动，交给外部的 NestedScrollView 来处理
- 适合 RecyclerView 的内容不是特别多的情况，因为 android:nestedScrollingEnabled="false" 会使 RecyclerView 在初始化就将所有的 Item 都加载出来（高度变成了所有 Item 总的高度），也就是说 RecyclerView 的复用机制就不起作用了

## NestedScrollView 包裹 ConstraintLayout 里的两个 RecyclerView
- 通过 android:fillViewport="true" 配合 android:nestedScrollingEnabled="false" 也无法达到最预期的效果，可以在 ConstraintLayout 外面嵌套一个 FrameLayout（可以去掉 android:fillViewport="true"）
