
# Android 事件分发机制
- 事件分发机制遵循了一种类似于责任链模式的设计，是一套在不同的视图层级中进行传递和处理事件的机制
- 事件从 Activity 开始，先传到 Window，然后通过 ViewGroup 按照视图树的层次结构依次向下传递到子视图，寻找一个可以处理事件的视图，如果该视图没有消费该事件，则事件会逐级向上回传，直到找到一个能够处理事件的视图，或者最终没有被任何视图处理就到达 Activity 的 onTouchEvent 方法进行处理
- 如果 onTouchEvent 方法返回 true 表示要消费处理这个事件，返回 false 则表示不处理事件，事件会按照分发流程向上返回传递
- View#dispatchTouchEvent(MotionEvent event) 分发事件，ViewGroup 重写了该方法
- ViewGroup#onInterceptTouchEvent(MotionEvent event) 是 ViewGroup 特有的方法，代表 ViewGroup 是否拦截事件，默认返回 false，可按需重写
- View#onTouchEvent(MotionEvent event) 处理事件，ViewGroup 复用了该方法
- 事件序列：通常情况下 MotionEvent.ACTION_DOWN 标志着一个事件序列的开始，而 MotionEvent.ACTION_UP 则标志着一个事件序列的结束，只有当一个视图处理了 ACTION_DOWN 事件，它才会收到后续的 ACTION_MOVE 和 ACTION_UP 事件
- ViewGroup#requestDisallowInterceptTouchEvent(boolean disallowIntercept) 请求不允许拦截


```java
/**
 * ViewGroup 的三个核心方法
 * 伪代码
 */
public boolean dispatchTouchEvent(MotionEvent event) {
    //dispatchTouchEvent 进行事件分发
    //onInterceptTouchEvent 是否拦截事件
    if (onInterceptTouchEvent(event)) {
        //判断如果拦截则走 ViewGroup 自己的 onTouchEvent 进行处理
        //onTouchEvent 处理事件
        return onTouchEvent(event);
    }
    return child.dispatchTouchEvent(event);
}
```

## 事件流
- 事件分发传递的顺序：Activity -> Window（PhoneWindow） -> DecorView（FrameLayout） -> ViewGroup -> View

## Activity 事件分发
```java
 Activity#dispatchTouchEvent
 -PhoneWindow#superDispatchTouchEvent //直接调用 DecorView#superDispatchTouchEvent
 --DecorView#superDispatchTouchEvent  //直接调用 super.dispatchTouchEvent，就是 ViewGroup#dispatchTouchEvent
 ---super.dispatchTouchEvent          //至此事件就实现从 Activity 传递到 ViewGroup 里去了，如果事件被消费返回 true 则事件结束，就不继续走 Activity#onTouchEvent 了
 -Activity#onTouchEvent               //如果事件未被消费才会走 Activity 自己的 onTouchEvent 进行消费
```

```java
//Activity#dispatchTouchEvent
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();//空方法，可按需重写
    }
    //就是 PhoneWindow#superDispatchTouchEvent 
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true; //事件结束
    }
    //Activity#onTouchEvent
    return onTouchEvent(ev);
}
public boolean onTouchEvent(MotionEvent event) {
     if (mWindow.shouldCloseOnTouch(this, event)) {
         //比如 Dialog 样式的 Activity 点击对话框外侧后弹框消失功能
         finish();
         return true;
     }
     return false;
 }
```

## ViewGroup 事件分发
- ViewGroup 包含子 View，侧重是否需要拦截事件，意味着是否需要把事件继续分发给子 View，如果 ViewGroup#onInterceptTouchEvent 方法返回 true 则表示这个 ViewGroup 要拦截事件，那后续事件就不会再传递给它的子视图了，就是由这个 ViewGroup 自己来处理，而如果返回 false 则会继续传递事件给它的子视图

