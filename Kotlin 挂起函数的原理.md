# Kotlin 挂起函数的原理

## CPS 思想
- Continuation Passing Style 延续传递风格，是一种函数式的编程风格（思想），核心理念是函数接受一个 Continuation 参数，最终会通过调用该参数来传递结果，而不是直接返回结果给调用者
- Continuation 延续（继续），是一种抽象，表示当前函数执行完后剩余要执行的逻辑（即后续操作），这样可以更灵活地控制执行顺序

## ​​Continuation
Kotlin 挂起函数
```kotlin
suspend fun getInfo() : String {
    val result = "testInfo"
    return result
}
```

感受一下 Kotlin 生成字节码后反编译成 Java 代码
```java
@Nullable
public final Object getInfo(@NotNull Continuation $completion) {
    String result = "testInfo";
    return result;
}
```

感受一下 Java 代码调用 Kotlin 挂起函数
```kotlin
object KotlinTool {
    suspend fun getInfo():String {
        delay(2000L)
        val result = "Kotlin testInfo"
        return result
    }
}
```

```java
//相当于挂起函数在经过 Kotlin 编译器转换后会多出一个 Continuation 参数
KotlinTool.INSTANCE.getInfo(new Continuation<String>() {
    @Override
    public void resumeWith(@NonNull Object result) {
        //延时后会打印 resumeWith: Kotlin testInfo
        Log.e("TAG", "resumeWith: " + result);
    }

    @Override
    public CoroutineContext getContext() {
        return EmptyCoroutineContext.INSTANCE;
    }
});
```

Continuation 接口
```kotlin
//kotlin.coroutines.Continuation
public interface Continuation<in T> {
    public val context: CoroutineContext
    //
    public fun resumeWith(result: Result<T>)
}
```

普通函数（非挂起函数）
```kotlin
//相当于挂起函数原本的返回值类型被挪到了 Continuation 参数当中作为泛型，而转换过后的函数返回值类型变成了 Any? 类型
fun getInfo(continuation: Continuation<String>): Any? {
    val result = "testInfo"
    continuation.resume(result)
    return Unit
}
```

Continuation 相当于一个 Callback，换成 Callback 接口感受一下
```kotlin
interface Callback<T> {
    fun onSuccess(data: T)
}
//
fun getInfo(callback: Callback<String>): Any? {
    val result = "testInfo"
    callback.onSuccess(result)
    return Unit
}
```

## 状态机
- 状态机就是一种在不同状态之间进行流转并执行相应逻辑的行为
- 每个挂起函数会被 Kotlin ​​编译器编译自动生成状态机，当函数挂起时，挂起点的状态（挂起时的执行上下文，比如局部变量、执行位置等）被保存到 Continuation 对象中，恢复时从对象中获取信息继续执行
- 通过 label 代码段嵌套，配合 switch 表达式巧妙构造出一个状态机结构，状态转换由 Continuation 管理和触发

```kotlin
suspend fun testSuspend() {
    Log.e("TAG","0 start")
    val info = getInfo()
    Log.e("TAG","1 info=$info")
    val detailInfo = getDetailInfo(info)
    Log.e("TAG","2 detailInfo=$detailInfo")
    val moreInfo = getMoreInfo(detailInfo)
    Log.e("TAG","3 moreInfo=$moreInfo")
}
```

```java
//生成字节码后反编译
public final Object testSuspend(Continuation completion) {
    //testSuspend 原本的代码被拆分到状态机里各个状态中分开执行
    ContinuationImpl continuation = new ContinuationImpl(completion) {
            //...
            int label; //状态机当前的状态

            @Nullable
            public final Object invokeSuspend(@NotNull Object $result) {
                //...
                return testSuspend(this);
            }
         };
    //...
    Object result;
    //switch 表达式实现了状态机，label 改变一次，就代表了挂起函数被调用了一次，如果函数被挂起了就会返回 COROUTINE_SUSPENDED（代码中已省略）
    //状态机会把之前的结果保存在 continuation 实例中
    switch (continuationImpl.label) {
        case 0: //初始状态
            Log.e("TAG", "0 start");
            continuationImpl.label = 1; //状态控制，准备进入下一次状态
            result = this.getInfo(continuation);
            break;
        case 1:
            String info = (String)result; //获取上一步的结果
            Log.e("TAG", "1 info=" + info);
            continuationImpl.label = 2; //状态控制，准备进入下一次状态
            result = this.getDetailInfo(info, continuation);
            break;
        case 2:
            String detailInfo = (String)result; //获取上一步的结果
            Log.e("TAG", "2 detailInfo=" + detailInfo);
            continuationImpl.label = 3;
            result = this.getMoreInfo(detailInfo, continuation);
    }
    //...
    String moreInfo = (String)result;
    Log.e("TAG", "3 moreInfo=" + moreInfo);
    return Unit.INSTANCE;
}

```

## 总结
- Kotlin 挂起函数的挂起和恢复的本质是 CPS 思想（挂起函数的基础） + 状态机，CPS 帮我们引入 Continuation，通过 ​​Continuation 传递上下文配合 Kotlin ​​编译器生成的状态机的​​流转，从而实现了异步代码的同步化编写（可以用同步的方式写异步代码，就是因为 Continuation 包含了 Callback 的形态）
- 状态机原理：每两个挂起点之间可以看为一个状态，每次进入状态机时都有一个当前的状态，然后执行该状态下对应的代码，如果程序执行完毕则返回结果值，否则返回一个特殊值（比如 COROUTINE_SUSPENDED），表示从这个状态退出并等待下次进入，相当于创建了一个可重复进入的方法，每次都进入使用同一个方法，然后根据不同状态来执行不同分支的代码