# Java Structural 结构型模式之 Proxy 代理模式
- 在不改变目标类的情况下通过代理类增加功能来扩展目标类的逻辑功能，真正的业务逻辑还是由目标类来实现
- 通常情况下代理类、目标类都需要实现同一个接口，代理类持有原接口对象，内部维护了目标类的引用
- 比较注重隔离限制，隔离调用类和被调用类之间的关系，划分职责和降低耦合
- 代理模式主要有静态代理和动态代理两种实现方式

## 示例
接口
```java
interface ICameraOperation {
    void takePicture();
}
```
目标类
```java
class CameraOperation implements ICameraOperation {
    @Override
    public void takePicture() {
        //拍照操作
        System.out.println("take picture");
    }
}
```
代理类
```java
//静态代理
class CameraOperationProxy implements ICameraOperation {
    private ICameraOperation target;

    public CameraOperationProxy(ICameraOperation target) {
        this.target = target;
    }

    @Override
    public void takePicture() {
        if (checkCameraPermission()) {
            //权限校验通过
           target.takePicture();
        }else{
          System.out.println("No permission to take picture");
        }
    }
}
```
使用
```java
//目标类
ICameraOperation cameraOperation = new CameraOperation();
//代理类
ICameraOperation proxyCameraOperation = new CameraOperationProxy(cameraOperation);
//
proxyCameraOperation.takePicture();
```

## 特点
- 代理模式可以将代理类与目标类逻辑分离，职责清晰，降低系统的耦合度，如果很多地方已经使用了目标类，可以不影响原有的业务逻辑
- 可以针对抽象接口进行编程，由于增加代理类无须修改源代码，比如三分库的业务增强等场景，可拓展性强，符合开闭原则
- 代理类可以对目标类的访问权限进行控制，比如前置的权限校验，校验不通过则可以不访问目标类，从而一定程度提升了系统的安全性
- 如果是静态代理，假设目标接口有很多方法，代理类可能需要考虑实现这些方法，也许会导致代码冗余
- 如果是动态代理，可以在运行期间动态生成代理对象，为多个被代理对象提供代理，代码的复用性更好
- 由于额外增加了代理对象，可能会导致多引入一定程度的性能开销




 