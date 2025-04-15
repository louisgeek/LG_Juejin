# Kotlin inline、noinline 和 crossinline 的区别
- 间接调用：内联函数的函数体中不直接调用函数类型参数，而是将其放在一个拥有另一个上下文的 Lambda 表达式或匿名类中
- 非局部返回：Lambda 表达式中 return 返回到外层函数的情况，而局部返回就是 return 返回到函数本身
- 普通函数的 Lambda 表达式中不允许非局部返回
```kotlin
fun fun01(block: () -> Unit): () -> Unit {
    block() //编译正常
    //
    Runnable {
        block() //编译正常，间接调用
    }
    return block //编译正常
}
fun main() {
    //
    fun01 {
        println("Test01")
        return@fun01 //局部返回，编译正常
        return@main  //非局部返回，编译报错：'return' is not allowed here
        return       //编译报错：'return' is not allowed here
    }
}
```

## inline 内联函数
- 用于标记一个函数为内联函数，在编译时将函数体代码（包括参数）直接内联展开嵌入到调用处（函数中的代码直接复制到调用它的地方），而不是通过传统函数调用（压栈进栈）的方式执行，从而减少函数调用和对象创建等额外的开销，inline 非常适合用在包含 Lambda 参数的函数上（特别是涉及循环的场景）
- 函数调用：减少函数调用栈的开销（减少函数栈的层级），调用一个方法是一个压栈和出栈的过程
- 对象创建：调用高阶函数（带 Lambda 参数）而创建函数对象（比如 Function 接口的实例）
- inline 不仅让函数调用被内联，还会让 Lambda 参数也被内联，还可以让 Lambda 表达式里面的直接使用 return 返回（就是允许非局部返回）
- 允许非局部返回：在 inline 函数的 Lambda 中可以使用 return 直接返回到外层函数（而非 Lambda 自身），相当于会退出外层函数，不过这其实算是一个缺点，因为代码逻辑很容易出现不是预期想象的结果，所以就引入了一种方式解决，就是利用 noinline 或 crossinline 来限制在 Lambda 表达式中直接使用 return 返回的行为
- 内联函数的 Lambda 参数是不允许作为参数传递给非内联的函数的，需要加 noinline 解决
- 内联函数中不允许 Lambda 参数对象作为返回值返回，需要加 noinline 解决

```kotlin
inline fun fun02(block: () -> Unit): () -> Unit {
    block() //编译正常
    //
    Runnable {
        block() //编译报错，提示加 crossinline
    }
    return block //编译报错，提示加 noinline
}
fun main() {
    //
    fun02 {
        println("Test02")
        return@fun02 //局部返回，编译正常
        return@main  //非局部返回，编译正常
        return       //编译正常，return 直接返回外层 main 函数
    }
}
```

## noinline 禁止内联特定参数
- 与 inline 结合使用，用于标记一个函数的某个参数（通常是 Lambda 表达式）不被内联，Lambda 会像普通函数参数一样（作为普通的函数对象传递），相当于使用 noinline 来指定某个函数参数不使用 inline 带来的内联特性（当前函数参数不参与高阶函数的内联）

```kotlin
inline fun fun03(
    block: () -> Unit,               //内联
    noinline callback: () -> Unit    //不内联
) : () -> Unit {
    block()
    callback()
    //
    Runnable {
        block()      //编译报错，提示加 crossinline
        callback()   //编译正常
    }
    return  block    //编译报错，提示加 noinline
    return  callback //编译正常
}
fun main() {
    fun03({
        println("Test03")
        return@fun03 //局部返回，编译正常
        return@main  //非局部返回，编译正常
        return       //编译正常，return 直接返回外层 main 函数
    }, {
        println("Test0302")
        return@fun03 //局部返回，编译正常
        return@main  //非局部返回，编译报错：'return' is not allowed here
        return       //编译报错：'return' is not allowed here
    })
}
```

## crossinline 跨内联限制非局部返回
- 用于标记一个函数的某个参数（通常是 Lambda 表达式），Lambda 可以被间接调用，但禁止在 Lambda 中使用 return（可以确保 Lambda 中不会出现非局部返回，防止 return 意外跳出外层函数），避免因内联导致 return 作用域混乱
- 保持内联优化：crossinline 标记的 Lambda 仍会被内联展开（除非被传递到非内联函数中），所以仍旧不允许 Lambda 参数对象作为返回值返回（因为它依然是内联的）

```kotlin
inline fun fun04(
    block: () -> Unit,                //内联
    noinline callback: () -> Unit,    //不内联
    crossinline action: () -> Unit    //跨内联
): () -> Unit {
    block()
    callback()
    action()
    //
    Runnable {
        block()      //编译报错，提示加 crossinline
        callback()   //编译正常
        action()     //编译正常
    }
    return  block    //编译报错，提示加 noinline
    return  callback //编译正常
    return  action   //编译正常提示加 noinline
}

fun main() {
    fun04({
        println("Test04")
        return@fun04 //局部返回，编译正常
        return@main  //非局部返回，编译正常
        return       //编译正常，return 直接返回外层 main 函数
    }, {
        println("Test0402")
        return@fun04 //局部返回，编译正常
        return@main  //非局部返回，编译报错：'return' is not allowed here
        return       //编译报错：'return' is not allowed here
    }, {
        println("Test0403")
        return@fun04 //局部返回，编译正常
        return@main  //非局部返回，编译报错：'return' is not allowed here
        return       //编译报错：'return' is not allowed here
    })
}
```

## 总结
- 普通函数（带 Lambda 参数），可以间接调用，不允许非局部返回
- inline 标记整个函数为内联函数，不可以间接调用，允许非局部返回，作用于方法
- noinline 标记禁止某一个参数内联，可以间接调用，不允许非局部返回，作用于参数
- crossinline 标记跨内联，保持内联优化，可以间接调用，不允许非局部返回，作用于参数

