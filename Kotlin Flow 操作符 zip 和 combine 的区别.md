# Kotlin Flow 操作符 zip 和 combine 的区别
- Flow 操作符 zip 和 combine 都是 Flow 的扩展函数，是用来实现合并流的操作符（组合操作符），都可以将两个流合并为一个流
- zip 用于将两个流中的元素合并在一起，当两个流中都有元素就会将这些元素组成一个新元素发出，如果任何一个流没有元素，就不会发出任何元素，如果其中一个流发送的元素比另一个慢，就会等待另一个流发送元素，意味着合并元素操作是同步的
- combine 用于将两个流中的最新元素组合在一起，至少有一个流有新的元素的话，组合操作就会继续进行下去，如果其中一个流发送的元素比另一个慢，就会使用最新的元素

## zip
- zip 操作符将两个 Flow 压缩在一起，它将两个 Flow 流中的元素按顺序一一配对合并在一起返回一个新的流（包含通过 transform 转换函数应用生成后的元素），如果两个流的长度大小不同，zip 会按照较短的流的长度来处理配对，会忽略较长的流中多余的元素
- 如果两个 Flow 流的元素发射不同步，那么先发射的元素会等待对应顺序的后发射的元素到来后才进行合并

```kotlin
    val flow1 = flowOf(1, 2, 3)
    val flow2 = flowOf("one", "two", "three", "four")
    flow1.zip(flow2) { i, s ->
        "$i -> $s"
    }.collect {
        println(it)
    }
    //--------- 打印 ---------
    //1 -> one
    //2 -> two
    //3 -> three
```

```kotlin
    val flow1 = flowOf(1, 2, 3).onEach { delay(300) }
    val flow2 = flowOf("one", "two", "three", "four").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    flow1.zip(flow2) { i, s ->
        "$i -> $s"
    }.collect {
        println("$it diffTime=${System.currentTimeMillis() - startTime}")
    }
    //--------- 打印 ---------
    //1 -> one diffTime=447
    //2 -> two diffTime=854
    //3 -> three diffTime=1261
```

## combine
- combine 操作符将两个 Flow 组合在一起，它将两个 Flow 流中的最新元素进行组合（任何一个流发射新值后通过 transform 转换函数应用生成后的元素），不过当其中一个流提前结束后它仍然会继续处理另一个流的剩余元素
- 只要任意一个流发出新元素就会触发组合操作去组合生成最新的结果，不需要等待两个流的元素同步发射

```kotlin
    val flow1 = flowOf(1, 2, 3)
    val flow2 = flowOf("one", "two", "three", "four")
    flow1.combine(flow2) { i, s ->
        "$i -> $s"
    }.collect {
        println(it)
    }
    //--------- 打印 ---------
    //1 -> one
    //2 -> two
    //3 -> three
    //3 -> four
```

```kotlin
    val flow1 = flowOf(1, 2, 3).onEach { delay(300) }
    val flow2 = flowOf("one", "two", "three", "four").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    flow1.combine(flow2) { i, s ->
        "$i -> $s"
    }.collect {
        println("$it diffTime=${System.currentTimeMillis() - startTime}")
    }
    //--------- 打印 ---------
    //1 -> one diffTime=460
    //2 -> one diffTime=654
    //2 -> two diffTime=872
    //3 -> two diffTime=965
    //3 -> three diffTime=1276
    //3 -> four diffTime=1679
```

## 应用场景
- zip 操作符：同时进行两个网络 Api 接口的调用，并在两个接口都调用完成后一起给出结果（实现在一个回调中返回两个网络请求的结果）进行下一步处理
- combine 操作符：同时监听多个输入字段的状态（比如用户名、密码），只有当所有的输入都满足条件时才启用登录提交的按钮

## 总结
- zip 可以将两个流中的元素成对进行合并处理，适用于两个流长度相等或者只关心较短长度的流的元素的场景，先发射的元素会等待对应顺序的后发射的元素到来后才进行合并操作
- combine 可以将两个流中的最新的元素组合在一起，适用于需要基于多个流的最新状态进行处理的场景，只要任意一个流发出新元素就会触发组合操作
