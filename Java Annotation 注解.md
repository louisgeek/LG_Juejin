# Java Annotation 注解
- 注解用于为代码添加额外描述信息（是一种代码级的说明、是一种代码 metadata 元数据），注解采用 @interface 作为关键字（很像一个接口，多了一个 @ 符号）
- 注解本质是一种标记，不直接影响代码逻辑，可以间接修改程序的行为和功能（配合编译器分析检查代码，提升代码逻辑严谨性）
- 比如 Spring 的 @Controller、@Autowired 实现依赖注入与路由映射，Retrofit 的 @GET、@POST 描述网络请求

## 内置注解
- @Override：用于标记方法是覆盖父类或接口的方法（编译器会检查方法签名是否匹配，避免错误的覆写）
- @Deprecated：用于标记过时（废弃的）的类、方法或字段等，编译器会提示警告（删除线提示）
- @SuppressWarnings：用于抑制编译器警告信息（黄色下划线），常用于忽略未使用泛型或资源未关闭的警告

@Override
```java
@Target(ElementType.METHOD) //指定注解适用的目标，此处代表只能用在方法上，ElementType.TYPE 类、接口（包括注解类型）、枚举，FIELD 字段（包括枚举常量），ANNOTATION_TYPE 用于元注解
@Retention(RetentionPolicy.SOURCE) //指定注解保留阶段，SOURCE 仅保留在源码中，编译后丢弃，CLASS 保留在字节码中，运行时不可见，RUNTIME 保留在运行时，可通过反射读取
public @interface Override {
}
```

@Deprecated
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE})
public @interface Deprecated {
    String since() default "";
    boolean forRemoval() default false;
}
```

## 元注解
- 用于定义其他注解的注解，控制自定义注解的行为和生命周期
- @Target：用于指定注解的目标，声明注解可以在什么地方使用（比如类、方法、字段等）
- @Retention：用于指定注解的生命周期和保留策略（SOURCE 源码阶段、CLASS 编译阶段（默认策略）或 RUNTIME 运行时）
- @Documented：声明注解信息将被包含在 Javadoc 中
- @Inherited：声明注解可以被子类继承（表示允许子类继承父类的注解）

## 自定义注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({TYPE, METHOD})
@Documented
public @interface MyCustomAnnotation {
    String value() default "default value";
    int id();
    String[] tags() default {};
}
```

```java
public class MyClass {
    @MyCustomAnnotation(id = 456)
    public void myMethod() {
        System.out.println("This method is annotated");
    }
}
```

## 注解的使用
- APT 注解处理器：编译时处理注解（基于注解动态生成代码，不需要用反射），处理 SOURCE 级别
- 插桩：编译后处理筛选（Eclipse AspectJ 修改字节码），处理 CLASS 级别
- 反射：运行时处理注解（利用反射动态获取），处理 RUNTIME 级别