```java
ViewGroup#dispatchTouchEvent
-判断当前是 MotionEvent.ACTION_DOWN 事件或者存在可以消费事件的对象，否则就直接标记 intercepted 为拦截
--判断 disallowIntercept 不允许拦截的值，如果有不允许拦截的条件，那么就直接标记 intercepted 为不拦截，如果没有不允许拦截的条件，就交由 ViewGroup#onInterceptTouchEvent 去处理 intercepted 拦截标记
---如果 intercepted 标记为拦截，那么子视图的事件都被拦截了，就会走 super#dispatchTouchEvent，也就是父视图自己的 View#dispatchTouchEvent
---如果 intercepted 标记为不拦截，那么通过 for 循环遍历当前 ViewGroup 下的所有 View，执行的就是子视图的 View#dispatchTouchEvent
```

```java
//ViewGroup#dispatchTouchEvent
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    //...
    boolean handled = false;
    //...
    // Handle an initial down.
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // Throw away all previous state when starting a new touch gesture.
        // The framework may have dropped the up or cancel event for the previous gesture
        // due to an app switch, ANR, or some other state change.
        cancelAndClearTouchTargets(ev);
        resetTouchState(); //内部会重置 mGroupFlags 标记，相当于把 disallowIntercept 置为 false
    }
    // Check for interception.
    final boolean intercepted; //是否拦截的标记
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
        //如果是 ACTION_DOWN 事件或者存在可以消费事件的对象，就准备拦截
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0; //不允许拦截
        if (!disallowIntercept) {
            //如果没有不允许拦截的条件，就交由 ViewGroup#onInterceptTouchEvent 去处理 intercepted 拦截标记，默认返回 false
            intercepted = onInterceptTouchEvent(ev);
            ev.setAction(action); // restore action in case it was changed
        } else {
            //如果有不允许拦截的条件，那么就直接标记为不拦截
            intercepted = false;
        }
    } else {
        //如果不是 ACTION_DOWN 事件且没找到可以消费事件的对象，就直接标记为拦截
        // There are no touch targets and this action is not an initial down
        // so this view group continues to intercept touches.
        intercepted = true;
    }
    //...
    if (!canceled && !intercepted) {
        //...
        final int childrenCount = mChildrenCount;
        //...
        final View[] children = mChildren;
        for (int i = childrenCount - 1; i >= 0; i--) {
            //...
            //调用 ViewGroup#dispatchTransformedTouchEvent
            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                //...
            }
            //...
        }
    }
    //...
    return handled;
}
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel, View child, int desiredPointerIdBits) {
    final boolean handled;
    //...
    final MotionEvent transformedEvent;
    if (newPointerIdBits == oldPointerIdBits) {
        if (child == null || child.hasIdentityMatrix()) {
            if (child == null) {
                //父视图自己的 View#dispatchTouchEvent
                handled = super.dispatchTouchEvent(event);
            } else {
                final float offsetX = mScrollX - child.mLeft;
                final float offsetY = mScrollY - child.mTop;
                event.offsetLocation(offsetX, offsetY);
                //子视图的 View#dispatchTouchEvent
                handled = child.dispatchTouchEvent(event);
                event.offsetLocation(-offsetX, -offsetY);
            }
            return handled;
        }
        transformedEvent = MotionEvent.obtain(event);
    } else {
        transformedEvent = event.split(newPointerIdBits);
    }
    // Perform any necessary transformations and dispatch.
    if (child == null) {
        //父视图自己的 View#dispatchTouchEvent
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        final float offsetX = mScrollX - child.mLeft;
        final float offsetY = mScrollY - child.mTop;
        transformedEvent.offsetLocation(offsetX, offsetY);
        if (!child.hasIdentityMatrix()) {
            transformedEvent.transform(child.getInverseMatrix());
        }
        //子视图的 View#dispatchTouchEvent
        handled = child.dispatchTouchEvent(transformedEvent);
    }
    // Done.
    transformedEvent.recycle();
    return handled;
}
```

## View 事件分发
- 侧重如何去消费处理事件

