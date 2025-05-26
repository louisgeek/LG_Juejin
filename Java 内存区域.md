# Java 内存区域
- 线程私有的内存区域：虚拟机栈、本地方法栈和程序计数器
- 线程共享的内存区域：堆和方法区（JDK 8 及以后称为元空间）

## Virtual Machine Stack 虚拟机栈
- 虚拟机栈用于存储方法调用的 Stack Frame 栈帧（每个方法在执行时会创建一个栈帧），栈帧包括局部变量（基本数据类型、引用数据类型对象的引用，不包括引用数据类型对象本身）、操作数栈、动态链接和方法返回地址（方法出口）等
- 自动管理：由编译器自动分配和回收内存（在编译时确定大小直接分配内存）
- 栈的分配速度较快，不过容量有限，StackOverflowError 栈溢出风险较高
- 常见错误：StackOverflowError：调用栈过深（如无限递归），OutOfMemoryError：栈帧过多导致栈空间不足（局部变量过多）

## Native Method Stack 本地方法栈
- 本地方法栈和虚拟机栈类似，只不过是虚拟机栈是为执行 Java 方法服务的，而本地方法栈为使用 Native 本地方法（比如用 C/C++ 代码实现的方法）服务的
- 常见错误：虚拟机栈一样也会抛出 StackOverflowError 和 OutOfMemoryError 异常

## Program Counter Register 程序计数器
- 程序计数器是一块较小的内存空间，用来记录当前线程所执行的字节码指令的行号指示器（保存当前程序的执行位置）
- 是唯一不会出现 OutOfMemoryError 的区域

## Heap 堆
- 也称为 GC 堆，是 JVM 中最大的内存区域，用于存储局部变量中的引用数据类型对象本身（包括数组）、全局变量中的实例变量、全局变量中的类变量（静态变量，JDK 8 以前存储在方法区中，而 JDK 8 及以后存储在堆中的）和字符串常量池（JDK 8 及以后）等
- 需 GC 管理：通过 new 关键字或反射分配内存创建对象（在运行时动态分配内存），依赖 Garbage Collector 垃圾回收器自动管理回收不再使用的内存
- 分代垃圾回收：将堆内存划分为 Young Generation 新生代和 Old Generation 老年代，新生代又进一步分为 Eden 伊甸园区和两个 Survivor 幸存者区，即 S0（From Survivor）和 S1（To Survivor），为了提高垃圾回收的效率，优化内存管理，避免频繁的全堆回收
- 堆分配速度相对较慢，堆的大小通常较大，适合存储对象、大型数据结构，堆内存不足时会抛出 OutOfMemoryError
- 常见错误：OutOfMemoryError: Java heap space（堆内存不足：内存泄漏或对象过多） 

## Method Area ​方法区（Metaspace 元空间）
- ​方法区（JDK 8 以前方法区被称为永久代，JDK 8 及以后方法区被称为元空间，使用本地内存，减少 OOM 风险）用于存储类的元数据信息（类名、父类、接口、字段、方法和构造器信息等）、运行时常量池和即时编译器编译后的代码等数据
- 需 GC 管理：类卸载时回收内存
- 常见错误：OutOfMemoryError: Metaspace（元空间容量不足：加载过多类或反射动态生成过多类）

```java
Object obj = new Object();
//obj 进栈（引用数据类型对象的引用）
//new Object() 进堆（引用数据类型对象本身）
//Object 类加载字节码文件，编译的 Object 类进方法区（元空间）
//于是形成了栈指向堆、堆指向方法区（Class 元数据）
```

## 编译时常量​、运行时常量和字符串常量
- 编译时常量：在编译时确定，初始化后不可修改，会直接嵌入到使用处（内联优化）
- 运行时常量：在运行时确定但不可变
- 字符串常量：相同内容的字符串共享同一引用，存储在字符串常量池
```java
final double PI = 3.14159; //编译时常量（字面量）
final String s0 = "test"; //编译时常量
final int value = getVal(); //运行时常量（需要运行时计算）
//
final String s1 = "Hello"; //自动放入字符串常量池
final String s2 = "Hello"; //直接返回字符串常量池中的引用
final String s3 = new String("Hello"); //在堆中生成新对象，不放入字符串常量池，与字符串常量池中的 "Hello" 不是同一个对象
final String s4 = new String("Hello"); //在堆中生成新对象
final String s5 = new String("Hello").intern(); //通过 intern 方法手动放入字符串常量池
System.out.println(s1 == s2); //true（指向字符串常量池中同一个 "Hello" 对象）
System.out.println(s1 == s3); //false
System.out.println(s3 == s4); //false（不同对象）
System.out.println(s1 == s5); //true
```

## 总结
- 虚拟机栈：存储方法调用栈帧（包括局部变量、方法返回地址等数据），支持 Java 方法的调用执行
- 本地方法栈：支持 Native 本地方法的调用执行
- 程序计数器：支持字节码指令的执行，记录当前程序的执行位置
- 堆：存储引用数据类型对象本身（包括数组）、全局变量和字符串常量池等数据，分新生代（又细分为伊甸园区和 S0 和 S1 两个 Survivor 幸存者区）和老年代
- 方法区（​元空间）：存储类元数据信息、运行时常量池等
- 局部变量：基本数据类型存储在栈中，引用数据类型对象的引用是存储在栈中，而引用数据类型对象本身是存储在堆中
- 全局变量中的实例变量：存储在堆中（类对象可能包含基本数据类型和引用数据类型作为一个整体存储在堆中），每个对象都有自己独立的实例变量副本
- 全局变量中的类变量（静态变量）：存储在堆中（JDK 8 以前存储在方法区中，而 JDK 8 及以后存储在堆中的），所有对象共享同一个类变量副本