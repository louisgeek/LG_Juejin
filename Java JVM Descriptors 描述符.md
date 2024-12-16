
## Java JVM Descriptors 描述符
- 在 JVM 中定义了两类描述符分别是 Field Descriptors 字段描述符和 Method Descriptors 方法描述符

## Field Descriptors 字段描述符
- 是用于唯一标识 Java 类的字段类型的字符串符号，可以通过字段描述符来准确地访问 Java 类对象中各个字段来获取或者设置字段的值

### 基本数据类型
```java
Z：boolean //特殊，不是 B
B：byte
C：char
S：short
I：int
J：long //特殊，不是 L
F：float
D：double
V：void
```

### 引用数据类型
- 用 L 开头，以 ; 结尾，中间是类的全限定名，其中 . 替换为 /，L 是 Class 的简写（也有说是 Language 的缩写），表示类、接口和枚举等
```java
Ljava/lang/String; //表示 java.lang.String 类型
Ljava/lang/Object; //表示 java.lang.Object 类型
Ljava/util/HashMap$Entry; //表示 java.util.HashMap.Entry 类型（内部类用 $ 分隔）
```
- 用 [ 开头表示数组类型 
```java
[I  //表示 int[]
[Ljava/lang/String; //表示 String[] 类型的一维数组
[Ljava/lang/Object; //表示 Object[] 类型的一维数组
[[I //表示 int[][] 类型二维的数组
[[Ljava/lang/String; //表示 String[][] 类型二维的数组
```
 

## Method Descriptors 方法描述符
- 方法描述符是对 Java 方法的参数和返回值类型的一种编码描述
- 用 ( 和 ) 描述包含零个或多个参数的方法
```java
()V //表示一个返回值是 void 类型的无参方法，比如 void test();
(I)V //表示一个返回值是 void 类型且带一个 int 类型参数的方法，比如 void test(int a);
(II)V //表示一个返回值是 void 类型且带两个 int 类型参数的方法，比如 void test(int a,int b);
()Ljava/lang/String; //表示一个返回值是 java.lang.String 的无参方法，比如 String test();
(Ljava/lang/String;Ljava/lang/String;)Z //表示一个返回值是 boolean 类型且带两个 java.lang.String 参数的方法，比如 boolean test(String a,String b);
([B)I //表示一个返回值是 int 且带一个 byte[] 数组参数的方法，比如 int test(byte[] bytes);
```

