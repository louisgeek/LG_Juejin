# Java Array




## Array 和 List 互转
Array -> List
```java
String[] array = new String[] {"1", "2", "3"};
//返回的是 java.util.Arrays.ArrayList 不是 java.util.ArrayList 所以不能直接操作 add 等方法 
List<String> listTemp = Arrays.asList(array);
List<String> list = new ArrayList<String>(listTemp);
list.add("4");
list.add("5");
list.add("6");
```

List -> Array
```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
//List 转成数组
String[] array = list.toArray(new String[list.size()]);
```