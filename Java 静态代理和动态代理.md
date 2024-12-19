# Java 静态代理和动态代理
- 代理模式主要有静态代理和动态代理两种实现方式

## 静态代理
- 指程序在运行之前所需要的代理类就已经被创建好了
- 通常情况下代理类、目标类都需要实现同一个接口
- 代理类内部维护了目标类的引用，在不改变目标类的情况下通过代理类增加功能来扩展目标类的逻辑功能，真正的业务逻辑还是由目标类来实现，代理类只是调用目标类的相关方法，符合开闭原则
- 一般需要为每个目标类都创建代理类和接口，可能导致类的数量大大增加
- 通常情况下接口一旦修改，代理类和目标类都需要修改，代码耦合度较高

### 示例
接口
```java
public interface ILogin {
    void doLogin();
}
```

目标类
```java
public class UserLogin implements ILogin {
  @Override
  public void doLogin(){
      System.out.print("用户登录逻辑");
  }   
}
```

代理类
```java
public class UserLoginProxy implements ILogin {
  private UserLogin mUserLogin;
  public UserLoginProxy() {
    mUserLogin = new UserLogin();
  }
  @Override
  public void doLogin(){
    //...
    System.out.print("登录前逻辑");
    //真正的业务逻辑还是由目标类来实现
    mUserLogin.doLogin();
    System.out.print("登录后逻辑");     
    //...
  }
}
```

使用
```java
	public static void main(String[] args){
     //
     ILogin iLogin = new UserLoginProxy();
     iLogin.doLogin();
	}
```


## 动态代理
- 动态代理指在程序运行时通过反射机制等技术动态创建出代理对象
- 基于接口的动态代理，通常通过 java.lang.reflect.Proxy 类和 java.lang.reflect.InvocationHandler 接口实现动态代理
- 基于类的动态代理，比如使用 CGLIB（Code Generation Library，一个强大的 Java 字节码生成库）
- InvocationHandler 调用处理器，是一个接口，有个 invoke 方法
- 通过使用 Proxy#newProxyInstance 来创建指定接口的代理实现类，当调用代理对象任何方法时都会调用 InvocationHandler#invoke 方法，可以在这个方法中拿到传入的参数，注解等
- 只能实现基于接口的动态代理，因为 Java 是单继承的，它在动态生成 $ProxyX 代理类的时候已经继承 Proxy 类了
- 可以减少类的数量，降低工作量，灵活性高，可以在运行时动态地为不同的目标类创建代理类对象
- 减少对业务接口的依赖，降低耦合，便于后期维护，可以用于实现 AOP 面向切面编程，比如在某个逻辑功能前后添加日志记录、监控性能和权限检查控制等操作
- 由于因为使用了反射，所以会在运行时会消耗一定的性能

```java
private InvocationHandler mInvocationHandler = new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            //target 是目标类对象
            Object result = method.invoke(target,args);
            //
            return result;
        }
    };
//方式1
//创建代理类的 Class 对象，参数 (ClassLoader 类加载器,Class<?>)
Class<?> proxyClass = Proxy.getProxyClass(Toast.class.getClassLoader(), Toast.class);
//通过反射实例化代理对象
Toast toastProxy = (Toast) proxyClass.getConstructor(InvocationHandler.class).newInstance(mInvocationHandler);
//方式2，是方法1的简化版，newProxyInstance 内部代码逻辑和方法1基本一致
//直接创建代理对象，参数 (ClassLoader 类加载器,Class<?>[],InvocationHandler)
Toast toastProxy2 = (Toast) Proxy.newProxyInstance(Toast.class.getClassLoader(),new Class<?>[]{Toast.class}, mInvocationHandler);
```

### 示例
接口
```java
public interface ICalculator {
    int add(int a, int b);
    int minus(int a, int b);
}
```
目标类
```java
public class Calculator implements ICalculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
    @Override
    public int minus(int a, int b) {
        return a - b;
    }
}
```
动态代理的处理类
```java
  public class CalculatorInvocationHandler implements InvocationHandler {
      private Object target;
      public CalculatorInvocationHandler(Object target) {
          this.target = target;
      }
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          // 在方法调用前可以添加一些逻辑
          Object result = method.invoke(target, args);
          // 在方法调用后也可以添加一些逻辑
          return result;
      }
  }
```
使用
```java
//目标类
ICalculator calculator = new Calculator();
//代理类
ICalculator proxyCalculator = (ICalculator) Proxy.newProxyInstance(
        calculator.getClass().getClassLoader(),
        calculator.getClass().getInterfaces(),
        new CalculatorInvocationHandler(calculator)
);
```