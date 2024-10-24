# Java Behavioral 行为型模式之 Pipeline Pattern 管道模式
- 管道模式是责任链模式的常用变种之一，如果下一个步骤不依赖上一个步骤输出结果的话，那么可以改用责任链模式     
- 管道模式可以看做是一个流水线作业，固定顺序步骤依次线性执行，每个步骤环节（阶段）都依赖上个环节的输出
- 每个步骤不管处不处理都会传递到下一个步骤，数据或任务在一个阶段被处理（加工）完成后，随即继续进入下一个阶段去处理
- 管道模式将多个处理步骤有序的连接在一起，使得数据处理流程更加清晰和模块化，每个步骤都可以进行独立测试和维护，从而提高了代码的可读性、可拓展性和可维护性

## 常规写法
Pipe 接口

```java
interface Pipe<I, O> {
    O process(I input);
}
```

若干个 Pipe 接口的现实类
```java
class TrimProcessor implements Pipe<String, String> {
    @Override
    public String process(String input) {
        return input.trim();
    }
}

class UpperCaseProcessor implements Pipe<String, String> {
    @Override
    public String process(String input) {
        return input.toUpperCase();
    }
}
```
使用
```java
 public static void main(String[] args) {
    String input = " Hello World  ";
    Pipe<String, String> firstPipe = new TrimProcessor();
    Pipe<String, String> secondPipe = new UpperCaseProcessor();
    String output = secondPipe.process(firstPipe.process(input));
    System.out.println("Input: '" + input + "'");
    System.out.println("Output: '" + output + "'");
    }
    //结果
    //Input: ' Hello World  '
    //Output: 'HELLO WORLD'
```

## 采用函数式接口
Pipe 接口用 java.util.function.Function （是一个函数式接口）替代

若干个 Pipe 接口的现实类
```java
class TrimProcessor implements Function<String, String> {
    @Override
    public String apply(String input) {
        return input.trim();
    }
}
class UpperCaseProcessor implements Function<String, String> {
    @Override
    public String apply(String input) {
        return input.toUpperCase();
    }
}
```

```java
 String input = " Hello World  ";
 Function<String, String> firstPipe = new TrimProcessor();
 Function<String, String> secondPipe = new UpperCaseProcessor();
 String output = secondPipe.apply(firstPipe.apply(input));
 System.out.println("Input: '" + input + "'");
 System.out.println("Output: '" + output + "'");
```

## 引入 Lambda 表达式
现实类也可以用 Lambda 表达式简化

```java
 String input = " Hello World  ";
 Function<String, String> firstPipe = it -> it.trim();
 Function<String, String> secondPipe = it -> it.toUpperCase();
 String output = secondPipe.apply(firstPipe.apply(input));
 System.out.println("Input: '" + input + "'");
 System.out.println("Output: '" + output + "'");
```

## 方法引用继续简化
```java
 String input = " Hello World  ";
 Function<String, String> firstPipe = String::trim;//方法引用
 Function<String, String> secondPipe = String::toUpperCase;//方法引用
 String output = secondPipe.apply(firstPipe.apply(input));
 System.out.println("Input: '" + input + "'");
 System.out.println("Output: '" + output + "'"); 
```


## 可以用 andThen 和 compose 连写

```java
 String input = " Hello World  ";
 Function<String, String> firstPipe = String::trim;//方法引用
 Function<String, String> secondPipe = String::toUpperCase;//方法引用
 Function<String, Integer> thirdPipe = String::length;//方法引用
 //...
 Function<String, Integer> pipeline = firstPipe.andThen(secondPipe).andThen(thirdPipe);
//        Function<String, String> pipeline = firstPipe.compose(secondPipe);
 Integer outputLen = pipeline.apply(input);
 System.out.println("InputLen: '" + input.length() + "'");
 System.out.println("OutputLen: '" + outputLen + "'");
```



## 应用场景
- 1 日志处理和记录（过滤、解析、存储等）
- 2 审批处理和验证（验证、审批等）
- 3 数据清理和处理（清洗、转换、分析等）
- 4 图像视频处理（缩放、旋转、滤镜、压缩等）

```java
        //1准备数据
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        //2定义数据处理步骤
        
        // 偶数
        Function<List<Integer>, List<Integer>> filterEven = list -> list.stream()
                .filter(n -> n % 2 == 0)
                .collect(Collectors.toList());
        // 除法
        Function<List<Integer>, List<Integer>> divisionByTwo = list -> list.stream()
                .map(n -> n / 2)
                .collect(Collectors.toList());
        // 求和
        Function<List<Integer>, Integer> sum = list -> list.stream()
                .mapToInt(Integer::intValue)
                .sum();

        //3构建管道
        Function<List<Integer>, Integer> pipeline = filterEven
                .andThen(filterEven)
                .andThen(divisionByTwo)
                .andThen(sum);

        //4执行管道
        int result = pipeline.apply(numbers);
        System.out.println("Result: " + result); //  Result: 15
```

## 管道模式特点
- 1 模块化：复杂业务拆分成每个处理步骤都是一个独立的模块，逻辑清晰了，也可以单独开发和测试
- 2 顺序执行：处理步骤按照定义的顺序依次执行，像流水线一样
- 3 可扩展性：可以轻松地添加新的处理步骤或修改现有的步骤，而不影响整个系统的结构
- 4 可重用性：处理步骤可以在不同的管道中重复使用