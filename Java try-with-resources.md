# Java try-with-resources
- try-with-resources 是 JDK 7 引入的一个语法糖（形成一种新的资源管理机制），是一种特殊的 try 语句，会在 try 块执行完毕后自动调用资源的关闭方法进行资源释放，其核心目标就是为了简化代码并确保资源在使用完后能被正确释放，避免因忘记关闭资源导致的内存泄漏或文件占用等问题
- 用于自动管理资源关闭，比如文件操作（FileInputStream、BufferedReader）、数据库连接、网络流（Socket、ServerSocket）和实现了 AutoCloseable 接口（包括其子接口 java.io.Closeable）的自定义资源
- 要求资源实现 java.lang.AutoCloseable 接口，编译器生成字节码，会自动插入资源关闭的逻辑，在 try 块执行完毕后（无论是否发生异常），自动调用每个资源的 close 方法将其关闭
- 异常抑制（压制）：编译器会处理异常抑制，如果在关闭资源时也发生了异常，原异常（try 块中的异常）会被保留，关闭资源的异常会被抑制并添加到原异常中，可以通过 Throwable#getSuppressed 获取被抑制的异常


## try-catch-finally
- 需要在 finally 块里手动关闭资源，容易遗漏
- 倘若 try 块里和 finally 块里均抛出了异常，那么 finally 块里的异常会覆盖 try 块里的异常，可能导致难以定位问题
```java
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    //业务逻辑
    String line;
    //读取操作
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try { 
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## try-with-resources
- 确保资源一定被关闭（即使发生异常）
- 资源在 try 代码块结束时自动调用 close 方法，无需手动关闭
- 可以省略 finally 块，减少代码嵌套
```java
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    //业务逻辑
    String line;
    //读取操作
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
    //关闭资源的异常会被抑制，作为 Suppressed Exception 附加
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Suppressed exception: " + suppressed);
    }
}
```

## 多资源
- 多个资源之间用分号隔开
- 多个资源按声明顺序的逆序关闭（后声明的先关闭）
```java
try (
    FileInputStream fis = new FileInputStream("input.txt");
    FileOutputStream fos = new FileOutputStream("output.txt")
) {
    //业务逻辑
    //使用 fis 和 fos 的代码
} catch (IOException e) {
    e.printStackTrace();
}
//关闭顺序：fos 先关闭，fis 后关闭
```

## 自定义资源
```java
public class MyResource implements AutoCloseable {

    public void fun01() {
        System.out.println("使用资源");
    }

    @Override
    public void close() throws Exception {
        //关闭资源的逻辑
        System.out.println("资源已关闭");
    }
}
//使用自定义资源
try (MyResource resource = new MyResource()) {
    //业务逻辑
    //使用 resource 的代码
    resource.fun01();
} catch (Exception e) {
    e.printStackTrace();
}
```