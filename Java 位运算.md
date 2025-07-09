# Java 位运算
- 计算机对二进制数据进行的运算（比如加、减、乘、除）被称为位运算，即对二进制数的每一位进行操作的运算

## 按位与 &
- and，对应位都为 1 时才为 1，否则为 0
- 应用：判断奇偶性，比如 (a & 1) == 1 为奇数，(a & 1) == 0 为偶数
```java
int a = 5;  //二进制: 0101
int b = 3;  //二进制: 0011
int result = a & b; //二进制: 0001，十进制: 1
System.out.println(result); //输出: 1
```

## 按位或 |
- or，对应位有一个为 1 就为 1，否则为 0
```java
int a = 5;  //二进制: 0101
int b = 3;  //二进制: 0011
int result = a | b; //二进制: 0111，十进制: 7
System.out.println(result); //输出: 7
```

## 按位异或 ^
- xor，按位异或，对应位不同时，结果才为 1，否则为 0
- 应用：快速交换变量，比如 a^=b; b^=a; a^=b;
```java
int a = 5;  //二进制: 0101
int b = 3;  //二进制: 0011
int result = a ^ b; //二进制: 0110，十进制: 6
System.out.println(result); //输出: 6
```

## 按位取反 ~
- inv，将每一位取反（0 变 1，1 变 0），取反运算不改变变量本身的值，仅返回临时结果
```java
int a = 5;  //二进制: 00000000 00000000 00000000 00000101
int result = ~a;  //二进制: 11111111 11111111 11111111 11111010（结果为补码形式，十进制: -6）
System.out.println(result);  //输出: -6
```

## 左移 <<
- shl，二进制位向左移动，右侧补 0
- 相当于乘以 2^n（左移、右移操作比乘除快）
```java
int a = 5;  //二进制: 0101
int result = a << 1; //二进制: 1010，十进制: 10
System.out.println(result); //输出: 10
```

## 右移 >>
- shr，二进制位向右移动，左侧补符号位（正数补 0，负数补 1）
- 相当于除以 2^n（并向下取整）
```java
int a = 8;  //二进制: 1000
int result = a >> 1; //二进制: 0100，十进制: 4
System.out.println(result); //输出: 4
```

## 无符号右移 >>>
- ushr，二进制位向右移动，左侧补 0（忽略符号位）
```java
int a = -5; //二进制: 11111111 11111111 11111111 11111011（使用补码存储负数）
int result = a >>> 1; // 二进制: 01111111 11111111 11111111 11111101，十进制: 2147483645
System.out.println(result); //输出: 2147483645
```


## 权限管理
- 通过位运算高效管理权限标志位
```java
public class Permission {
    public static final int READ = 1 << 0;    //00000001
    public static final int WRITE = 1 << 1;   //00000010
    public static final int EXECUTE = 1 << 2; //00000100
    public static final int DELETE = 1 << 3;  //00001000

    private int permissions;

    public void addPermission(int permission) {
        permissions |= permission;
    }

    public void removePermission(int permission) {
        permissions &= ~permission;
    }

    public boolean hasPermission(int permission) {
        return (permissions & permission) == permission;
    }

    public int getPermissions() {
        return permissions;
    }

    public static void main(String[] args) {
        Permission permission = new Permission();
        System.out.println(permission.getPermissions()); //0

        permission.addPermission(READ | WRITE);
        System.out.println(permission.getPermissions()); //3

        System.out.println(permission.hasPermission(READ));    //true
        System.out.println(permission.hasPermission(WRITE));   //true
        System.out.println(permission.hasPermission(EXECUTE)); //false
        System.out.println(permission.hasPermission(DELETE));  //false

        permission.removePermission(WRITE);
        System.out.println(permission.getPermissions()); //1

        permission.addPermission(EXECUTE | DELETE);
        System.out.println(permission.getPermissions()); //13

        System.out.println(permission.hasPermission(READ));    //true
        System.out.println(permission.hasPermission(WRITE));   //false
        System.out.println(permission.hasPermission(EXECUTE)); //true
        System.out.println(permission.hasPermission(DELETE));  //true
    }
}
```

## 数据简单压缩（加密）
- 比如用异或运算实现数据简单压缩算法
```java
int data = 31415926; //原始数据
int key = 666888;
int encrypted = data ^ key;
System.out.println(encrypted);  //30765950
int decrypted = encrypted ^ key; //恢复原始数据
System.out.println(decrypted);  //31415926
```



