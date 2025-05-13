# Java 静态内部类和非静态内部类
- Java 嵌套类是指定义在其他类内部的类结构，包括静态嵌套类（静态内部类）和非静态嵌套类（非静态内部类，即内部类）
- 静态内部类不能访问外部类的成员，非静态内部类可以访问外部类的所有成员
- 静态内部类不会持有外部类的引用，而非静态内部类会持有外部类的引用

## Static Nested Class 静态内部类
- 静态内部类是用 static 关键字显式声明的嵌套类
- 无需外部类实例，可以直接实例化
- 适用于工具类、辅助类等场景
```java
public class Outer {
    private int testValue = 1;
    static class StaticNested {
        void testFun() {
            System.out.println("StaticNested testFun");
            //无法访问外部类的成员
            System.out.println(testValue); //编译报错：Non-static field 'testValue' cannot be referenced from a static contex
        }
    }
}
public class OuterTest {
    public static void main(String[] args) {
        Outer.StaticNested staticNested = new Outer.StaticNested();
        staticNested.testFun();
    }
}
```

## Non-static Nested Class 非静态内部类
- 即 Inner Class 内部类，是没有用 static 关键字声明的嵌套类
- 必须先实例化外部类，才能实例化内部类
- 隐式持有 Outer.this 引用
- 适用于事件监听、回调逻辑等场景
```java
public class Outer {
    private int testValue = 1;
    class Inner {
        void testFun() {
            System.out.println("Inner testFun");
            //可以访问外部类成员
            System.out.println(testValue); //编译正常
        }
    }
}
public class OuterTest {
    public static void main(String[] args) {
        Outer.Inner inner = new Outer().new Inner();
        inner.testFun();
    }
}
```