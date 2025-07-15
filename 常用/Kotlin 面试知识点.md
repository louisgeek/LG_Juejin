# Kotlin 面试知识点

## var 和 val
- var：声明可变变量，可以重新赋值
- val：声明不可变变量（只读），类似 Java 的 final

## Any、Unit 和 Nothing 的区别
- kotlin.Any::class == java.lang.Object::class 成立
- kotlin.Nothing::class == java.lang.Void::class 成立
- kotlin.Unit 类似于 Java 的 void

## data class 数据类
```kotlin
data class Person(val age: Int, val name: String)
```

## 函数类型
- 函数类型是 Kotlin 中一种特殊的数据类型，定义了函数的参数类型和返回值类型（`( 参数类型列表 ) -> 返回值类型`）
- `(Int, Int) -> String` 表示一个接受两个 Int 参数并返回 String 的函数类型
- `() -> Unit` 表示一个无参数且无返回值的函数类型

## 高阶函数
- 高阶函数是指可以接受函数作为参数或返回值类型的函数，高阶函数利用了函数类型将函数当作数据来传递和操作
```kotlin
//函数作为参数
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int)
//函数作为返回值类型
fun getOperation(): (Int) -> Int
```

## Lambda 表达式
- Lambda 表达式是 Kotlin 中​​匿名函数​​的简洁写法，本质上是一个函数类型的实例
- `{ 参数列表 -> 代码块 }`
- 如果 Lambda 表达式是函数的最后一个参数，可以将其移到括号外

```kotlin
//高阶函数 operate，接收函数类型参数 operation
fun operate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

//函数类型的实例（Lambda 表达式赋值）
val add: (Int, Int) -> Int = { x, y -> x + y }

//传递函数类型
val resultAdd = operate(3, 5, add)

//直接传递 Lambda 表达式
val result = operate(3, 5) { x, y -> x + y }
```

## lateinit 和 by lazy 的区别
- lateinit 和 by lazy 都可以实现类似于延迟初始化的情景
- lateinit 延迟初始化，适用于非空类型的 var 属性（不能用于 val 只读属性），需要自行确保在属性访问前被初始化，不保证线程安全，不能用于基本数据类型，没有默认值
- by lazy 懒加载，适用于任何类型（可空和非空都行）的 var 属性和 val 只读属性，在首次访问时才完成延迟初始化（懒加载），默认线程安全，适用于延迟初始化开销较大的对象

## inline、noinline 和 crossinline 的区别
- 普通函数（带 Lambda 参数），可以间接调用，不允许非局部返回
- inline 标记整个函数为内联函数，不可以间接调用，允许非局部返回，作用于方法
- noinline 标记禁止某一个参数内联，可以间接调用，不允许非局部返回，作用于参数
- crossinline 标记跨内联，保持内联优化，可以间接调用，不允许非局部返回，作用于参数

## ShareFlow 和 StateFlow 的区别
- SharedFlow 比 StateFlow 更通用
- SharedFlow 默认不会出现粘性事件而 StateFlow 会
- StateFlow 默认空安全，强制 value 需要被赋值一个初始数据，而且 value 也是非空的，意味着 StateFlow 永远有值的 
- StateFlow 默认防抖，因为 StateFlow 每次发送数据都会与上次缓存的数据作比较，只有不一样才会发送
- SharedFlow 和 StateFlow 的使用场景侧重有所不同，前者更适用于 Event 事件相关，后者则更适用于 State 状态相关

## Flow 操作符 map 和 flatMap 的区别
- map：对 Flow 中的每个元素应用一个转换函数，每个元素转换为一个新的元素，然后生成一个新 Flow
- flatMap：对 Flow 中的每个元素应用一个转换函数，每个元素转换为一个新的 Flow，然后将所有 Flow 合并成一个 Flow

## Kotlin 委托
- 类委托：将接口实现委托给另一个类对象，`class DelegatePlayer(private val player: IPlayer): IPlayer by player`
- 属性委托：将属性的 get/set 逻辑委托给另一个类对象（此类需要提供 getValue 或者 setValue 约定函数），`val lazyValue: String by Delegate()`

## Scope Functions 作用域函数
- T.apply 和 T.also 返回作用域对象（就是 return this）
    - T.apply 使用 this 访问上下文对象，通常是对一个对象的属性进行设置
    - T.also 使用 it 访问上下文对象，通常是对一个对象进行一些附加处理
- T.run、T.let、kotlin.run 和 with(T) 返回 Lambda 表达式的结果值（就是最后一行的值）
    - T.run 使用 this 访问上下文对象，通常是对一个对象的属性进行设置并需要返回结果值
    - T.let 使用 it 访问上下文对象，通常是是在一个局部作用域内引入变量，在非 null 对象上执行表达式
    - kotlin.run 需要自行引用上下文对象，通常是在需要表达式的地方执行多条语句
    - with(T) 使用 this 访问上下文对象，通常是对一个对象进行一组函数调用