```java
View#dispatchTouchEvent
-View.OnTouchListener#onTouch 如果事件被 onTouch 监听消费返回 true 了则事件就不继续走 View#onTouchEvent 了
-View#onTouchEvent 如果事件未被 onTouch 消费才会走 View#onTouchEvent 进行处理
--View#performClick
---View.OnClickListener#onClick 点击事件
```

```java
//View#dispatchTouchEvent
public boolean dispatchTouchEvent(MotionEvent event) {
    //...
    boolean result = false;
    //...
    final int actionMasked = event.getActionMasked();
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // Defensive cleanup for new gesture
        stopNestedScroll();
    }
    if (onFilterTouchEventForSecurity(event)) {
        if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
            result = true;
        }
        //noinspection SimplifiableIfStatement
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                //View.OnTouchListener#onTouch，如果 OnTouchListener#onTouch 处理返回 true 代表消费了，就不继续走 View#onTouchEvent 方法了
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }
        //View#onTouchEvent，内部处理 View#performClick、View.OnClickListener#onClick 等逻辑
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }
    //...
    // Clean up after nested scrolls if this is the end of a gesture;
    // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
    // of the gesture.
    if (actionMasked == MotionEvent.ACTION_UP ||
            actionMasked == MotionEvent.ACTION_CANCEL ||
            (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }
    return result;
}
```

## ViewGroup#requestDisallowInterceptTouchEvent(boolean disallowIntercept)
- 请求不拦截，disallowIntercept 默认 false
- 默认情况下，父视图优先处理触摸事件，就有机会去进行拦截事件，而存在某些情况下可能希望子视图能够优先处理触摸事件，这时就可以使用这个方法，通常写在 View 的 onTouchEvent 方法中

```java
//ViewGroup#requestDisallowInterceptTouchEvent
@Override
public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
    if (disallowIntercept == ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0)) {
        // We're already in this state, assume our ancestors are too
        return;
    }
    if (disallowIntercept) {
        mGroupFlags |= FLAG_DISALLOW_INTERCEPT;
    } else {
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
    }
    // Pass it up to our parent
    if (mParent != null) {
        //ViewGroup
        mParent.requestDisallowInterceptTouchEvent(disallowIntercept);
    }
}
```

```java
//View#onTouchEvent
@Override
public boolean onTouchEvent(View v, MotionEvent event) {  
     switch (event.getAction()) {  
     case MotionEvent.ACTION_MOVE:   
         //请求父视图不要拦截触摸事件
         getParent().requestDisallowInterceptTouchEvent(true);  
         break;  
     case MotionEvent.ACTION_UP:  
     case MotionEvent.ACTION_CANCEL:  
         //当触摸结束或取消时，及时恢复父视图拦截触摸事件
         getParent().requestDisallowInterceptTouchEvent(false);  
         break;  
     }  
} 
```

## 事件冲突
- 外部拦截法：是在父 View 的 onInterceptTouchEvent 方法中进行事件拦截判断
- 内部拦截法：是在子 View 的 dispatchTouchEvent 方法中，通过调用父视图的 ViewGroup#requestDisallowInterceptTouchEvent 方法来控制父 View 是否拦截事件

## 实例分析
### 点击一个按钮
- 事件分发到 ViewGroup 默认不拦截
- 满足条件走 for 循环遍历子 View，找到这个 View 按钮
- 然后走这个子 View 的 View#dispatchTouchEvent -> View#onTouchEvent -> View#performClick -> View#onClick，执行了 View 的点击事件
- 此时 ViewGroup#dispatchTouchEvent 分发事件就被消费返回 true 了，所以 ViewGroup 的点击事件是不会走的

### 点击空白区域
- 事件分发到 ViewGroup 默认不拦截
- 满足条件走 for 循环遍历子 View，未找到子 View
- 然后 ViewGroup 自己执行 super#dispatchTouchEvent 
- 也就是此时走父视图自己的 View#dispatchTouchEvent -> View#onTouchEvent -> View#performClick -> View#onClick，也就是执行了 ViewGroup 自己的点击事件

