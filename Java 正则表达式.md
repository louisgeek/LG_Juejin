# Java 正则表达式
- 正则表达式又称为规则表达式，是一种强大的文本模式匹配工具，用于对字符串进行搜索、替换和验证等操作

```java
. 匹配任何字符（除了 \n 换行符、\r 回车符），比如 a.c 匹配 abc
^ 匹配字符串或行的开始
$ 匹配字符串或行的结束
| 表示或关系，比如 a|b 匹配 a 或 b
() 用于分组，匹配括号内整个表达式并可以引用

[abc] 字符集，匹配 a、b 或 c 中的任意一个字符
[^abc] 排除型字符集，匹配除 a、b、c 之外的任意一个字符

[a-z] 匹配小写字母 a 到 z 中的任意一个字符
[A-Z] 匹配大写字母 A 到 Z 中的任意一个字符
[0-9] 匹配数字 0 到 9 中的任意一个字符

\d 匹配一个数字字符，等价于 [0-9]
\D 匹配一个非数字字符，等价于 [^0-9]

\w 匹配一个单词字符（包括下划线），等价于 [a-zA-Z0-9_]
\W 匹配一个非单词字符，等价于 [^a-zA-Z0-9_]

\s 匹配一个空白字符（包括空格、\n 换行符、\t 制表符等）
\S 匹配一个非空白字符

//限定符
{n} 匹配前面的子表达式恰好 n 次，比如 \d{3} 代表匹配三位数字
{n,} 匹配前面的子表达式至少 n 次
{n,m} 匹配前面的子表达式至少 n 次，但最多不超过 m 次

* 匹配前面的子表达式零次或多次，等价于 {0,}，比如 \d* 表示零个或多个数字
+ 匹配前面的子表达式一次或多次，等价于 {1,} 
? 匹配前面的子表达式零次或一次，等价于 {0,1}
```

## Pattern 和 Matcher
```java
//方法1
boolean isNumber = Pattern.matches("[0-9]*", "abc");
//方法2
Pattern pattern = Pattern.compile("[0-9]*");
Matcher matcher = pattern.matcher("123");
boolean isNumber2 = matcher.matches();
//
System.out.println("test "+isNumber);
System.out.println("test "+isNumber2);
//--------- 打印 ---------
test false
test true  
```
方法1内部就是对方法2逻辑的封装

## Matcher#replaceAll 和 Matcher#replaceFirst
```java
Pattern pattern = Pattern.compile("[^0-9]"); //匹配除数字之外的任意字符
Matcher matcher = pattern.matcher("收费20元");
System.out.println(matcher.replaceAll("-")); //用横杠替换所有匹配的字符
//
Pattern pattern2 = Pattern.compile("[0-9]"); //匹配数字
Matcher matcher2 = pattern2.matcher("收费20元");
System.out.println(matcher2.replaceAll("*"));
//
Pattern pattern3 = Pattern.compile("[0-9]");
Matcher matcher3 = pattern3.matcher("收费20元5角");
System.out.println(matcher3.replaceFirst("*")); //用星号替换第一个匹配的字符
//--------- 打印 ---------
--20-
收费**元
收费*0元5角
```


## IP、Email 等正则表达式
```java
//androidx.core.util.PatternsCompat
androidx.core.util.PatternsCompat#IP_ADDRESS
androidx.core.util.PatternsCompat#EMAIL_ADDRESS
androidx.core.util.PatternsCompat#PHONE
```
 
