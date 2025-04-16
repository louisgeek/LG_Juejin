# Dart final 和 const 的区别
- final 和 const 都是用于声明不可变的变量的关键字
- final 和 const 声明的变量在赋值后都是不可变的（只能赋值一次，不能被重新赋值），但 const 的限制更加严格（相当于 const 包含了 final 的特性，所以 final 和 const 不能一起使用）
```dart
final const x = 10; //编译报错（不能一起使用）
```

## final 运行时常量
- 可在声明时或构造函数中进行赋值，在函数中声明的，必须在声明时赋值，而在类中声明的，可以在构造函数中进行赋值
- 但对象的内部状态可能可变（比如 list 列表里的元素）
- final 支持延迟初始化（通过 late 关键字）
- 可能多次创建相同值的对象
```dart
//声明时初始化
final PI = 3.14159;
final name = "Dart";
final now = DateTime.now();
final list = [1, 2, 3];
//延迟初始化（需要 late 配合）
late final int age;
//
list.add(4); //允许（list 引用不变，但内容可改变）
list = [5];  //编译报错（引用不可变）
```

## const 编译时常量
- 必须在声明时立即赋值，且值在编译时已知
- 编译器会在编译阶段将其替换为具体值（编译时内联），多次使用同一个 const 变量时，内存中只会存在一份实例（同一常量值共用同一个实例，指向同一内存地址）
```dart
//声明时初始化（字面量、常量表达式）
const PI = 3.14159; //字面量
const name = "Dart";
const now = DateTime.now(); //编译报错
const list = [1, 2, 3];
const age = 16 + 2; //常量表达式
//
list.add(4); //编译报错
```

## 总结
- final 变量的值可以在运行时确定，而 const 变量的值必须在编译时就能计算出来（不能在运行时动态计算）
- final 强调引用的不可变性，const 强调值和内存的不可变性