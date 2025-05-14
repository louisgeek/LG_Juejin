# Kotlin 嵌套类和内部类
- 嵌套类和内部类是两种定义在其他类内部的类结构
- 嵌套类不能访问外部类的成员，内部类可以访问外部类的所有成员
- 嵌套类不会持有外部类的引用，而内部类会持有外部类的引用

## Nested Class 嵌套类
- 嵌套类是直接定义在其他类内部的类，默认是静态的（类似于 Java 的静态内部类）
- 无需外部类实例，可以直接实例化
- 适用于工具类、辅助类等场景
```kotlin
class Outer {
    private val testValue = 1

    //嵌套类
    class Nested {
        fun testFun() {
            println("Nested testFun")
            //无法访问外部类的成员
            println("testValue: $testValue") //编译报错：Unresolved reference: testValue
        }
    }
}

fun main() {
    //直接实例化嵌套类
    val nested = Outer.Nested()
    nested.testFun()
}
```


## Inner Class 内部类
- 内部类是通过 inner 关键字显式声明的嵌套类（类似于 Java 的非静态内部类）
- 必须先实例化外部类，才能实例化内部类
- 可以通过 this@Outer 获取外部类的引用
- 适用于事件监听、回调逻辑等场景
```kotlin
class Outer {
    private val testValue = 1

    //内部类
    inner class Inner {
        fun testFun() {
            println("Inner testFun")
            //可以访问外部类成员
            println("testValue: $testValue") //编译正常
        }
    }
}

fun main() {
    //需要先实例化外部类，再实例化内部类
    val inner = Outer().Inner()
    inner.testFun()
}
```

## 总结
- Kotlin 的嵌套类对应 Java 的静态内部类
- Kotlin 的内部类对应 Java 的非静态内部类