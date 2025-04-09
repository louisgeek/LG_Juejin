# Java 类加载机制
- 类加载机制是 JVM 动态加载类的核心机制，指在程序运行期间，类从磁盘等地方加载到内存中的过程，其负责将编译后的字节码文件（.class 文件）加载到内存中并转换为可执行的结构
- 类加载机制包括 Loading 加载、Linking 链接和 Initialization 初始化三个关键核心阶段
- Lazy Loading 懒加载：类只有在第一次被使用时才会被加载（按需加载），而不是一次性全部加载，这样可以减少内存占用，提高程序的启动速度
- Parent Delegation Model 双亲委派模型：通过双亲委派确保类的安全性和唯一性 

## 加载
- 将 .class 文件的二进制数据读入内存（通过类的全限定名获取二进制字节流），并创建生成对应的 java.lang.Class 对象

## 链接
- Verification 验证：检查字节码是否符合规范，确保正确性和安全性
- Preparation 准备：为类变量分配内存并设置默认初始值
- Resolution 解析：将符号引用（如类名、方法名）转换为直接引用

## 初始化
- 执行类的初始化代码，包括静态初始化块和静态变量的赋值操作

## ClassLoader 类加载器
- Bootstrap ClassLoader 启动（引导）类加载器：负责加载 /jre/lib 路径下的核心类库（比如 java.lang、java.util 等），使用 C++ 实现的，不继承 java.lang.ClassLoader，在 Java 中无法直接获取，不可直接访问
- Extension ClassLoader 扩展类加载器：负责加载 jre/lib/ext 路径下的扩展类库
- Application ClassLoader 应用程序类加载器（System ClassLoader 系统类加载器）：负责加载用户 classpath 类路径下的类
- Custom ClassLoader 自定义类加载器：可以继承 java.lang.ClassLoader 并重写 findClass 方法实现，通常用于加载非标准路径中的类（比如动态代理、加密文件中加载和网络下载后加载等）

## 双亲委派模型
- 当一个类加载器在查找 Class 时候，先判断该类是否已经加载（避免重复加载，判断已加载就直接读取已加载的 Class），如果没有，它首先不会自己去加载这个类，而是委派父类加载器去查找，递归向上委托直到最上层 Bootstrap ClassLoader 启动类加载器，如果找到就返回，如果还没找到才继续往下找，最后还没找到就回到了自身类加载器去尝试查找加载
- 不同类加载器负责不同范围的类
- 安全性：核心类库（比如 java.lang.String）由启动类加载器加载，防止核心类被自定义类冒充篡改
- 唯一性：一个类只能由一个类加载器加载，同一个类不会被加载多次，避免重复加载，保证加载的类的唯一性

```java
//父子类加载器之间是 Composition 组合关系，而不是 Inheritance 继承关系
Custom ClassLoader -> Application ClassLoader -> Extension ClassLoader -> Bootstrap ClassLoader
```

## 总结
- Java 类加载机制就是把类的字节码文件动态加载到内存中，并且对其进行加载、链接和初始化等操作
- 当一个类加载器收到加载请求时，优先委派父类加载器处理，仅在父类加载器无法加载时，才由当前类加载器尝试加载，确保核心类库的优先加载，保证核心类安全
- Java 类加载机制确保了 Java 程序的灵活性、安全性和高效性










