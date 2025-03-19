# Kotlin 面试知识点



## Kotlin data class 没有无参构造函数的问题如何解决？


## 解构
解构声明的原理： Kotlin 解构声明可以把一个对象的属性分解为一组变量，所以解构声明的本质是局部变量。


## inline 和 noinline  crossinline


## 顶层函数和属性
顶级成员（函数 & 属性）的原理： Kotlin 顶级成员的本质是 Java 静态成员，编译后会自动生成文件名Kt的类，可以使用@Jvm:fileName注解修改自动生成的类名。


## 扩展函数
扩展函数的原理： 扩展函数的语义是在不修改类 / 不继承类的情况下，向一个类添加新函数或者新属性。本质是静态函数，静态函数的第一个参数是接收者类型，调用扩展时不会创建适配对象或者任何运行时的额外消耗。在 Java 中，我们只需要像调用普通静态方法那样调用扩展即可


委托机制的原理： Kotlin 委托的语法关键字是 by，其本质上是面向编译器的语法糖，三种委托（类委托、对象委托和局部变量委托）在编译时都会转化为 “无糖语法”。例如类委托：编译器会实现基础接口的所有方法，并直接委托给基础对象来处理。例如对象委托和局部变量委托：在编译时会生成辅助属性（prop$degelate），而属性 / 变量的 getter() 和 setter() 方法只是简单地委托给辅助属性的 getValue() 和 setValue() 处理
 

中缀函数： 声明 infix 关键字的函数是中缀函数，调用中缀函数时可以省略圆点以及圆括号等程序符号，让语句更自然。



------------------- Kotlin Coroutines 协程知识点 ----------------

## 协程是什么

广义上的协程：和线程类似，也是一种解决异步任务的方案

Kotlin 的协程：可以理解是对线程的一种封装，一种可以让异步任务的代码写成一种类似同步的形式，可以看作是一个线程封装框架，官方称为一种轻量级线程

- 可以解决异步任务代码的嵌套
- 必须依赖线程存在，但是可以在不同的线程之间切换
- 由开发者管理，不需要操作系统进行调度和切换
## Kotlin 协程挂起的本质

## Coroutines 协程是什么
- 广义上的协程：和线程类似，也是一种解决异步任务的方案
- Kotlin 的协程：可以理解是对线程的一种封装，一种可以让异步任务的代码写成一种类似同步的形式，可以看作是一个线程封装框架，官方称为一种轻量级线程
    - 可以解决异步任务代码的嵌套
    - 必须依赖线程存在，但是可以在不同的线程之间切换
    - 由开发者管理，不需要操作系统进行调度和切换

## CoroutineContext 协程上下文
- CoroutineContext 接口的设计和 Map 很相识，也是根据 Key 取值的，相同 Key 的值会被覆盖，支持合并（CoroutineContext 重载了 plus 操作符，所以可以用 + 号组合），可以看作是一系列 CoroutineContext.Element 元素的集合
    - CoroutineName 代表协程的名称，通常可用于调试时候的识别
    - Job 用于管理协程生命周期，算是已启动协程的句柄，比如可以调用 job.cancel() 取消正在运行的协程
    - CoroutineDispatcher（Dispatchers）用于协程处理线程调度，将任务分派到当的线程
    - CoroutineExceptionHandler 用于协程的异常处理，处理未捕获的异常
    - ContinuationInterceptor 的作用是拦截协程的，CoroutineDispatcher 也实现了 ContinuationInterceptor 接口

## suspend 挂起函数
- CPS 转换：Kotlin 编译器会将 suspend 挂起函数转换成一个带有 Callback 参数的回调函数（为其额外添加 Continuation 类型的参数），这里的 Callback 回调接口就是 Continuation 接口
- 简单的理解就是协程使用 Continutation 替换传统的 Callback，每一个协程程序的创建都会伴随 Continutation 的存在，同时协程创建的时候都会自动回调一次 Continutation 的 resumeWith 方法，以便让协程开始执行


兼容 Java 遇到的最大的 “坑”

