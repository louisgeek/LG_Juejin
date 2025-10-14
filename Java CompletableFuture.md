# Java CompletableFuture
- CompletableFuture 是 Java 8 引入的一种现代化的、函数式的方式来简化异步编程的复杂性，实现了 Future 和 CompletionStage 接口，相比传统的 Future 和回调机制的功能更加灵活和强大，支持链式调用操作（替代嵌套回调，解决回调地狱）、组合协调多个异步任务、异常处理等能力，使代码更加清晰、可维护性更高
- 异步执行：支持将任务提交到线程池进行异步执行（避免阻塞主线程），支持多个任务的编排组合和结果处理，简化了非阻塞异步任务的处理，核心优势是可以在任务完成后自动触发后续的操作处理，避免传统 Future 的阻塞等待任务完成，大幅简化复杂异步场景的代码逻辑
- 链式调用: 支持链式处理结果
- 并行处理: 支持同时处理多个独立任务以提高性能
- 组合操作: 能够组合多个独立或依赖的异步任务（比如 thenCombine, allOf）
- 异常处理: 提供 handle, exceptionally 等方法处理异常
- ​​线程安全​​：CompletableFuture 本身线程安全，但回调中的共享变量需要同步
- 避免在回调中阻塞：thenApply, thenAccept 等同步回调方法会在上一个任务所在的线程中执行

## 创建 CompletableFuture
```java
//立即完成
CompletableFuture<String> future = CompletableFuture.completedFuture("Hello");

//手动触发完成
CompletableFuture<String> future = new CompletableFuture<>();
future.complete("Result"); //正常完成
future.completeExceptionally(new RuntimeException("错误")); //异常完成

//异步执行有返回值的任务（默认使用 ForkJoinPool.commonPool() 线程池，建议指定线程池）
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Async Result";
});

//异步执行无返回值的任务
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    //doSomething
});
```

## 链式处理结果
```java
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> "Hello")

//处理结果，有参无返回值 Consumer：void accept(T t)
future.thenAccept(result -> System.out.println(result));
future.thenAcceptAsync 异步处理结果

//转换结果 有参有返回值 Function：R apply(T t)
future.thenApply(result -> result.toUpperCase());
future.thenApplyAsync 异步转换结果

//执行操作，无参无返回值 Runnable：void run()
future.thenRun(() -> System.out.println("Task completed"));
future.thenRunAsync 异步执行操作

//链式处理
future.thenApply(s -> s + " World")
    .thenApply(String::toUpperCase)
    .thenApplyAsync(this::validate) //thenApplyAsync 虽然是异步的，但后续的 thenAccept 仍会等待它完成
    .thenAccept(validatedResult -> {
        System.out.println(validatedResult);
        this.saveToDB(validatedResult);
    })
    .thenRun(() -> System.out.println("Done"));
```

## 组合多个任务
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

//顺序执行
future1.thenCompose(result -> CompletableFuture.supplyAsync(() -> result + " processed"));

//合并两个结果，并返回新值，用于数据转换
CompletableFuture<String> combined = future1.thenCombine(future2, 
    (result1, result2) -> result1 + " " + result2);

//并行执行等待所有完成
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2);

//等待任一完成
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);

//等待两个都完成后执行， 消费两个结果但不返回值
future1.thenAcceptBoth(future2, (result1, result2) -> 
    System.out.println(result1 + " " + result2));
```

## 处理异常
```java
//处理异常
future.exceptionally(throwable -> {
    System.err.println("Error occurred: " + throwable.getMessage());
    return "defaultValue"; //返回默认值
});

//统一处理结果和异常
future.handle((result, throwable) -> {
    if (throwable != null) {
        return "error handled";
    }
    return result; //可按需返回新的结果
});

//处理结果和异常
future.whenComplete((result, throwable) -> {
        if (throwable != null) {
            System.out.println("异常: " + throwable.getMessage());
        } else {
            System.out.println("结果: " + result);
        }
    });
```

## 自定义线程池
```java
Executor executor = Executors.newFixedThreadPool(4);

CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "任务", executor)
    .thenApplyAsync(s -> s + "处理", executor);
```

## 其他
```java
//任一完成后执行
future1.applyToEither(future2, result -> "最快的结果: " + result);

//任一完成后消费
future1.acceptEither(future2, result -> System.out.println(result));

//检查完成状态
boolean isDone = future.isDone();
boolean isCancelled = future.isCancelled();
boolean isCompletedExceptionally = future.isCompletedExceptionally();

//取消任务
future.cancel(true); //等效于 future.completeExceptionally(new CancellationException())

//超时处理
CompletableFuture<String> timeoutFuture = CompletableFuture
    .supplyAsync(() -> slowOperation())
    .orTimeout(3, TimeUnit.SECONDS) //抛出 TimeoutException
    .exceptionally(ex -> "超时默认值");

//3秒后如果未完成，用默认值完成
future.completeOnTimeout("default", 3, TimeUnit.SECONDS); //不会抛出异常

//延迟执行​
future.completeAsync(() -> {}, CompletableFuture.delayedExecutor(5, TimeUnit.SECONDS));
```

## 总结
- supplyAsync：异步执行有返回值的任务，runAsync：异步执行无返回值的任务
- thenAccept: 消费结果（无返回值）、thenApply: 转换结果（有返回值）和 thenRun: 执行无依赖任务
- thenCompose 依赖、顺序执行，thenCombine 并行执行，可以返回组合后数据和 thenAcceptBoth 等待两个任务完成后消费结果
- exceptionally 捕获异常并返回默认值、handle 统一处理结果和异常和 whenComplete 处理结果和异常
