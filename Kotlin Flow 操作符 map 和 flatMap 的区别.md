# Kotlin Flow 操作符 map 和 flatMap 的区别
- map：对 Flow 中的每个元素应用一个转换函数，每个元素转换为一个新的元素，然后生成一个新 Flow
- flatMap：对 Flow 中的每个元素应用一个转换函数，每个元素转换为一个新的 Flow，然后将所有 Flow 合并成一个 Flow

## map
- 转换后的 Flow 元素数量与原 Flow 相同，顺序保持一致
- 应用场景：比如数值计算、数据格式化等

```kotlin
flowOf(1, 2, 3)
  .map { it * 2 }
  .collect { 
   println(it) 
  }
//--------- 打印 ---------
//2
//4
//6
```

```kotlin
flowOf(listOf(1, 2), listOf(3, 4))
  .map { it } //List<Int>
  .collect { println(it) }
//--------- 打印 ---------
//[1, 2]
//[3, 4]
```

## flatMap
- flatMap 的转换函数必须返回 `Flow<R>`
- 应用场景：比如组合异步操作（需要针对每个值触发异步操作，比如 API 网络请求）的场景、展平嵌套数据结构（需要合并多个子 Flow）等
- flatMap 根据合并方式不同，分成 flatMapConcat、flatMapMerge 和 flatMapLatest，而 flatMap 默认就是 flatMapConcat
- flatMapConcat：串行（连接）合并，按顺序依次处理执行子 Flow，结果按顺序输出（比如串行化任务） 
- flatMapMerge：并行合并，并行处理子 Flow，最终合并所有结果，因此结果输出的顺序可能会有所不同（比如并行下载）
- flatMapLatest：取消前一个未完成的子 Flow 的处理操作，仅保留最新的子 Flow 的结果（比如输入框实时输入的搜索请求，可以及时取消过期请求）

```kotlin
fun getToken(): Flow<String> = flow { emit("token") }
fun getUserInfo(token: String): Flow<UserInfo> = flow { emit(UserInfo(token)) }

getToken()
  .flatMapConcat { token ->
    //
    getUserInfo(token)
  }
  .collect { userInfo -> println("UserInfo: $userInfo") }
```

```kotlin
//展平为单一 Flow
flowOf(listOf(1, 2), listOf(3, 4))
  .flatMapConcat { it.asFlow() } //子 Flow
  .collect { println(it) }
//--------- 打印 ---------
//1
//2
//3
//4
```






