# Android Activity、Window 和 DecorView
- android.app.Activity 负责承载 UI 用户界面、处理用户事件交互和管理生命周期，内部会包含一个 Window 对象
- android.view.Window 是一个抽象类，代表了一个窗口的抽象
- com.android.internal.policy.PhoneWindow 类继承自 Window 类，负责确定窗口外观属性（比如窗口大小、位置、背景、颜色、透明度、全屏和分屏等）、控制窗口行为策略和用于承载 DecorView 这个视图容器
- com.android.internal.policy.DecorView 类继承自 FrameLayout，用于实际承载视图的，它通常包含了状态栏、标题栏和内容区域等

## Activity
```java
public class Activity extends ContextThemeWrapper implements LayoutInflater.Factory2, Window.Callback ... {
    private Window mWindow; //实际上就是 PhoneWindow
    public Window getWindow() {
        return mWindow;
    }
    final void attach(Context context, ActivityThread aThread ...) {
        //初始化 PhoneWindow
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
    }
    public void setContentView(@LayoutRes int layoutResID) {
        //就是 PhoneWindow#setContentView
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar(); //初始化 ActionBar
    }
    @Nullable
    public <T extends View> T findViewById(@IdRes int id) {
        //就是 Window#findViewById，内部直接调用了 DecorView#findViewById 方法
        return getWindow().findViewById(id);
    }
}
```

## Window
- 包含着一些窗口的属性以及控制着一些和窗口相关的操作行为等
- 比如 Activity、Dialog 都对应着一个 Window

```java
public abstract class Window {
    public abstract void setContentView(@LayoutRes int layoutResID);
    public abstract @NonNull View getDecorView();
     @Nullable
    public <T extends View> T findViewById(@IdRes int id) {
        //就是 View#findViewById（FrameLayout 继承 ViewGroup，ViewGroup 继承 View）
        return getDecorView().findViewById(id);
    }
}
```

```java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
    private DecorView mDecor;
    @Override
    public final @NonNull View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor(); //初始化 DecorView 和 ContentParent
        }
        return mDecor;
    }
    ViewGroup mContentParent;
    private void installDecor() {
        if (mDecor == null) {
            mDecor = generateDecor(-1);
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
        }
    }
    protected DecorView generateDecor(int featureId) {
        //...
        return new DecorView(context, featureId, this, getAttributes());
    }
    protected ViewGroup generateLayout(DecorView decor) {
        //...
        mDecor.startChanging();
        //
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
        //ID_ANDROID_CONTENT 是 com.android.internal.R.id.content 就是 android:id/content
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        //...
        mDecor.finishChanging();
        return contentParent;
    }
    @Override
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            //在setContentView中会将layout填充到此DecorView中
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        //...
    }
}        
```
 
## DecorView
- DecorView 是所有视图的根视图
- ContentRoot 用来承载 ActionBar 和 ContentParent 的
- ContentParent 就是用来承载我们通过 setContentView 设置的 ContentView 的

```java
public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {
    ViewGroup mContentRoot;
    //PhoneWindow#generateLayout 调用这里
    void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
        //...
        final View root = inflater.inflate(layoutResource, null);
        //...
        mContentRoot = (ViewGroup) root;
        //...
    }
}
```

## 总结
- Activity 包含了一个 Window（PhoneWindow）
- Window 包含了一个 DecorView（FrameLayout）
- DecorView 包含了一个 ContentRoot（ViewGroup，跟据主题决定，比如 ActionBarOverlayLayout、LinearLayout 等）
- ContentRoot 包含了一个 ActionBar 和一个 ContentParent（FrameLayout，id 是 android:id/content）
- 而 setContentView 设置 layoutResID 的 ContentView 布局就是被添加到 ContentParent 中的
- Activity 和 Window 都是抽象概念的，是肉眼看不见的，而实际看得见其实就是 DecorView
