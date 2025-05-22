# Android include、merge 和 ViewStub
- `<include>`、`<merge>` 和 `<ViewStub>` 是用于优化布局性能和代码复用的三个核心标签

## include 复用布局
- 将重复的布局片段抽取为独立的 xml 文件，然后通过 include 标签进行复用，比如公共头部、底部和通用组件
- 避免重复编写相同的布局代码，提高代码可维护性
- 可以覆盖被引用布局的根容器的属性（包括 View 的 ID）
```xml
<include layout="@layout/titleBarLayout"
         android:id="@+id/titleBar"
         layout_width="match_parent" 
         layout_height="wrap_content"/>
```

## merge 减少视图层级
- 必须作为布局的根标签，与 include 标签配合使用，消除（合并）因 include 标签引入的冗余父容器（当被包含的布局与父布局类型相同时，比如都是 LinearLayout），减少层级嵌套
- 不能单独作为根布局使用，必须通过 include 标签引用
```xml
<!-- 主布局 -->
<LinearLayout>
    <!-- 复用标题栏 -->
    <include layout="@layout/titleBarLayout"/>
    <!-- 内容 -->
</LinearLayout>

<!-- 被引用的 merge 布局，利用 merge 标签替代 LinearLayout -->
<merge>
    <Button />
    <Button />
</merge>
```

## ViewStub 延迟加载布局
- 懒加载布局的占位符，通过 inflate（或 setVisibility(View.VISIBLE)）按需加载减少初始内存占用，仅在需要时才加载指定的布局文件，比如不常用的 UI 组件（如错误提示、加载进度条）和复杂消耗大的组件
- ViewStub 不支持 merge 作为子布局
```xml
<!-- 按需加载错误提示 -->
<ViewStub
    android:id="@+id/stubError"
    android:layout="@layout/errorLayout"
    android:inflatedId="@+id/errorUI"/>
```

```java
ViewStub stub = findViewById(R.id.stubError);
//stub.setLayoutResource(R.layout.customContent); //支持替换布局
View inflatedView = stub.inflate(); //加载布局并返回根视图，只能调用一次
```

## 总结
- include：复用布局，减少重复代码
- merge：需配合 include 标签使用，优化布局层级嵌套，提升性能
- ViewStub：按需延迟加载布局，减少初始化时的资源消耗