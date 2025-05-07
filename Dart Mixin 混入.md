# Dart Mixin 混入
- Mixin 是一种实现代码复用的机制，允许宿主类通过 with 关键字“混入”封装好的功能模块（即复用 Mixin 的方法和属性），而不是通过传统继承或实现接口的方式，解决了 Dart 单继承的限制（可以实现类似 “多继承” 的效果），使得代码更加灵活且避免了传统多继承的复杂性
- Mixin 可以包含方法和属性，但不能被实例化，其目的就是为了提供可复用的代码，专门用于被其他类 “混入”
- Mixin 可以定义抽象方法，强制宿主类实现
- Mixin 通常需要以 Mixin 结尾，以提高代码可读性
- 每个 Mixin 应该保持单一职责，提高代码的可维护性

```dart
//通过 mixin 关键字声明
mixin FlyMixin {
  String info = "来自 FlyMixin";
  void fly() => print('Flying');
}

class BirdClass with FlyMixin {}

void main() {
  //
  var birdClass = BirdClass();
  print('birdClass info=${birdClass.info}');
  birdClass.fly();
  //--------- 打印 -------------
  //birdClass info=来自 FlyMixin
  //Flying
}
```

## 多个 Mixin
- 通过 `with AMixin, BMixin` 混入多个 Mixin，用逗号分隔
- 如果基类中和多个 Mixin 中都有同名成员（方法或属性），最后混入的 Mixin 的成员优先级最高（后混入 Mixin 的成员会覆盖前者）

```dart
class BaseClass {
  void test() => print("test BaseClass");
}

mixin AMixin {
  void test() => print("test AMixin");
}
mixin BMixin {
  void test() => print("test BMixin");
}

class CClass with AMixin, BMixin {}

class DClass with BMixin, AMixin {}

//类（非 Mixin）需要位于 Mixin 之前
class EClass extends BaseClass with AMixin, BMixin {}

void main() {
  var cClass = CClass();
  cClass.test();
  //
  var dClass = DClass();
  dClass.test();
  //
  var eClass = EClass();
  eClass.test();
  //--------- 打印 -------------
  //test BMixin
  //test AMixin
  //test BMixin
}

```

## 限制 Mixin 应用范围
- 可以通过 on 关键字限定 Mixin 能够混入的类类型（限制只能被指定类型的类混入），要求宿主类必须继承自指定的基类，以确保类型安全

```dart
mixin AMixin { }
//相当于
mixin AMixin on Object { }
```

```dart
class AnimalClass {
  String info = "来自 AnimalClass";
}

mixin FlyMixin on AnimalClass {
  void fly() => print('Flying');
}

class BirdClass extends AnimalClass with FlyMixin {}

class CarClass with FlyMixin {} //编译报错：'FlyMixin' can't be mixed onto 'Object' because 'Object' doesn't implement 'AnimalClass'. 
```

## 重写 Mixin 方法
- 宿主类可以通过 `@override` 重写 Mixin 方法，并通过 super 调用 AMixin 原始方法

```dart
mixin AMixin {
  void test() {
    print("test AMixin");
  }
}

class BClass with AMixin {
  @override
  void test() {
    super.test(); //调用 AMixin 的 test 方法
    print("test BClass");
  }
}

void main() {
  var bClass = BClass();
  bClass.test();
  //--------- 打印 -------------
  //test AMixin
  //test BClass
}
```

## mixin class 混入类
- 使用 mixin class 同时声明混入和类，该类既可以当作常规类使用，又可以当作混入使用（比如 ChangeNotifier）
- 应用于类或混入的任何限制也适用于混入类（比如不能使用 on）

```dart
mixin class A {}

class B with A {}

class C extends A {}
```

## 总结
- Mixin 只能通过 with 关键字将 Mixin 的功能添加到目标宿主类中，避免了传统多继承的复杂性，同时保持代码的模块化和可维护性
- 可以通过多个 Mixin 灵活组合使用，将日志记录、网络请求、数据校验和权限校验等通用功能逻辑分别封装成一个个 Mixin，避免代码重复
- 可以使用 on 限制 Mixin 的应用范围
- 如果多个 Mixin 或父类都定义了同名方法，会按线性化顺序选择最后混入的 Mixin（最后混入的优先级最高，变量的解析规则和方法一致）