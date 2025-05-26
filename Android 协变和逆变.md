# Android 协变和逆变
- Dog 是 Animal 的子类
- 型变分为：Invariant 抗变（不变）、Covariance 协变和 Contravariance 逆变
- 抗变：`List<Dog>` 和 `List<Animal>` 没有任何继承关系，直接赋值会编译报错，所以抗变严重制约了程序的灵活性
- 协变：`List<Dog>` 是 `List<Animal>` 的子类型，即子类型可以赋值给父类型
- 逆变: `List<Animal>` 是 `List<Dog>` 的子类型，即父类型可以赋值给子类型

## 协变
- 适用于 Producer 生产者角色的泛型类或接口，只能读取数据，不能写入
- 类中不能有接受该类型参数作为方法参数的成员，只能返回该类型
- Java 的 `List<? extends Animal>` 对应 Kotlin 的 `List<out Animal>`

```java
public void getOutAnimal(List<? extends Animal> list) {
    for (Animal animal : list) { 
        //读取数据
    }
}
```

```kotlin
//Producer<String> 可以赋值给 Producer<Any>
interface Producer<out T> {
    fun produce(): T  //只能返回 T，不能接受 T 作为参数
}
```

## 逆变
- 适用于 Consumer 消费者角色的泛型类或接口，只能写入数据，不能读取
- 类中不能有返回该类型参数的成员，只能接受该类型作为参数
- Java 的 `List<? super Dog>` 对应 Kotlin 的 `List<in Dog>`

```java
public void putAnimalIn(List<? super Dog> list) {
    //写入数据
    list.add(new Dog()); //可以安全写入 Dog 或其子类
}
```

```kotlin
//Consumer<String> 可以赋值给 Consumer<Any>
interface Consumer<in T> {
    fun consume(value: T)  //只能接受 T，不能返回 T
}
```

## 总结
- 协变：在 Java 中使用 ? extends T 表示，在 Kotlin 中使用 out T 表示，表示上界为 T
- 逆变：在 Java 中使用 ? super T 表示，而在 Kotlin 中使用 in T 表示，表示下界为 T
- PECS 原则：Producer-Extends，Consumer-Super
- Producer -> output -> out
- Consumer -> input -> in