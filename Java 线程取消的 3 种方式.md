# Java 线程取消的 3 种方式
- Thread 取消线程通常有以下几种方式：
    - 1 使用 stop 方法来强制停止线程，是非常不安全的方式
    - 2 使用 volatile 设置标记位停止线程，是一种不完全可靠的方式
    - 3 使用线程中断机制以协作式的方式来停止线程
- Thread#stop 停止线程，强制停止一个正在运行的线程，该方法已废弃
- Thread#interrupt 中断线程，请求线程中断，将线程的中断状态设置为 true 
- Thread#isInterrupted 检查线程的中断状态，返回当前线程的中断状态，不会改变线程的中断状态
- Thread#interrupted 检查并清除线程的中断状态，返回当前线程的中断状态，并将线程的中断状态重置为 false，不过需要注意的是这是一个静态方法


## 强制停止线程
- 不推荐使用 Thread#stop 来粗暴的停止线程，可能会引起以下问题：
    - 1 Thread#stop 会立刻停止 run 方法中剩余的工作，包括在 catch 或 finally 等中的逻辑，因此可能会导致一些清理性的工作的得不到完成，比如文件，数据库等的关闭操作，可能会导致资源泄露的问题
    - 2 Thread#stop 会立即释放该线程所持有的锁，可能会导致数据得不到同步，出现数据不一致，也可能导致正在更新的数据结构被损坏等问题

## 设置 volatile 标记位来停止线程
- 使用 volatile 关键字来设置标记位来停止线程在某些情况下是不完全可靠的：
    - 1 单纯使用 volatile 关键字虽然可以保证变量的可见性（即一个线程对变量的修改能够及时让其他线程看到），但不能保证复合操作的原子性，可能会导致线程在不正确的时间点结束，导致出现不可预期的行为
    - 2 如果线程当前正在阻塞或等待情况下，那么它将不会立刻响应标记位的变化，而是必须等到它从阻塞或等待状态恢复过来才有机会去检查标记位，然后才能开始处理结束流程，时间点可能就出现一定的滞后了

## 协作式中断线程
- 线程中断机制是一种协作式机制，允许一个线程请求另一个线程应该停止其正在执行的操作，调用线程 Thread#interrupt 中断操作并不会直接中断线程（不会立即停止线程），而只是改变了目标线程的中断状态这个标志位，被请求中断的线程可以在适当位置（如循环条件、方法调用前后等）通过检查自身的中断状态来判断是否被中断了，从而采取相应的行动去响应这个中断请求，不过甚至也可以选择忽略它（因为响应中断并不是强制性的要求，这取决于具体的实现和需求） 
- 通常情况下线程检查自身发现中断状态为 true 时，表示该线程已经被中断，此时要尽快处理中断请求，可以选择去执行一些清理资源、保存数据等操作，然后再结束线程
- 这种设计使得线程中断变得相对安全，因为线程仍有机会在合适的时地方用合理的方式结束自身的工作，而不是像 Thread#stop 那么被强制停止


线程任务执行完后正常结束
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        @Override
        public void run() {
            //模拟线程执行任务
            for (int i = 0; i < 100000; i++) {
               System.out.println("线程运行中 i=" + i);
            }
            //在执行完所有任务后线程正常结束
            System.out.println("---- 线程结束 ----");
        }
    };
    thread.start();
}
```

加入请求线程中断的逻辑
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        @Override
        public void run() {
            //模拟线程执行任务
            for (int i = 0; i < 100000; i++) {
               System.out.println("线程运行中 i=" + i);
            }
            //虽然调用了 Thread#interrupt 方法请求线程中断但还是会执行完所有的任务
            System.out.println("---- 线程结束 ----");
        }
    };
    thread.start();
    //演示逻辑
    try {
        //等待一段时间后去中断 thread 线程
        Thread.sleep(10);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("请求线程中断");
    //对 thread 线程请求线程中断
    thread.interrupt();
}
```


