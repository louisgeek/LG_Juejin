# Kotlin inline、noinline 和 crossinline 的区别
- 间接调用：比如作为参数传递给其他函数或作为返回值返回
- 非局部返回：Lambda 中 return 直接返回外层函数的情况，而局部返回就是 return 返回到这个函数本身

## inline 内联函数
- 用于标记一个函数为内联函数，在编译时将函数体代码（包括参数）直接内联展开嵌入到调用处（函数中的代码直接复制到调用它的地方），而不是通过传统函数调用（压栈进栈）的方式执行，从而减少函数调用和对象创建等额外的开销，inline 适合用在包含 Lambda 参数的函数上
- 函数调用：减少函数调用栈的开销（减少函数栈的层级），调用一个方法是一个压栈和出栈的过程
- 对象创建：调用高阶函数（带 Lambda 参数）而创建函数对象（比如 Function 接口的实例）
- inline 不仅让函数调用被内联，还会让函数参数也被内联，它可以让函数参数里面的直接 return 生效（因为普通 Lambda 不能直接使用 return），在 inline 函数的 Lambda 中可以使用 return 直接返回到调用该函数的外层函数（而非 Lambda 自身），相当于会退出外层函数，不过这其实算是一个缺点，因为代码逻辑很容易出现不是想象中预期的结果，所以引入了一种方式解决，就是利用 noinline 或 crossinline 来限制在 Lambda 中直接使用 return 的行为

```kotlin
inline fun fun01(block: () -> Unit) {
    block() //调用 Lambda
}
fun main() {
    fun01 {
      println("Test01") 
      return //直接退出 main 函数（支持 return 返回到外层函数），如果没有 inline 标记的话，编译器会报错（不过可以改成 return@fun01，但是这样只会退出 Lambda）
    }
    println("Test02") //这行代码不会执行
}
```

## noinline 禁止内联特定参数
- 与 inline 结合使用，用于标记一个函数的某个参数（通常是 Lambda）不被内联，Lambda 会像普通函数参数一样（作为普通的函数对象传递），相当于使用 noinline 来指定某个函数参数不使用 inline 带来的特性
- 间接调用：如果需要将 Lambda 传递给其他非内联函数时（因为内联函数的函数参数是不允许作为参数传递给非内联的函数的），就必须使用 noinline，如果需要将 Lambda 作为返回值返回时，必须使用 noinline

```kotlin
inline fun fun02(
    action: () -> Unit,              //内联
    noinline callback: () -> Unit    //不内联
) {
    action()
    callback()
}
fun main() {
    fun02({
        println("Test01")
        //return  //这里直接用 return 编译器不会报错
    }, {
        println("Test02")
        //return  //这里直接用 return 编译器会报错
    })
    println("Test03")
}
```

## crossinline 跨内联限制非局部返回
- 用于标记一个函数的某个 Lambda 参数，表示该 Lambda 可以被间接调用，但禁止在 Lambda 中使用 return（可以确保 Lambda 中不会出现非局部返回，防止 return 意外跳出外层函数），避免因内联导致 return 作用域混乱
- 保持内联优化：crossinline 标记的 Lambda 仍会被内联展开（除非被传递到非内联函数中）

```kotlin
inline fun fun03(crossinline block: () -> Unit) {
    block() //调用 block 时，不能通过 return 返回到外层函数
}
fun main() {
    fun03 {
        return //这里直接用 return 编译器会报错（crossinline 禁止使用 return）
    }
}
```

## 总结
- 普通函数（带 Lambda 参数），可以间接调用，不允许非局部返回
- inline 标记整个函数为内联函数，不可以间接调用，允许非局部返回
- noinline 标记禁止某一个参数内联，可以间接调用，不允许非局部返回
- crossinline 标记跨内联，保持内联优化，可以间接调用，不允许非局部返回

