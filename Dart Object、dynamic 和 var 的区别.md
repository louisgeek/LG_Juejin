# Dart Object、dynamic 和 var 的区别
- Object、dynamic 和 var 是三种不同的变量声明方式，主要区别在于类型检查、类型推断和类型安全性等方面

## Object
- Object 类是 Dart 中所有类的基类，包括 int、String、List 和 null 等（都隐式继承自 Object）
- 编译时会进行类型检查
```dart
Object obj = "Hello";
obj.length; //编译错误，因为 Object 类没有 length 方法
(obj as String).length; //需要显式类型转换
```

## dynamic
- 动态类型，使用 dynamic 声明的变量可以在运行时具有任何类型的值
- 编译时关闭类型检查（运行时类型检查，相当于绕过编译器的静态类型检查，所以很可能会导致运行时错误）

## var
- 类型推断关键字，声明变量时让编译器根据初始值自动推断变量的类型，类型一旦确定后​​不可更改（所以本质是静态类型）
- 编译时会进行类型检查（编译时推断类型，之后静态检查）

## 总结
- Object 通用类型、dynamic 动态类型和 var 类型推断
- Object 可以存储任意类型，适合泛型等应用场景
- dynamic 需要谨慎使用，仅在必要时使用，比如 JSON 解析处理未知结构的数据
- var 优先推荐使用，能够简化代码书写而且保证类型安全