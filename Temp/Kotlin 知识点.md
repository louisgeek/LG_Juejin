# Kotlin 知识点

## measureTimeMillis 和 measureNanoTime
```kotlin
//currentTimeMillis
val costTime = measureTimeMillis { 
    //需要统计耗时的代码
}
//nanoTime
val costTime = measureNanoTime { 
    //需要统计耗时的代码
}
```

```kotlin
public inline fun measureTimeMillis(block: () -> Unit): Long {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val start = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - start
}

public inline fun measureNanoTime(block: () -> Unit): Long {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val start = System.nanoTime()
    block()
    return System.nanoTime() - start
}
```


## kotlin.repeat
```kotlin
repeat(5) {
    println("test $it")
}
//--------------
test 0
test 1
test 2
test 3
test 4
```

```kotlin
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }
    //就是 for 循环
    for (index in 0 until times) {
        action(index)
    }
}
```




## kotlin.runCatching 和 T.runCatching
```kotlin
kotlin.runCatching {
    //需要 try-catch 的代码
    "data" //返回值
}.onSuccess { data ->
}.onFailure { throwable ->
}

obj.runCatching {
    //直接访问 obj 属性
    //需要 try-catch 的代码
    "data" //返回值
}.onSuccess { data ->
}.onFailure { throwable ->
}
```

```kotlin
@InlineOnly
@SinceKotlin("1.3")
public inline fun <R> runCatching(block: () -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}

@InlineOnly
@SinceKotlin("1.3")
public inline fun <T, R> T.runCatching(block: T.() -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}
```