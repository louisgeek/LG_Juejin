# Java 知识点

## 面向对象编程的最基本思想
类与对象：可以创建一类事物，通过对类的封装，在适当的时候可以创建这些类的实例



Java 四种线程池


## 线程池的参数



## JVM、classloaders、classreflect以及垃圾回收的基本工作机制



## 动态代理的实现


HashMap1.7和1.8的实现原理

GCRoot


## String 字符串常量


接口和抽象类的区别 接口中可以有属性么

CPU核心数，线程数，时间片轮转机制解读
synchronized、Lock、volatile、ThreadLocal如何实现线程共享
Wait,Notify/NotifyAll，Join方法如何实现线程间协作

Callbale、Future和FutureTask源码解读
线程池底层实现分析
线程池排队机制
手写线程池实战
Executor框架解读实战



Java的双亲委派


HashMap讲一下，数据结构、hash过程、扩容、加载因子为何是0.75等

 

## ArrayList
- 内部 Object[] elementData 数组加了 transient
- 默认初始容量为 10，每次扩充容量扩充为原来的1.5倍
## Vector（不推荐使用）
- 内部 Object[] elementData 数组不加 transient
- 当 capacityIncrement 扩容增量大于0时，扩充为原有容量加上capacityIncrement的容量值。否则采取在原有容量基础上扩充为原来的1.5倍

ArrayList提供的writeObject和readObject方法来实现定制序列化，而Vector只是提供了writeObject方法，并没有完全实现定制序列化
## ArrayList
- 随机访问（get和set操作）允许直接按序号索引元素，所以效率高，但删除、插入（add和remove操作）涉及数组元素移动等操作，所以效率较低
 
## LinkedList
- 随机访问（get和set操作）需要按序号索引数据需要进行前向或后向遍历，所以效率低，但删除、插入（add和 remove 操作）只需要记录本项的前后项即可，所以效率较高
- 查找操作需要从头开始查找元素，速度较慢







## Java

基础
Java 属于编译型语言还是解释性语言

Java 语言的运行机制是 Java 代码编译后生成一种特殊的 class 文件，这个格式的文件只有 Java 虚拟机才能识别，所以这个 Java 虚拟机就充当了解释器的角色，将 class 文件解释成计算机可直接识别的二进制数据后进行执行，所以 Java 属于解释性语言，Python 和 JavaScript 也属于解释性语言，C/C++ 属于编译型语言





 




单例模式：双重校验锁，volatile 关键字 

- ### volatile 关键字

  1. 保证可见性,不保证原子性

  2. 禁止指令重排序

  3. 不缓存,每次都是从主存中取

     https://www.cnblogs.com/zhengbin/p/5654805.html

  ### hashmap 原理



### 红黑树



二叉树



### jvm内存



- Java堆





### 类加载器

Android平台上虚拟机运行的是Dex字节码,一种对class文件优化的产物,传统Class文件是一个Java源码文件会生成一个.class文件，而Android是把所有Class文件进行合并，优化，然后生成一个最终的class.dex,目的是把不同class文件重复的东西只需保留一份,如果我们的Android应用不进行分dex处理,最后一个应用的apk只会有一个dex文件。
 Android中常用的有两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader。区别在于调用父类构造器时，DexClassLoader多传了一个optimizedDirectory参数，这个目录必须是内部存储路径，用来缓存系统创建的Dex文件。而PathClassLoader该参数为null，只能加载内部存储目录的Dex文件。所以我们可以用DexClassLoader去加载外部的apk。





 



### 线程池

- 对线程统一管理，统一调度的工具，可以复用已存在的线程，减少线程的创建和销毁的开销，降低消耗，增加速度

- 能够控制并发的线程数的上限，既能够降低资源的消耗浪费，也一定程度上避免了系统处理时候的堵塞，提高了系统资源的合理利用率，增强了对线程的管理性

- 能够实现定时周期任务

  ​	

### 内置线程池

FixedThreadPool

SingleThreadPool

ScheduleThreadPool

CachedThreadPool

### 自定义线程池





注解是什么，元注解又是什么？

注解：是针对代码的一种标注、描述、解释，可以针对代码来配置一些功能，能够用来自动生成 Java 代码



元注解：修饰注解的注解就叫元注解

- @Target
- @Retention
- @Inherited
- @Documented



### effectively final

- 事实上地、实际上地

java 8 后，匿名内部类、Lambda 表达式访问的局部变量不需要再显示声明成 final 了

[为什么必须是final的呢？ (cuipengfei.me)](https://cuipengfei.me/blog/2013/06/22/why-does-it-have-to-be-final/)





### effective java

- 有价值的，有效的



- google

  - guava

    https://github.com/google/guava

    Google Guava官方教程（中文版）http://ifeve.com/google-guava/

- java 集合

JAVA集合类汇总http://www.cnblogs.com/leeplogs/p/5891861.html

- Float Double

JAVA float double数据类型保留2位小数点5种方法
http://www.cnblogs.com/simpledev/p/4765834.html

Float或Double浮点型计算导致小数点后面显示过长数据的解决方法

 http://blog.csdn.net/bbg_0622/article/details/5616094

java float、double精度研究（转）
http://www.cnblogs.com/chenfei0801/p/3672177.html

Java浮点数float，bigdecimal和double精确计算的精度误差问题总结
http://www.cnblogs.com/wangyt223/p/6210916.html



- 零基础学Java 04

java 类名和文件名必须一致

- 零基础学Java 05

  cmd --> dir 命令



- 零基础学Java 10

int 区间 -2^31 ~ 2^31-1  



- 零基础学Java 11

表达式 语句 代码块

整数的字面值默认是 int

关键字

标识符

运算符

字面值

数据类型

变量

表达式 

语句  ;

代码块  {  }



- 零基础学Java 12

bit  和 byte

二进制的位 叫 bit

byte 是计算机存储单元

整数类型

byte -128 ~ 127

short (2 byte) -32768 ~ 32767

int (4 byte)

long (8 byte)

浮点（小数）类型

float (4 byte)

double (8 byte)

布尔类型

1 byte

char 类型 --- 单引号

2 byte



- 零基础学Java 13



算术运算符

比较运算符

布尔运算符



！ 非 not

& 且 and

&& 且且 and and 

| 或 or

|| 或 或 or or 






# Kotlin

### Kotlin 的协程是什么？




广义上的协程：和线程类似，也是一种解决异步任务的方案

Kotlin 的协程：可以理解是对线程的一种封装，一种可以让异步任务的代码写成一种类似同步的形式，可以看作是一个线程封装框架，官方称为一种轻量级线程

- 可以解决异步任务代码的嵌套
- 必须依赖线程存在，但是可以在不同的线程之间切换
- 由开发者管理，不需要操作系统进行调度和切换



### Kotlin data class 没有无参构造函数的问题如何解决？