说明仅仅只改变中断状态是达不到取消线程的效果的，需要继续加入响应中断的逻辑
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        @Override
        public void run() {
            //模拟线程执行任务
            for (int i = 0; i < 100000; i++) {
                //目的就是让线程在适当的时候检查自身的中断状态，并完成响应中断请求的逻辑
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程检测到自身的中断状态为 true ，于是准备停止");
                    //释放资源并结束线程
                    break; //这里用 return 也行
                }
                System.out.println("线程运行中 i=" + i);
            }
            //
            System.out.println("---- 线程结束 ----");
        }
    };
    thread.start();
    //演示逻辑
    try {
        //等待一段时间后去中断 thread 线程
        Thread.sleep(10);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("请求线程中断");
    //对 thread 线程请求线程中断
    thread.interrupt();
}
```

存在阻塞状态（比如 Object#wait、Thread#sleep、Thread#join 或 Condition#await 等方法调用产生的）下的响应中断
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        @Override
        public void run() {
            //模拟线程执行任务
            for (int i = 0; i < 100000; i++) {
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    //当一个线程在阻塞状态时如果调用了该线程的 interrupt 方法的话，那么阻塞方法就会抛出 InterruptedException 异常，类似于线程检测到自身的中断状态为 true，也就意味着这里需要加入响应中断的逻辑了
                    //这里抛出 InterruptedException 异常的设计是为了线程可以从阻塞状态恢复（唤醒）过来（表示阻塞操作由于中断而提前结束），能在线程结束前有机会去处理中断请求
                    //另外抛出 InterruptedException 异常的同时会清除线程的中断标志位（中断状态被重置为 false）
                    //所以这里可以做一些停止线程任务继续执行的逻辑（比如直接退出循环）或者也可以在这里再次调用 Thread#interrupt 重设中断状态（标记回中断状态为 true）然后和适当位置的 Thread#isInterrupted() 判断配合来完成响应中断请求的逻辑
                    break;
                }
                System.out.println("线程运行中 i=" + i);
            }
            //
            System.out.println("---- 线程结束 ----");
        }
    };
    thread.start();
    //演示逻辑
    try {
        //等待一段时间后去中断 thread 线程
        Thread.sleep(10);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("请求线程中断");
    //对 thread 线程请求线程中断
    thread.interrupt();
}
```

线程在阻塞状态下也能正确响应中断请求
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        @Override
        public void run() {
            //模拟线程执行任务
            for (int i = 0; i < 100000; i++) {
                //适当位置检查自身的中断状态，并完成响应中断请求的逻辑
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程检测到自身的中断状态为 true ，于是准备停止");
                    //释放资源并结束线程
                    break; //这里用 return 也行
                }
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    //抛出 InterruptedException 异常的同时会清除中断标志位（中断状态被重置为 false）
                    //所以这里要特别注意，可以按需重设中断标志位，就能做到即使线程在阻塞状态下也能够正确地响应中断请求了（不然很容易错过外部设置的那一次中断请求）
                    Thread.currentThread().interrupt();
                }
                System.out.println("线程运行中 i=" + i);
            }
            //
            System.out.println("---- 线程结束 ----");
        }
    };
    thread.start();
    //演示逻辑
    try {
        //等待一段时间后去中断 thread 线程
        Thread.sleep(10);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("请求线程中断");
    //对 thread 线程请求线程中断
    thread.interrupt();
}
```

## 总结
- 不推荐使用 stop 方法来强制停止线程，也不推荐使用 volatile 设置标记位停止线程
- 线程中断机制是一种交互式取消方式，一个线程请求另一个线程中断，而另一个线程就是对这个线程的请求中断做出响应（一个线程不应该由其他线程来直接强制中断或者停止，而是应该由线程自己自行进行停止，因为任务本身的代码肯定比发出取消请求的代码更清楚了解自身该如何执行清除释放结束等工作，说到底目标线程比调用者更加了解自身线程应不应该被停止，何时停止，如何停止）
- 线程自己检查自身中断状态或者捕获 InterruptedException 然后进行合适的处理以响应中断
- 通过合理地运用线程中断机制，可以确保线程能够安全、有序地停止，从而避免资源泄露以及一些潜在的问题



