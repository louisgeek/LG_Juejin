# java.lang.IllegalStateException Can not perform this action after onSaveInstanceState 解决方案

## Fragment

### 闪退复现

```kotlin
Handler(mainLooper).postDelayed({
    //打开页面后在 5 秒内按 home 或 back 键
     val fragment = Fragment()
     supportFragmentManager.beginTransaction()
     .replace(R.id.container, fragment)
     .commit() //此处会抛异常
}, 5_000)
```

### 错误日志

```shell
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
        at androidx.fragment.app.FragmentManagerImpl.checkStateLoss(FragmentManagerImpl.java:1536)
        at androidx.fragment.app.FragmentManagerImpl.enqueueAction(FragmentManagerImpl.java:1558)
        at androidx.fragment.app.BackStackRecord.commitInternal(BackStackRecord.java:317)
        at androidx.fragment.app.BackStackRecord.commit(BackStackRecord.java:282)
        at com.louis.mykttest.MainActivity.onCreate$lambda-0(MainActivity.kt:47)
```

### 原因分析

```
//commit 抽象方法
FragmentTransaction#commit
//commit 抽象方法实现
- BackStackRecord#commit
-- BackStackRecord#commitInternal(allowStateLoss = false)
--- FragmentManagerImpl#enqueueAction(allowStateLoss = false)
---- FragmentManagerImpl#checkStateLoss
----- FragmentManagerImpl#isStateSaved
isStateSaved 方法返回 true 则会直接抛出该 IllegalStateException 异常
```

### 解决方案

#### 思路 1 提前判断 isStateSaved 状态及时处理

```kotlin
//提前判断
//注意这里是 FragmentManager#isStateSaved 不是 Fragment#isStateSaved，因为此时 FragmentManager 和 Fragment 还未关联起来，所以 Fragment#mFragmentManager 为 null
if (supportFragmentManager.isStateSaved) {
    return
}
val fragment = Fragment()
supportFragmentManager.beginTransaction()
.replace(R.id.container, fragment)
.commit()
```

#### 思路 2 改用 commitAllowingStateLoss 方法处理

```kotlin
 val fragment = Fragment()
supportFragmentManager.beginTransaction()
.replace(R.id.container, fragment)
.commitAllowingStateLoss()
```

## DialogFragment

### 闪退复现

```kotlin
Handler(mainLooper).postDelayed({
   	//打开页面后在 5 秒内按 home 或 back 键
   	val dialogFragment = DialogFragment()
  	dialogFragment.show(supportFragmentManager, "test")
}, 5_000)
```

### 原因分析

```
DialogFragment#show
//commit 抽象方法
- FragmentTransaction#commit
//commit 抽象方法实现
-- BackStackRecord#commit
--- BackStackRecord#commitInternal(allowStateLoss=false);
---- FragmentManagerImpl#enqueueAction(allowStateLoss=false);
----- FragmentManagerImpl#checkStateLoss
------ FragmentManagerImpl#isStateSaved
isStateSaved 方法返回 true 则会直接抛出该 IllegalStateException 异常
```

### 解决方案

#### 思路 1 提前判断 isStateSaved 状态及时处理

重写 DialogFragment#show 方法

```Kotlin
override fun show(manager: FragmentManager, tag: String?) {
//提前判断
//注意这里是 FragmentManager#isStateSaved 不是 Fragment#isStateSaved，因为此时 FragmentManager 和 Fragment 还未关联起来，所以 Fragment#mFragmentManager 为 null
    if (manager.isStateSaved) {
       return
    }
    super.show(manager, tag)
}
```

#### 思路 2 增加 showAllowingStateLoss 方法

```
因为有 
DialogFragment#dismiss -> 直接调用 DialogFragment#dismissInternal(allowStateLoss = false)
//
DialogFragment#dismissAllowingStateLoss -> 直接调用 DialogFragment#dismissInternal(allowStateLoss = true)
//
DialogFragment#show -> 间接调用 BackStackRecord#commit -> 直接调用BackStackRecord#commitInternal(allowStateLoss=false)
//
但为啥没有 DialogFragment#showAllowingStateLoss 方法？（其实源码里有的，只是打了 hide 标记）
使得
DialogFragment#showAllowingStateLoss -> 间接调用 BackStackRecord#commitAllowingStateLoss -> 直接调用 BackStackRecord#commitInternal(allowStateLoss = true)
所以考虑新增一个方法实现该逻辑
```

重写 DialogFragment，反射处理 mDismissed 和 mShownByMe 状态

```Kotlin
fun showAllowingStateLoss(manager: FragmentManager, tag: String?) {
        /**
        mDismissed = false;
        mShownByMe = true;
        FragmentTransaction ft = manager.beginTransaction();
        ft.add(this, tag);
        ft.commit();
         */
        /*
          mDismissed = false
          mShownByMe = true
          */
       DialogFragmentHelper.setFieldValue(this, "mDismissed", false)
       DialogFragmentHelper.setFieldValue(this, "mShownByMe", true)
       val ft = manager.beginTransaction()
       ft.add(this, tag)
       ft.commitAllowingStateLoss()
}
```

PS：附 DialogFragmentHelper 代码

```Kotlin
object DialogFragmentHelper {
    fun setFieldValue(obj: Any, fieldName: String, fieldValue: Any) {
        //androidx.fragment.app.DialogFragment.mDismissed
        //androidx.fragment.app.DialogFragment.mShownByMe
        //java.lang.NoSuchFieldException: No field mDismissed in class Lcom/louis/mykttest/FixedDialogFragment; (declaration of 'com.louis.mykttest.FixedDialogFragment' appears in
//        val field = obj.javaClass.getDeclaredField(fieldName)
        val field = DialogFragment::class.java.getDeclaredField(fieldName)
        field.isAccessible = true
        field[obj] = fieldValue
    }
    fun <T> getFieldValue(obj: Any, fieldName: String): T {
        val field = DialogFragment::class.java.getDeclaredField(fieldName)
        field.isAccessible = true
        return field[obj] as T
    }
}
```

### 利用扩展函数简化

- 可以采用扩展函数替代重写操作

```kotlin
//不要用 show 这个名字
fun DialogFragment.showFixed(manager: FragmentManager, tag: String?) {
    //
    if (manager.isStateSaved) {
        return
    }
    this.show(manager, tag)
}
//
fun DialogFragment.showAllowingStateLoss(manager: FragmentManager, tag: String) {
    /**
    mDismissed = false;
    mShownByMe = true;
    FragmentTransaction ft = manager.beginTransaction();
    ft.add(this, tag);
    ft.commit();
     */
	/*
      mDismissed = false
      mShownByMe = true
     */
    DialogFragmentHelper.setFieldValue(this, "mDismissed", false)
    DialogFragmentHelper.setFieldValue(this, "mShownByMe", true)
    val ft = manager.beginTransaction()
    ft.add(this, tag)
    ft.commitAllowingStateLoss()
}
```
