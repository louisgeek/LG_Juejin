# Java Behavioral 行为型模式之 Chain of Responsibility 责任链模式
- 责任链模式允许将请求沿着一条若干个处理者的链上进行传递，每个处理者都有机会处理请求，直到请求被处理或到达链的末端
- 简化了对象之间的连接，将责任合理分担到每个类（注意明确职责范围，避免职责重叠或遗漏），每个类只需要处理自己该处理的工作，降低了对象之间的耦合度，增强了系统的灵活性和扩展性
- 通常情况下如果一个处理者不能处理请求，它会将请求转发给下一个处理者，如果处理了请求了，那么这个流程就结束了，不过有些责任链模式，每个处理者都有可能参与处理请求，通常这种责任链被称为 Interceptor 拦截器或者 Filter 过滤器，它可以让每个处理者都做一部分工作
- 过长的责任链会影响性能，应尽量保持链的长度适中，同时也要确保责任链中没有循环引用，否则会导致无限循环
- 设计处理者时，可以尽可能地考虑它们的可插拔性，使得系统易于扩展和维护

处理者抽象类
```java
public abstract class Processor {
    //下一个处理者
    protected Processor nextProcessor;
    //设置下一个处理者
    public void setNextProcessor(Processor nextProcessor) {
        this.nextProcessor = nextProcessor;
    }
    //自身处理逻辑
    public abstract void processRequest(Request request);
}
```

若干个实际处理者
```java
public class AProcessor extends Processor {
    @Override
    public void processRequest(Request request) {
        if (1 <= request.code && request.code < 3) {
            System.out.println("组长审批通过，请假 3 天以内，不含 3 天，实际 " + request.code + " 天");
        } else {
            //给下一个处理者设置处理请求，没有则流程结束
            if (nextProcessor != null) {
                nextProcessor.processRequest(request);
            } else {
                System.out.println("组长后面没有下一个处理者，无法处理 " + request.code + " 天");
            }
        }
    }
}
//
public class BProcessor extends Processor {
    @Override
    public void processRequest(Request request) {
        if (3 <= request.code && request.code < 5) {
            System.out.println("经理审批通过，请假 5 天以内，不含 5 天，实际 " + request.code + " 天");
        } else {
            //给下一个处理者设置处理请求，没有则流程结束
            if (nextProcessor != null) {
                nextProcessor.processRequest(request);
            } else {
                System.out.println("经理后面没有下一个处理者，无法处理 " + request.code + " 天");
            }
        }
    }
}
//
public class CProcessor extends Processor {
    @Override
    public void processRequest(Request request) {
        if (5 <= request.code && request.code < 7) {
            System.out.println("总监审批通过，请假 7 天以内，不含 7 天，实际 " + request.code + " 天");
        } else {
            //给下一个处理者设置处理请求，没有则流程结束
            if (nextProcessor != null) {
                nextProcessor.processRequest(request);
            } else {
                System.out.println("总监后面没有下一个处理者，无法处理 " + request.code + " 天");
            }
        }
    }
}
```

使用
```java
public static void main(String[] args) {
    //创建处理者
    Processor aProcessor = new AProcessor();//组长
    Processor bProcessor = new BProcessor();//经理
    Processor cProcessor = new CProcessor();//总监
    //设置责任链
    aProcessor.setNextProcessor(bProcessor);
    bProcessor.setNextProcessor(cProcessor);
    //发送请求
    Request request1 = new Request(1,"请假 1 天");
    Request request2 = new Request(2,"请假 2 天");
    Request request3 = new Request(3,"请假 3 天");
    Request request4 = new Request(4,"请假 4 天");
    Request request5 = new Request(5,"请假 5 天");
    Request request6 = new Request(6,"请假 6 天");
    Request request7 = new Request(7,"请假 7 天");
    aProcessor.processRequest(request1);
    aProcessor.processRequest(request2);
    aProcessor.processRequest(request3);
    aProcessor.processRequest(request4);
    aProcessor.processRequest(request5);
    aProcessor.processRequest(request6);
    aProcessor.processRequest(request7);
    //-------- 结果 --------
    //组长审批通过，请假 3 天以内，不含 3 天，实际 1 天
    //组长审批通过，请假 3 天以内，不含 3 天，实际 2 天
    //经理审批通过，请假 5 天以内，不含 5 天，实际 3 天
    //经理审批通过，请假 5 天以内，不含 5 天，实际 4 天
    //总监审批通过，请假 7 天以内，不含 7 天，实际 5 天
    //总监审批通过，请假 7 天以内，不含 7 天，实际 6 天
    //总监后面没有下一个处理者，无法处理 7 天
}
```

## 应用场景
- 1 日志记录：不同级别的日志可以由不同的处理者来处理，比如错误日志可以由专门的错误处理者处理，而信息日志可以由普通的日志处理者处理
- 2 审批流程：比如 OA 申请流程需要经过多个层级的审批，每个层级的审批人只负责处理自己权限范围内的逻辑，不同级别的审批人组成了一个责任链，根据请求的级别进行审批处理
- 3 事件处理；按钮的点击事件，事件可以沿着组件的层次结构传递，直到有一个组件处理它为止，比如 Android 中的事件分发
- 4 异常处理：多层次异常处理，在应用程序中，不同类型的异常可以由不同的处理者处理，形成一个异常处理链

## 特点
- 1 降低耦合度：发送者和接收者解耦，处理者之间也松散耦合（只需要知道下一个处理者的存在，而不需要知道链中其他处理者的具体细节），将请求的处理逻辑分散到各个处理者中，每个处理者只负责自己的逻辑，职责单一，使得代码结构更加清晰，易于理解和维护
- 2 链式结构：请求从链的第一个对象开始，依次传递到下一个对象，直到请求被处理后结束或一直到达链的末端（可以考虑有默认处理逻辑兜底）
- 3 易于扩展和维护：由于松散耦合的特点，可以很容易地添加、删除或修改处理者去调整职责，对整个系统的结构影响不大，大大增强了系统的灵活性
- 4 常见应用：Android 的事件分发机制、Android 网络请求库 Retrofit 里的拦截器