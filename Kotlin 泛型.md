# Kotlin 泛型
- 和 Java 类似，Kotlin 也存在类型擦除，不过 Kotlin 对部分场景进行了一些优化（比如对于 inline 内联函数，可以支持类型参数不会被擦除）

## 边界约束
- 约束类型参数的上界，Kotlin 中没有像 Java 中的下界通配符（super）
```kotlin
//使用 : 限制 T 必须是 Number 类或其子类
fun <T : Number> printInfo(info: T) {
    println(info)
}
//使用 where 子句限制 T 必须是 Number 类或其子类
fun <T> printElements(list: List<T>) where T : Number {
    for (element in list) {
        println(element)
    }
}
//多重边界，使用多个上界
fun <T> copy(t: T): T where T : Cloneable, T : Comparable<T> {
    return t.clone()
}
```

## 星号投影
- <*> 表示一个任意的类型，类似于 Java 的 <?>
- List<*> 读场景等价于 List<out Any?> 可读不可写
- List<*> 写场景等价于 List<in Nothing> 可写不可读

## 协变和逆变
- 协变：使用 out T 泛型通配符表示，表示上界为 T，用于读取 T 及其子类对象（保证类型安全，生产者角色）
- 逆变：使用 in T 泛型通配符表示，表示下界为 T，用于写入 T 及其父类的对象（保证能存入 T 类型，消费者角色）
- PECS 原则：Producer-Extends，Consumer-Super
```kotlin
//协变
//Producer<String> 可以赋值给 Producer<Any>
interface Producer<out T> {
    fun produce(): T  //只能返回 T，不能接受 T 作为参数
}
//逆变
//Consumer<String> 可以赋值给 Consumer<Any>
interface Consumer<in T> {
    fun consume(value: T)  //只能接受 T，不能返回 T
}
```

## 实化（具体化）类型参数
- 通过 inline 内联函数和 reified 关键字使得类型参数被实化
- 通常运用在序列化、反射等场景
```kotlin
//带 reified 类型参数的内联函数，Java 是无法直接调用的
inline fun <reified T> isType(value: Any): Boolean {
    return value is T
}
//JSON 解析
inline fun <reified T> parseJson(json: String): T {
    return Gson().fromJson(json, T::class.java)
}
inline fun <reified T> Context.startActivity()  {
    startActivity(Intent(this, T::class.java)) 
}
```