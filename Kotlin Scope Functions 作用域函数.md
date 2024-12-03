# Kotlin Scope Functions 作用域函数
- Kotlin Standard 标准库中提供了一系列函数, 用来在某个指定的对象上下文中执行一段代码，可以对一个对象调用这些函数, 并提供一个 Lambda 表达式, 函数会创建一个临时的 Scope 作用域，在这个作用域内, 你可以访问这个对象, 而不需要指定名称. 这样的函数叫作 Scope Functions 作用域函数
- 两大区别：它们的返回值和访问上下文对象的方式


## kotlin.run
- kotlin 包下的顶层函数 run
- 参数 block 这个 lambda 表达式的返回值就是 run 函数的返回值
```kotlin
@kotlin.internal.InlineOnly
//inline 表示在编译时期会把调用这个函数的地方用这个函数的方法体进行替换
public inline fun <R> run(block: () -> R): R {
    contract { //契约，告诉编译器这里会遵守一些约定（是开发者和编译器沟通的桥梁），帮助编译器更好的分析代码
        //告知编译器 block 这个 lambda 将会被执行的次数，EXACTLY_ONCE 代表只执行一次，AT_MOST_ONCE 代表至多一次，AT_LEAST_ONCE 代表至少一次，默认 UNKNOWN 未知
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block() //这里等同于 block.invoke()
}
```

```kotlin
//result 就是返回的 Boolean 值
val result = kotlin.run {
    initView()
    finish() // Activity
    true //lambda 表达式的返回值
}
```



## T.run 或 run
- T 的扩展函数 run
- run 其实就是 this.run

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block() //这里等同于 block.invoke(this)
}
```

```
val numList = arrayListOf(1,2,3)
//result 就是返回的 Boolean 值
val result = numList.run {
    val e = first()
    true
}
```


## with(T)
- 顶层函数 with，带一个参数
```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block() //这里不能用 invoke 替代
}
```

```kotlin
val numList = arrayListOf(1,2,3)
//result 就是返回的 Boolean 值
val result =  with(numList) {
    val e = first()
    true
}
```

## T.apply
- T 的扩展函数 apply
```kotlin
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block() //这里等同于 block.invoke(this)
    return this
}
```

```kotlin
val numList = arrayListOf(1,2,3)
//result 就是 numList
val result =  numList.apply {
    val e = first()
}
```

## T.also
- T 的扩展函数 also
```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this) //这里等同于 block.invoke(this)
    return this
}
```

```kotlin
val numList = arrayListOf(1,2,3)
//result 就是 numList
val result =  numList.also {
    val e = it.first() //需要用 it
}
```

## T.let
- T 的扩展函数 let
```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this) //这里等同于 block.invoke(this)
}
```

```kotlin
val numList = arrayListOf(1,2,3)
//result 就是返回的 Boolean 值
val result =  numList.let {
    val e = it.first() //需要用 it
    true
}
```

## 总结
- T.apply 和 T.also 返回作用域对象（就是 return this）
    - T.apply 使用 this 访问上下文对象，通常是对一个对象的属性进行设置
    - T.also 使用 it 访问上下文对象，通常是对一个对象进行一些附加处理

- T.run、T.let, kotlin.run, 和 with(T) 返回 Lambda 表达式的结果值（就是最后一行的值）
    - T.run 使用 this 访问上下文对象，通常是对一个对象的属性进行设置并需要返回结果值
    - T.let 使用 it 访问上下文对象，通常是是在一个局部作用域内引入变量，在非 null 对象上执行表达式
    - kotlin.run 需要自行引用上下文对象，通常是在需要表达式的地方执行多条语句
    - with(T) 使用 this 访问上下文对象，通常是对一个对象进行一组函数调用
