# Kotlin Flow 操作符 debounce 和 sample 的区别
- Flow 操作符 zip 和 combine 都是 Flow 的扩展函数，是用于控制数据流高频发射的限流操作符，通过不同的机制实现数据过滤操作

## debounce 防抖
- debounce 操作符用于过滤短时间内连续快速发射的数据，仅在指定一段时间内没有新数据时发送最后一次数据，如果在等待期间有新数据（定时器未超时又有新数据），则重新计时并丢弃之前的数据
- 适用场景：搜索框输入（用户停止输入后再发送搜索请求）、表单输入校验
```kotlin
//停止输入后延迟 300ms 触发搜索请求
viewModelScope.launch {
    textChangesFlow
        .debounce(300) //timeoutMillis
        .collectLatest { query ->
            //searchApi(query)
        }
}
```

## sample 采样
- sample 操作符用于定期采样，在固定时间间隔发射最新数据，可能跳过中间的数据，每次发送该间隔内最后一次数据（有点类似 Throttle 节流的策略）
- 
- 适用场景：传感器数据定期采集、定期进行 UI 更新 
```kotlin
//停止输入后延迟 300ms 触发搜索请求
viewModelScope.launch {
    dataEventsFlow
        .sample(1000) //periodMillis
        .collectLatest {
            //updateUI
        }
}
```

## 总结
- debounce 侧重指定时间内无新数据时触发执行（延迟执行），sample 侧重定时去采样一次最新的数据
- debounce 当数据流发射新数据时，启动一个定时器，而 sample 中的定时器开始的点在时间轴上是固定的，不管当下有没有新数据的产生