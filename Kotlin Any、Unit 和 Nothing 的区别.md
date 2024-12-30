# Kotlin Any、Unit 和 Nothing 的区别

## Any
- Any 是一个 open 的类，声明了 equals、hashCode 和 toString 方法，如果声明一个类但没有显式指定父类，那么它默认会隐式继承自 Any
- Kotlin 中所有非空类型的父类型都是 Any，而 Any? 表示所有可空类型的父类型，Any? 是 Any 的父类（或者说 Any? 等价于 Any 和 null 的合集）
- 在运行时，自动映射成 Java 的 java.lang.Object 类（Object 是 Java 中所有引用类型的父类型，Any 和 Any? 都相当于 Java 中的 Object 类）

## Unit
- Unit 是一个 object 单例类，只重写了 Any 的 toString 方法，通常用于函数声明中，当一个函数不需要返回值时（不返回任何有意义的值），可以将其返回类型声明为 Unit，Unit 默认可以省略（前提是函数体中最后一个表达式不是返回具体值的表达式），表示函数没有返回值
- Unit 的父类是 Any，而 Unit? 的父类是 Any?
- Unit 类似于 Java 的 void（不过 Unit 可访问到，但是 void 不能）

## Nothing
- Nothing 是一个构造函数声明为私有的没有实例的类（意味着是一个不能创建 Nothing 类型对象的类），用来表示没有值（不存在的值），Nothing? 唯一允许的值是 null
- 用于表示一个函数不会正常返回的情况，比如抛出异常、终止程序、进入一个无限循环或者用于标记那些不应该被执行到的代码块（比如 throw Exception 就相当于一个返回 Nothing 的表达式，函数结果永远不会返回）
- 可用来表示一个集合是空的，并且永远不会包含任何元素（比如 ```val emptyList: List<Nothing> = emptyList()```）
- Nothing 相当于 Java 中的 java.lang.Void 类

## 应用场景
- 常见的实现接口时候默认生成的代码 ```TODO("Not yet implemented")```，方法返回值写的是 Nothing，然后抛出一个异常

```kotlin
//按照前文，这里的 testAbstract 没写返回值，那么默认返回 Unit，但是方法里又写了 TODO 方法，它的返回值是 Nothing，所以 Nothing 可以代替原方法的返回类型（这里就是 Nothing 可以代替 Unit）
override fun testAbstract(test: String) {
    TODO("Not yet implemented")
}
```
- TODO 方法
```kotlin
/**
 * Always throws [NotImplementedError] stating that operation is not implemented.
 */

@kotlin.internal.InlineOnly
public inline fun TODO(): Nothing = throw NotImplementedError()

/**
 * Always throws [NotImplementedError] stating that operation is not implemented.
 *
 * @param reason a string explaining why the implementation is missing.
 */
@kotlin.internal.InlineOnly
public inline fun TODO(reason: String): Nothing = throw NotImplementedError("An operation is not implemented: $reason")

```

## 总结
- kotlin.Any::class == java.lang.Object::class 成立
- kotlin.Nothing::class == java.lang.Void::class 成立
- kotlin.Unit 类似于 Java 的 void