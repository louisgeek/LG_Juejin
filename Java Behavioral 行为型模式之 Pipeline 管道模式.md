# Java Behavioral 行为型模式之 Pipeline 管道模式
- 管道模式也称为流水线模式，是责任链模式的常用变种之一，通常情况下一个步骤会依赖上一个步骤输出的结果，如果下一步骤不依赖上一步骤的结果的话，那么可以考虑改用责任链模式     
- 管道模式可以看做是一个流水线作业，固定顺序步骤依次线性执行，每个步骤环节（阶段）都依赖上个环节的输出
- 每个步骤不管处不处理都会传递到下一个步骤，数据或任务在一个阶段被加工处理完成后，随即继续进入下一个阶段去处理
- 管道模式将多个处理步骤有序的连接在一起，使得数据处理流程更加清晰和模块化，每个步骤都可以进行独立测试和维护，从而提高了代码的可读性、可拓展性和可维护性

## 常规写法

Pipe 接口
```java
public interface Pipe<I, O> {
    O process(I input);
}
```

若干个 Pipe 接口的实现类
```java
public class TrimProcessor implements Pipe<String, String> {
    @Override
    public String process(String input) {
        return input.trim();
    }
}

public class UpperCaseProcessor implements Pipe<String, String> {
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
    //-------- 结果 --------
    //Input: ' Hello World  '
    //Output: 'HELLO WORLD'
}
```

## 采用函数式接口
- Pipe 接口用 java.util.function.Function （是一个函数式接口）替代

若干个 Pipe 接口的实现类
```java
public class TrimProcessor implements Function<String, String> {
    @Override
    public String apply(String input) {
        return input.trim();
    }
}
public class UpperCaseProcessor implements Function<String, String> {
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
- 实现类也可以用 Lambda 表达式简化

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

## 特点
- 1 模块化解耦合：复杂业务拆分成每个处理步骤环节（阶段）都是一个独立的模块，专注于自身数据处理，每个阶段职责明确，代码结构更清晰，也可以单独地进行开发和测试，降低了耦合度
- 2 顺序执行：处理步骤按照定义的顺序依次执行，像流水线一样，数据在每个步骤被加工处理后，传递到下一个步骤进行处理，直到全部步骤处理完毕
- 3 可扩展性：可以轻松地添加新的处理步骤或修改现有的步骤和顺序，而不影响整个系统的结构，大大增强了灵活性
- 4 可重用性：处理步骤环节（阶段）可以在不同的管道中重复使用，各个阶段能灵活搭配使用


## 责任链模式和管道模式
- 处理对象不同：责任链模式中，每个处理者都有机会处理请求，判断自己是否能处理，如果不能处理则传递给下一个处理者，通常情况下只有一个处理者会处理请求并产生最终结果（不严格要求，比如 Interceptor 应用）；而管道模式中，每个阶段通常只负责特定的处理任务，并在处理完后传递给下一个处理者，会按照顺序依次处理
- 处理结果不同：责任链模式中，请求可能会在中途被某个处理者处理产生最终结果后终结，不一定走完整个链条；而在管道模式中，数据通常会流经所有设定的阶段，除非遇到错误或其他终止条件，所以最终结果是所有处理者都处理后的结果
- 应用场景区别：责任链模式主要用于解耦请求的发送者和接收者，适用于需要根据条件判断选择处理对象的场景；而管道模式更侧重于构建灵活的数据处理流，适用于任务可以拆分为多个子步骤且需要依次处理的场景
- 责任链模式和管道模式两者可以根据具体需求选择合适的模式，或者将两种模式结合使用，以实现更加灵活和高效的处理流程，以满足更复杂的应用逻辑



