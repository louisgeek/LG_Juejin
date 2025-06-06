# Java 泛型
- Java 泛型（参数化类型）是编译期特性，允许在定义类（包括接口）和方法时使用类型参数，将类型参数化（具体的类型泛化，在使用时动态指定具体类型），用于在编译阶段进行类型约束检查，减少类型转换错误，增强代码的可读性、稳定性、可复用性和可维护性
- Java 泛型是通过类型擦除机制实现的，所以有人也称之为伪泛型
- 类型安全：在编译阶段进行类型安全检查，避免运行时出现 ClassCastException 异常
- 消除强制类型转换：从集合取元素时自动类型转换，减少显式类型转换代码，实现开发效率和可读性的提升
- 代码复用：通过泛型类或者方法编写可复用的通用逻辑代码
- T 是 Type Parameter 类型参数，可以定义多个（比如 <T, S> 多参数声明）

## 命名约定
```java
E：Element，主要用于集合元素
K：Key，主要用于 Map 的 Key
V：Value，主要用于 Map 的 Value
N：Number，主要用于数字
T：类型，通常用于表示第一个泛型类型参数
S：类型，通常用于表示第二个泛型类型参数
U：类型，通常用于表示第三个泛型类型参数
V：类型，通常用于表示第四个泛型类型参数
```

## 类型擦除
- 编译器在编译阶段会擦除所有泛型类型信息，将泛型的类型参数替换为边界类型（比如 T 替换成 Object、`T extends Number` 替换成 Number），运行时仅保留 Raw Type 原始类型（比如 `List`，生成的字节码中不包含泛型的具体类型参数）
- 目的是为了兼容旧版本的 Java 代码（即非泛型代码）
- 所以通过反射无法直接获取泛型类型信息，需通过 ParameterizedType 等方式间接获取泛型信息

## 桥接方法
- 当子类继承泛型类时，在覆写泛型方法时，编译器会生成桥接方法以保持多态性

## 边界约束
```java
//约束上界
public <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
//约束下界
public <T super Integer> void process(T obj) { }
//多重边界，T 应该是 Number 类的子类型，并且必须实现 Comparable 接口
public static <T extends Number & Comparable<T>> T maximum(T x, T y, T z) { }
```

## 类型通配符
- <?> 代表无界通配符，表示未知的类型
- <? extends T> 代表上界通配符，表示类型参数可以是 T 或其子类
- <? super T> 代表下界通配符，表示类型参数可以是 T 或其父类
- List<?> 表示一个未知类型的 List（通配符类型，可以是 List<String>、List<Integer> 或其他任何类型的 List），不能添加任何元素（除 null 外）

## 协变和逆变
- 协变：使用 ? extends T 泛型通配符表示，表示上界为 T，用于读取 T 及其子类对象（保证类型安全，生产者角色）
- 逆变：使用 ? super T 泛型通配符表示，表示下界为 T，用于写入 T 及其父类的对象（保证能存入 T 类型，消费者角色）
- PECS 原则：Producer-Extends，Consumer-Super
```java
//协变
public void getOutAnimal(List<? extends Animal> list) {
    for (Animal animal : list) { 
        //读取数据
    }
}
//逆变
public void putAnimalIn(List<? super Dog> list) {
    //写入数据
    list.add(new Dog()); //可以安全写入 Dog 或其子类
}
```