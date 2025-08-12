# Java UnmodifiableList 和 AbstractImmutableList 的区别
- java.util.Collections.UnmodifiableList
- java.util.ImmutableCollections.AbstractImmutableList


## UnmodifiableList
- 本质上是对一个原始可变列表的包装，UnmodifiableList 只读不允许增删改操作
- 不保证线程安全，如果原始列表被修改，此列表会实时反映这些变化
```java
public static void main(String[] args) {
    //可变列表
    List<String> mutableList = new ArrayList<>();
    mutableList.add("A");
    mutableList.add("B");
    //UnmodifiableList 不可修改列表
    List<String> unmodifiableList = Collections.unmodifiableList(mutableList);
    //unmodifiableList.add("C"); //运行时报错：UnsupportedOperationException
    //可修改原始可变列表
    mutableList.add("D");
    System.out.println(mutableList); //[A, B, D]
    System.out.println(unmodifiableList); //[A, B, D]
}
```

## AbstractImmutableList
- ImmutableCollections 是 Java 9 引入的可以用于创建不可变集合的工具类
- 线程安全，元素固定，不能修改，与原列表完全隔离，修改原列表不影响此列表
- 不支持 null 元素
```java
public static void main(String[] args) {
    //不可变列表
    List<String> immutableList = List.of("A", "B");
    //immutableList.add("C"); //运行时报错：UnsupportedOperationException
    System.out.println(immutableList); //[A, B]
}
```

```java
public static void main(String[] args) {
    //可变列表
    List<String> mutableList = new ArrayList<>();
    mutableList.add("A");
    mutableList.add("B");
    //不可变列表
    List<String> immutableList = List.copyOf(mutableList);
    //immutableList.add("C"); //运行时报错：UnsupportedOperationException
    //修改原始可变列表
    mutableList.add("D");
    System.out.println(mutableList); //[A, B, D]
    System.out.println(immutableList); //[A, B]
}
```

## UnmodifiableList 应用
- 约束全局修改，让修改方法严格控制在类本身内部
```java
public class DeviceManage {

    private String userId;

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    private List<String> deviceList = new ArrayList<>();

    //仅允许内部修改，不允许外部直接修改
    public void addDevice(String device) {
        deviceList.add(device);
    }

    //仅允许内部修改，不允许外部直接修改
    public void removeDevice(String device) {
        deviceList.remove(device);
    }
    //deviceList 仅对外提供 get 方法，不提供 set 方法
    public List<String> getDeviceList() {
        //而且返回的是不可修改的列表
        return Collections.unmodifiableList(deviceList); 
    }
}

public static void main(String[] args) {
    DeviceManage deviceManage = new DeviceManage();
    deviceManage.addDevice("device 01");
    deviceManage.addDevice("device 02");
    deviceManage.addDevice("device 03");
    deviceManage.removeDevice("device 02");
    //deviceManage.getDeviceList().add("device 04"); //运行时报错：UnsupportedOperationException
    for (String device : deviceManage.getDeviceList()) {
        //device 01
        //device 03
        System.out.println(device);
    }
}
```