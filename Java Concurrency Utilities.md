# 前言

Java concurrency utilities package 提供了强大的、可扩展的框架，主要用于高性能的多线程编程工具，比如线程池和阻塞队列。

这些工具为高级并发编程提供 low-level primitives；

# 概览

Java platform 提供了 concurrency utilities 包。这些类被设计为在构建并发类或应用程序时用作构建块。

就像集合框架通过提供常用数据结构的实现简化了内存数据的组织和操作一样，并发工具通过提供并发设计中通用的理念的实现来简化并发编程。

- concurrency utilities 包括高性能、灵活的线程池；
- 用于异步执行任务的框架；
- 提供并发访问保证的集合类；
- 同步工具，比如信号量、原子变量；
- 锁；
- 条件变量；

和自己开发组件相比，使用  concurrency utilities 有以下好处：

（1）减少编程工作：直接用现成的组件去开发，不用自己重复写轮子；

（2）提供性能：这些并发工具都是经过专家评审检验的，这些实现可能比典型的实现更快、更可伸缩，即使是由熟练的开发人员实现；

（3）高度可靠：开发并发工具类是非常困难的，因为 Java 平台提供的并发原语（如 synchronized、volatile、wait()、notify()、notifyALL()）想要正确的使用是有一定难度的，并且一旦遇到错误较难排查。但是通过使用标准的、可扩展的并发工具，就可以解决一些潜在的问题，比如死锁、线程饥饿、线程竞争、过多的线程上下文切换；

（4）提高可维护性：使用标准类库的程序很好理解和维护；

（5）提高生产效率：一般来说，开发人员对标准类库是很熟悉的，因此不需要学习 API 和临时并发组件的行为。此外，当并发应用程序构建在可靠的、经过良好测试的组件上时，它们更容易调试。

这些 concurrency utilities 包括以下几个部分：

- 任务调度框架（Task scheduling framework）：`Executor` 接口根据一组执行策略标准化异步任务的调用、调度、执行和控制；
- Fork/join framework：基于 `ForkJoinPool` 类，该框架也是 Executor 接口的实现。适用于用线程池执行大量任务的场景。`work-stealing` 技术可以让所有的线程一直处于忙碌状态，充分发挥多处理器的优势；
- Concurrent collections：对集合框架的扩展，提供包括阻塞队列、阻塞双端队列，Map、List 和 Queue 的同步实现；
- Atomic variables（原子变量）：这些工具类提供原子的操作一个变量（原始数据类型或者引用）。提供原子计算以及 CAS（compare-and-set）操作。原子变量定义在 `java.uitl.concurrent.atomic` 包下面，在大多数平台上提供比 synchronization 更高的性能，使它们对于实现高性能并发算法和方便地实现计数器和序列号生成器非常有用；
- Synchronizers（同步器）：通用的同步类，比如 `semaphores`、`barriers`、`latches`、`phasers` and `exchangers`，用这些工具可以很方便的协调多个线程；
- Locks：Java 的 synchronized 关键字提供的内置的 locking 机制，但是内置的 monitor lock 有很多限制。所以 `java.util.concurrent.lock` 包提供了和 synchronization 具有相同的内存语义但是拥有更高性能的锁，并且在获取锁时提供超时时间，每个锁有多个 condition variables，非嵌套的持有多个锁，并且支持中断等待获取锁的线程；
- Nanosecond-granularity timing：`System.nanoTime` 方法允许访问纳秒粒度的时间源，用于进行相对时间测量和接受超时的方法（比如：`BlockingQueue.offer`、`BlockingQueue.poll`，`Lock.tryLock`，`Condition.await` 以及 `Thread.sleep`）。`System.nanoTime` 方法依赖于系统平台。

# Tutorials

## Concurrency

计算机用户想当然的认为他们的系统可以在同一时间做多个事情。他们假设当其他应用程序下载文件、管理打印队列和流音频时，他们可以继续在文字处理器中工作。即使是单个应用程序也经常被期望同时做多件事。例如，流式音频应用程序必须同时从网络上读取数字音频、解压缩、管理回放并更新其显示。即使是文字处理器也应该随时准备对键盘和鼠标事件作出响应，无论它有多忙，重新格式化文本或更新显示。能够做这些事情的软件被称为并发软件（concurrent software）。

Java 平台从一开始就被设计为支持并发编程，Java 编程语言和Java 类库提供了基本的并发支持。从 5.0 版开始，Java 平台还包含了高级并发 api。本节课介绍了平台的基本并发支持，并总结了 `java.util.concurrent` 包中的一些高级 api。

## 进程和线程

在并发编程中，有两个基本执行单位：进程和线程。在 Java 编程语言中，并发编程主要与线程有关。然而，进程也很重要。

计算机系统通常有许多活动进程和线程。即使在只有一个执行核心的系统中也是如此，因此在任何给定时刻只有一个线程实际执行。单个核心的处理时间通过称为时间切片（`time slicing`）的操作系统特性在进程和线程之间共享。

计算机系统拥有多个处理器或具有多个执行核的处理器正变得越来越普遍。这极大地增强了系统并发执行进程和线程的能力 —— 但是，即使在没有多个处理器或执行核心的简单系统上，并发也是可能的。

### 进程

进程具有自包含（`self-contained`）的执行环境。一个进程通常有一个完整的、私有的基本运行时资源集（`basic run-time resources`）；特别是，每个进程都有自己的内存空间（`memory space`）。

进程通常被视为程序或应用程序的同义词。然而，用户所看到的单个应用程序实际上可能是一组协作过程。为了让进程之间能够通信，大多数操作系统都支持进程间通信资源（`Inter Process Communication`，IPC），例如 pipes 和 sockets。IPC 不仅用于同一系统上的进程之间的通信，还用于不同系统上的进程之间的通信。

Java 虚拟机的大多数实现都是作为单个进程运行的。Java 应用程序可以通过 `java.lang.ProcessBuilder` 创建额外的进程。

### 线程

线程也被称作轻量级的进程（`lightweight processes`），进程和线程都提供了一个执行环境，但是创建一个新线程比创建一个新进程需要更少的资源。

一个进程中可以包含多个线程 —— 每个进程至少包含一个线程。线程共享进程的资源，包括内存和 open files。这有助于线程之间更高效的沟通，但也有潜在的问题。

多线程执行是Java平台的一个基本特性。每个应用程序至少有一个线程 —— 如果算上做内存管理和信号处理等工作的 “系统” 线程，就会有几个。但是从应用程序开发者的角度来看，开始时只有一个线程，称为主线程（`main thread`）。该线程具有创建其他线程的能力。



## 线程对象

每个线程都和 `Thread` 类的一个实例关联，这里有两种方式使用 Thread 对象开发并发应用：

- 直接控制线程的创建和管理：在需要开启线程执行任务时，直接创建 Tread 的实例；
- 对线程管理进行抽象，将异步任务传递给 Executor 调度执行；

### 定义和启动 Thread

应用在创建 Thread 的实例时需要将要运行的代码也一并传递过去，这里有两种方式实现：

（1）提供 `Runnable` 对象。Runnable 接口中定义了一个方法 `run` 包含了线程要执行的任务。通过 Thread 的构造器把 Runnable 传递进去，比如下面这样：

```java
public class HelloRunnable implements Runnable {

    public static void main(String[] args) {
        new Thread(new HelloRunnable()).start();
    }

    @Override
    public void run() {
        System.out.println("say hello.");
    }
}
```

（2）继承 Thread 类，因为 Thread 类本身就实现了 Runnable 接口：

```java
public class HelloThread extends Thread {

    @Override
    public void run() {
        System.out.println("hello thread#run()");
    }

    public static void main(String[] args) {
        new HelloThread().start();
    }
}
```

注意上面两个例子中都调用了 `start()` 方法，JVM 才会开启新的线程并执行 run 方法。

那么我们使用哪一种方式更好一些？

第一种方式使用 Runnable 对象通常使用的更多一些，因为其他类也可以实现 Runnable 接口，而 Thread 的继承仅局限于 Thread 类体系。

第二种方式在简单的应用程序中可以使用，但是受限于你的任务类必须是 Thread 的子类。

我们更倾向于第一种方式，因为可以将 Runnable 任务和执行它的 Thread 对象分离开。这种方法不仅更加灵活，而且适用于后面介绍的高级线程管理 api。

此外，Thread 类中定义了一些方法用于管理线程。这些方法包括静态方法，它们提供关于调用该方法的线程的信息或影响其状态。其他方法从涉及管理线程和 thread 对象的其他线程调用。

### 使用 Sleep 暂停线程执行

`Thread.sleep` 方法可以让当前线程暂停一段时间。这是一种有效的方法，可以将处理器时间提供给应用程序的其他线程或可能在计算机系统上运行的其他应用程序。

此外，Thread 还提供了 sleep 方法的两个重载方法，可以传递时间参数，但是并不能保证 sleep 的时间非常准确，因为这依赖于底层操作系统。

同时，睡眠时间可以通过中断终止。

无论如何，你都不能假定调用 sleep 会在指定的时间内挂起线程。

```java
public class TestSleepMethod {
    public static void main(String[] args) throws InterruptedException {
        String[] importantInfo = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
        };

        for (int i = 0; i < importantInfo.length; i++) {
            // Pause for 4 seconds
            Thread.sleep(4000);
            // Print a message
            System.out.println(importantInfo[i]);
        }
    }
}
```

注意，main 方法声明可以抛出 `InterruptedException` 异常，当其他线程终止了正在 sleep 的线程时，sleep 方法就会抛出该异常。

### Interrupts 中断

中断（`interrupt`）是对线程的一个指示，它应该停止正在做的事情，然后做点别的事。由开发者决定线程是如何响应 interrupt 指令的，但是注意线程终止是很常见的。

如果要让某个线程中断，只需要调用该线程关联的 Thread 对象的 `interrupt` 方法就可以了，为了使中断机制正确工作，被中断的线程必须支持自己的中断。

#### Support Interruption（支持中断）

怎么样让一个线程支持中断机制呢？这取决于它当前正在做什么。

如果一个线程经常调用某个方法，该方法可以抛出 InterruptedException 异常。那么只需要简单的捕获异常，然后结束 run 方法就可以了，比如下面的 `run` 方法代码块：

```java
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```

很多方法都可以抛出 InterruptedException 异常，比如 sleep 方法，被设计为取消当前操作并在接收到中断时立即返回。

那么如果一个线程在处理需要很长时间的任务，且没有调用可以抛出 InterruptedException 异常的方法，又该如何处理中断指令呢？此时可以使用 `Thread.interrupted` 方法来检查当前吸纳从是否是中断状态，如果返回 true 则表示当前线程收到了 interrupt 指令：

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```

在这个简单的示例中，代码只是测试中断，如果收到中断则退出线程。在更复杂的应用程序中，抛出 InterruptedException 可能更有意义：

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

这允许中断处理代码集中在 catch 子句中。

#### The Interrupt Status Flag

中断机制的实现原理是基于一个内部的 flag（被称为 `interrupt status`）。调用 `Thread.interrupt` 会更新这个 flag，而调用静态方法 `Thread.interrupted` 就会检查并清空中断状态。而非静态的方法 `isInterrupted` 则是其他线程检查某个线程是否处于中断状态时调用的，不会改变 status flag。

按照惯例，任何通过抛出 InterruptedException 退出的方法都会在退出时清除中断状态。然而，中断状态总是可能立即被另一个调用 `interrupt()` 的线程再次设置。

### Joins

join 方法允许一个线程等待另一个线程执行完毕。

```java
t.join();
```

如果 t 是一个 Thread 实例，且它关联的线程正在执行，就会导致当前线程暂停执行，直到 t 关联的线程执行完毕。

`join` 的重载方法允许提供一个 wait 时间，但是，与 `sleep` 一样，join 依赖于操作系统的计时，因此您不应该假设 join 等待的时间与您指定的时间一样长。

像 sleep 一样，join 通过中断退出 InterruptedException 来响应中断。

### SimpleThreads Example

下面的代码结合了当前章节的某些概念，SimpleThreads 中包含了两个线程，第一个是每个 Java 应用都有的一个主线程，main 线程利用 Runnable 对象创建了一个新的 Thread，然后等待它执行完毕。如果 MessageLoop 用了很长时间执行，主线程就会中断它。

MessageLoop 线程会打印一系列消息，如果在完成工作之前就收到了中断指令，MessageLoop 会提示退出信息：

（ps：可以调小 patience 时间来观察效果）

```java
public class SimpleThreads {
    static void threadMessage(String message) {
        System.out.printf("%s: %s%n", Thread.currentThread().getName(), message);
    }

    public static void main(String[] args) throws InterruptedException {
        // 等待 patience 秒后中断子线程, 这里默认值是 1 分钟
        long patience = 1000 * 1 * 1;

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();
        threadMessage("Waiting for MessageLoop thread to finish");

        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // 最多等一秒
            t.join(1000);
            if ((System.currentTimeMillis() - startTime) > patience && t.isAlive()) {
                threadMessage("Tired of waiting!");
                // 中断
                t.interrupt();
                // 上面的中断操作执行速度依赖底层 OS, 此时可以等一下
                t.join();
            }
        }
        threadMessage("Finally!");
    }

    public static class MessageLoop implements Runnable {
        @Override
        public void run() {
            String[] importantInfo = {
                    "One", "Two", "Three", "Four", "Five"
            };
            try {
                for (String info : importantInfo) {
                    // 等待 4s
                    Thread.sleep(4000);
                    threadMessage(info);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }
}
```

## Synchronization（同步）

线程的通信通常是依据某个所有线程都可以访问到的字段或者对象来实现的。这种通信方式非常高效，但是也会导致两种类型的错误：

（1）`thread interference`（线程干扰）；

（2）`memory consistency errors`（内存一致性错误）；

而 `synchronization`（同步）机制正是用于解决这两类错误的工具。

但是，同步机制也会导致 `thread contention`（线程竞争）：当多个线程同时去操作同一个资源将会导致一个或多个线程执行速度变慢，甚至暂停执行。

线程竞争有两种表现形式：`starvation`（饥饿）和 `livelock`（活锁）；

本小节介绍以下主题：

- `Thread Interference`：当多线程访问共享资源时会发生的错误；
- `Memory Consistency Errors`：描述共享内存视图不一致所导致的错误；（原文：`describes errors that result from inconsistent views of shared memory.`）
- `Synchronized Methods`：同步方法是一种简单有效的阻止线程干扰和内存一致性错误的手段；
- `Implicit Locks and Synchronization`：基于同步原理的隐式锁是一种更加常用的同步手段；
- `Atomic Access`：原子访问是一种可以不被其他线程干扰的 operation；

### Thread Interference 线程干扰

参考下面的代码：

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

Counter 类很简单，每次调用 increment 方法就会使 c 增加 1，每次调用 decrement 就会使 c 减少 1。

然而一旦 Counter 对象被多个线程引用，就会发生线程干扰使得最终的结果并不一定会像我们期望的那样。

当在不同线程中运行但作用于相同数据的两个操作交叉时，就会发生干扰。这意味着这两个操作由多个步骤组成，并且这些步骤的序列重叠。

Counter 实例上的操作似乎不可能交织，因为 c 上的两个操作都是单一的简单语句。然而，即使是简单的语句也可以被虚拟机转换为多个步骤。我们不会研究虚拟机所采取的具体步骤-知道 `c++` 的单个表达式可以分解为三个步骤就足够了:

1. 读取 c 当前的值；
2. 将读取到的值加一；
3. 将相加后的结果回写给 c；

表达式 `c--` 可以用同样的方法分解，只是第二步是递减而不是递增。

假设线程 A 在线程 B 调用递减函数的同时调用递增函数。如果 c 的初始值为 0，它们的交错动作可能遵循以下顺序:

1. Thread A：读取 c；
2. Thread B：读取 c；
3. Thread A：递增读取到的值，结果为 1；
4. Thread B：递减读取到的值，结果为 -1；
5. Thread A：回写给 c，此时 c = 1；
6. Thread B：回写给 c，此时 c = -1；

假设 JVM 按照上述步骤执行，则最终 c = -1，线程 A 的操作相当于被抛弃了，生效的是线程 B。这种特殊的交织只是一种可能。在不同的情况下，线程 B 的结果可能丢失，或者根本没有错误。由于线程干扰错误不可预测，因此很难发现和修复。

### Memory Consistency Errors 内存一致性错误

当多线程访问同一个共享内存资源时，如果每个线程看到的内存视图存在不一样的清空，此时就会发生内存一致性错误。内存一致性错误非常复杂，但是幸运的是程序员不需要详细了解这些原因。我们所需要的只是一种避免它们的策略。

避免内存一致性错误的关键是理解 `happens-before` 关系。这种关系只是保证一个特定语句写入的内存对另一个特定语句可见。要了解这一点，请参考以下示例。假设定义并初始化了一个简单的int字段：

```java
int counter = 0;
```

counter 字段被两个线程所共享，A 和 B 线程。假设线程 A 递增了 counter：

```java
counter++;
```

不久后，线程 B 打印了 counter：

```java
System.out.println(counter)
```

如果这两个操作在同一个线程内顺序执行（JVM 保证同一线程内执行的语句遵循顺序一致性原则，是同步的），那么就可以认定打印的结果一定是 `1`。但是如果这两条语句在两个不同的线程内分别执行，打印的结果就可能是 0 了。因为不能保证线程 A 对 counter 的改变对线程 B 可见 —— 除非程序员在这两个语句之间建立了 `happens-before` 关系。

Java 平台提供一些可以构建 `happens-before` 关系的 actions。一种方式是 `Synchronization` 机制，后面的章节会介绍。

到此为止，我们已经见过的有两种 actions 可以创建 `happens-before` 关系：

- 调用 `Thread.start` 语句时，每个与该语句（调用 start 方法的语句）具有 `happens-before` 关系的语句，也与新线程中执行的每个语句具有 `happens-before` 关系；如果在新的线程内又创建了新的线程，那么这个关系会继续延续下去（ps：比如主线程中创建的新线程，新线程中又继续创建新新线程，关系是这样的，main happens-before new1 happens-before new2）；
- 当某个线程调用了其他线程的 `Thread.join()` 方法来结束其他线程，此时结束线程内部的所有语句与某个线程 `Thread.join()` 语句后面的语句具有 `happens-before` 关系。如果结束线程内部继续创建新的线程且使用 `Thread.join()` 结束，则该关系会延续下去，和 start 是类似的。

可以参考下面的例子：

```java
public static void main(String[] args) throws InterruptedException {
    Counter counter = new Counter();

    Runnable r = () -> {
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            counter.increment();
        }
    };
    
    Thread t1 = new Thread(r);
    Thread t2 = new Thread(r);
    Thread t3 = new Thread(r);

    t1.start();
    t2.start();
    t3.start();
    
    Thread.sleep(1000);

    System.out.println(counter.value());
}
```

Counter 还是用的前面的代码片段，这里程序输出的结果不一定是我们期望的 c = 30，因为存在内存一致性问题。

从 happens-before 的角度看，这段代码存在下面的关系：

- 按照顺序一致性模型，`t1.start()` 语句之前的语句都 hb 它，它又 hb 线程 t1 内部的所有语句，也就是前面定义的 r；
- `t2.start()` 和 `t3.start()` 也是一样的道理；
- 但是存在这样的问题：Thread t1、Thread t2、Thread t3 彼此没有 happens-before 的关系，因此就可能出现问题。

下面把上面的代码改变一下：

```java
t1.start();
t1.join();

t2.start();
t2.join();

t3.start();
t3.join();
```

此时就不会出现内存一致性错误了，因为现在程序中 hb 的关系如下：

- 按照顺序一致性模型，`t1.start()` 之前的代码 hb 于它，它又 hb 于 Thread t1；
- `t1.join()` 使得 Thread t1 中的所有语句 hb 于 它之后的语句；
- 同理后面的 t2、t3 也是一样；
- 这样程序中存在如下 hb 关系：
  - `t1.start()` 之前的语句 hb -> t1；
  - t1 hb  ->`t1.join()`；
  - `t1.join()` hb -> `t2.start()`；
  - t2 hb -> `t2.join()`；
  - `t2.join()` hb -> `t3.start()`；
  - t3 hb -> `t3.join()` 之后的代码；
- 根据 happens-before 的传递性可得知，对于线程 t1、t2、t3，t1 中所有语句 hb t2 中所有语句 hb t3 中所有语句；

多线程执行在 JVM 层面将语句的执行顺序转变为和顺序执行一致，这样就避免了顺序一致性错误；

更多 happens-before actions 参考 [java.util.concurrent 包的描述部分](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility)；

也可以参考 Java 语言声明中关于 [happens-before order 的部分](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.5)；

简单提一下：

### Memory Consistency Properties  happens-before 关系

[JLS](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5) 中定义了对共享资源 reads/writes 的内存操作的 `happens-before` 关系。利用 happens-before 关系可以保证一个线程对共享资源的写操作的结果对另一个线程对该资源的读操作是可见的（具体内存层面就是这个写操作发生在读操作前面）；Java 的 `synchronized` 和 `volatile` 关键字、线程的 `Thread.start()`、`Thread.join()` 方法都遵循 `happens-before` 关系，还有下面这些：

（1）在同一个线程中，代码语句的执行顺序和它们在程序中声明的顺序是一致的；

（2）对 monitor 的解锁操作（synchronized 块或方法的出口）happens-before 后续对同一个 monitor 的加锁操作（synchronized 块或方法的入口），且由于 happens-before 的传递性，线程内发生在解锁之前的所有 actions 都 happens-before 后续加锁之后的所有 actions；（比如锁定了某个代码块）

（3）对 `volatile` 修饰的字段的写操作都 happens-before 后续对该字段的读操作；volatile 字段的写入和读取与具有进入和退出的 monitor（锁定代码块）类似的内存一致性效果，但不需要互斥锁定（后者是通过对 monitor 加互斥锁实现的）；

（4）一个线程中某个地方调用了 `Thread.start()` 方法，则该语句 happens-before 创建的新线程中的所有 actions；

（5）一个线程中的某个地方调用了 `Thread.join()` 方法，则 join 的线程中的所有 actions 都 happens-before 调用 join 方法的线程中 join() 返回之后的所有 actions；

`java.util.concurrent` 包以及扩展了它的子包中的所有类中的方法都可以保证 higher-level synchronization。包含下面这些：

（1）多个线程访问某个非同步保证的集合对象，如果其中一个线程将该对象放入了并发安全的集合，那么该线程中对集合做同步封装之前的 actions happens-before 其他线程对非同步保证集合的操作访问和移除元素操作的后续 actions；

（2）在将 Runnable 提交给 Executor 之前的 actions happens-before 该 Runnable 开始执行的 action；同理，将 Callable 提交给 ExecutorService 也是一样的；

（3）由 Futrue 表示的异步计算的 action happens-before 其他线程调用 `Future.get()` 方法后面的 actions；

（4）在调用释放同步器方法（比如：`Lock.unlock`、`Semaphore.release`、`CountDownLatch.countDown`）之前的 actions happens-before 其他线程中调用了对同一个同步器调用获取方法（比如：`Lock.lock`、`Semaphore.acquire`、`Condition.await`、`CountDownLatch.await`）的后续 actions；（ps：和前面提到的 monitor 机制类似）

（5）对每一对通过 Exchanger 的 `exchange()` 方法成功交换对象的线程，在每个线程中 `exchange()` 操作之前执行的 actions happens-before 在另一个线程中对应的 `exchange()` 的后续 actions；

（6）调用 `CyclicBarrier.await` 和 `Phaser.awaitAdvance`（以及它的重载方法）之前的 actions happens-before barrier action（参考循环栅栏的概念），同时 barrier action 中的操作 happens-before 其他线程中 `await` 操作成功执行后的后续 actions；

### Synchronized Methods 同步方法

Java 提供了两种基础的同步机制：同步方法和同步语句。

先看第一种同步方法：

想要使一个方法同步，只需要在方法签名上加上 `synchronized` 关键字修饰即可：

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

还是之前计数器的例子，只不过在两个方法上加了 `synchronized` 关键字，这样做有两个好处：

- 首先，对同一对象的同步方法的两次调用是不可能交织的。当一个线程正在为一个对象执行同步方法时，所有为同一对象调用同步方法的其他线程将暂停执行，直到第一个线程处理完该对象；
- 其次，当一个同步方法退出时，它自动与同一个对象的同步方法的任何后续调用建立 happens-before 关系。这保证了对对象状态的更改对于所有线程都是可见的。

注意，构造函数不能同步，在构造函数中使用 synchronized 关键字是语法错误。同步构造函数没有意义，因为只有创建对象的线程才能在构造对象时访问它。

----

Warning：在构造一个将在线程之间共享的对象时，要非常小心，不要过早地“泄漏”对象的引用。例如，假设您希望维护一个名为 instances 的 List，其中包含类的每个实例。你可能会想在你的构造函数中添加下面的行：

```java
instances.add(this);
```

但是其他线程可以在对象构造完成之前使用实例访问该对象。

----

同步方法为防止线程干扰和内存一致性错误提供了一个简单的策略：如果一个对象对多个线程可见，那么对该对象变量的所有读取或写入都是通过同步方法完成的。（一个重要的例外：`final` 字段在对象被构造后不能被修改，一旦对象被构造，就可以通过非同步方法安全地读取）。这种方式非常有效，但是也会带来 `liveness` 的问题，后面会介绍。



### Intrinsic Locks and Synchronization  隐式监视器锁和同步机制

`Synchronization` 机制是构建于一个内在的实体（也被称为 `intrinsic lock` 或者 `monitor lock`）来实现的。API 文档中通常用 `"monitor"` 来指代它。Intrinsic locks 在同步机制中发挥两个作用：强制对对象状态的独占访问（排它锁），并建立对可见性至关重要的 happens-before 关系。

每个对象都有一个和其关联的 Intrinsic lock。通常来讲，如果一个线程需要一致性访问某个对象的字段，则必须在访问对象字段之前获取对象的隐式锁，并在使用完字段后及时释放掉隐式锁。线程在获取锁和释放锁之间的时间内持有隐式锁。只要一个线程拥有一个内在锁，其他线程就不能获得相同的锁。另一个线程在试图获取锁时将阻塞。

当线程释放一个隐式锁时，就会和后续获取该锁并执行的操作语句建立 happens-before 关系

#### 同步方法锁

当一个线程调用某个同步方法时，它自动获取该方法对象的内在锁，并在方法返回时释放它。即使方法的返回是由未捕获的异常引起的，也会发生锁释放。

您可能想知道调用静态同步方法时会发生什么，因为静态方法关联的是类，而不是方法对象。在这种情况下，线程获得与类关联的 Class 对象的内在锁。因此，对类的静态字段的访问由一个不同于类的任何实例的锁控制。

#### 同步语句/同步代码块

另一种同步机制就是 `synchronized statements`。它不像同步方法那样会自动获取相关的隐式锁，开发者必须显式提供一个隐式锁给同步代码块：

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

在这个例子中，addName 方法需要同步的改变 lastName 和 nameCount 属性，但是需要避免同步调用其他对象的方法（在同步代码块中调用其他对象的方法会导致 `Liveness` 问题，后续讲解）。

如果没有同步语句，就必须有一个单独的非同步方法，其唯一目的是调用 nameList.add。

同步代码块可以通过提供细粒度（`fine-grained` synchronization）的同步保证来优化程序的并发性。参考下面的例子：

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```

在 MsLunch 类中有两个实例字段 c1 和 c2，它们从来没有一起使用过。对字段的更新操作必须是同步保证的，但是又没有必要阻止 c1 和 c2 的更新操作交织在一起 —— 如果那样做就会导致不必要的 blocking 进而降低并发性。

值得一提的是这里没有使用同步方法或者同步代码块，也就是说没有利用和对象相关的隐式监视器锁或者和 this 相关的 Class 对象的隐式锁，而是额外创建了两个对象来充当锁。

使用这种方式必须非常小心，你必须可以保证对这两个字段的交叉访问是安全的。

#### Reentrant Synchronization 可重入机制

前面提到过，一个线程不能获得另一个线程拥有的锁。但是线程可以获得它已经拥有的锁。如果一个线程可以多次获得相同的锁，这种清空就叫做 `reentrant synchronization`（可重入同步机制）。

这种情况大概是这样的：在某个同步代码块中直接或者间接地调用了包含同步代码块的方法，并且两组代码块使用相同的锁。

如果没有 `reentrant synchronization`，同步代码块将不得不采取许多额外的预防措施，以避免线程在获取相同的锁时阻塞（常规的同步代码块如果获取一把锁后，再去获取已经拥有的这把锁，就会阻塞，因为这把锁已经有持有者了）。

### Atomic Access 原子访问

在程序中，一个 `atomic action` 是指只会成功生效一次的动作。原子操作不会在操作中间停止，它只有两种结果：要么一次全部成功，要么都失败。在操作完成之前，原子操作的副作用是不可见的。

前面提到这样的执行语句：`c++`，它不是原子操作，因为它包含了三个 action，且没有同步保证执行顺序。

但是这种声明语句的方式告诉我们，即使是非常简单的表达式也可以定义复杂的操作，这些操作可以分解为其他操作。但是你也可以指定一些操作是原子的：

-  对引用变量以及原始数据类型的读/写是原子的；
- 对使用 `volatile` 修饰符修饰的变量的读/写是原子的；

原子操作是不能交织的，这就意味着不必担心发生线程干扰。但是这并不意味着执行一组原子操作就不需要同步保障了，因为也有可能发生内存一致性错误。

使用 volatile 变量可以有效降低内存一致性风险，因为对 volatile 变量的写操作和后续对该变量的所有读操作会建立起 happens-before 关系。这就意味着 volatile 变量的改变对所有线程是可见的。更重要的是，当一个线程想要读取 volatile 变量时，它看到的不仅是 volatile 变量最新的状态，还有导致这种变化的代码的副作用（原文： it sees not just the latest change to the `volatile`, but also the side effects of the code that led up the change.）。

使用简单的原子变量访问要比使用同步代码块更高效，但是也需要更加小心避免内存一致性错误。使用哪种方式还是要取绝于应用程序的大小和复杂性。

`java.util.concurrent` 包提供了一些不依赖同步机制的原子方法，后面会在高级 API 中讲解。

## Liveness 活性

A concurrent application's ability to execute in a timely manner is known as its `liveness`.

一个并发程序能够及时执行的能力被称为 `liveness`；

本小节介绍 liveness 的常见问题：`deadlock`（死锁），已经进一步简单的介绍 liveness 的另外两个问题：`starvation and livelock`。

### Deadlock 死锁

`Deadlock` 描述了这样一种现象：两个或更多的线程一直阻塞，都在等待彼此。下面是一个例子：

```java
public class Deadlock {
    static class Friend {
        private final String name;
        public Friend(String name) {
            this.name = name;
        }
        public String getName() {
            return this.name;
        }
        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s"
                + "  has bowed to me!%n", 
                this.name, bower.getName());
            bower.bowBack(this);
        }
        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s"
                + " has bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new Runnable() {
            public void run() { alphonse.bow(gaston); }
        }).start();
        new Thread(new Runnable() {
            public void run() { gaston.bow(alphonse); }
        }).start();
    }
}
```

Alphonse 和 Gaston 是 friends，并且都很有礼貌，有这样一种规则，当你向朋友鞠躬时，你必须一直鞠躬，直到你的朋友有机会回礼。但是这条规则没有考虑到两个朋友同时向对方鞠躬的可能性。上面的代码模拟了这种可能性。

当程序运行时，两个线程极有可能在试图调用 `bowBack` 方法时发生死锁，两个代码块都不会退出，因为每个线程都在等待另一个线程退出。

详细分析一下是这样的，例子中使用了同步方法，锁就是和 Method Object 关联的隐式监视器对象，bow 和 bowBack 方法使用的锁都是同一个；

解决方法也很简单，细化锁即可：

```java
private final Object l1 = new Object();
private final Object l2 = new Object();

public void bow(Friend bower) {
    synchronized (l1) {
        System.out.printf("%s: %s has bowed to me!%n", this.name, bower.getName());
        bower.bowBack(this);
    }
}

public void bowBack(Friend bower) {
    synchronized (l2) {
        System.out.printf("%s: %s has bowed back to me!%n", this.name, bower.getName());
    }
}
```

### Starvation and Livelock 饥饿和活锁

相比于死锁而言，Starvation 和 Livelock 发生的要更少一些，但也是每个并发软件的设计者都可能遇到的一些问题。

#### Starvation

Starvation 描述了这样的情况：线程无法获得对共享资源的常规访问，并且无法进行进一步的操作。当共享资源被某些 "贪婪的" 线程长时间使用的时候，其他线程无法获得该资源，此时就会发生 Starvation。

比如说，某个对象提供一个同步方法，该方法会执行很长时间后才会 return。如果一个线程频繁调用此方法，其他同样需要频繁同步访问同一对象的线程通常会被阻塞。

#### Livelock

一个线程经常对另一个线程的动作做出响应。如果另一个线程的动作也是对另一个线程动作的响应，那么就会出现 `livelock` 的情况。和 deadlock 情况一样，出现了 livelock 时所有相关的线程也无法做进一步操作。但是线程并不会阻塞 —— 他们只是忙着回复对方，无暇继续工作。

这就好比两个人在走廊里擦肩而过：Alphonse 向左移动让 Gaston 过去，Gaston 向右移动让 Alphonse 过去，但是并不能解决问题，所以 Alphonse 转而向右移动，而 Gaston 转而向左移动，不断循环下去。

## Guarded Blocks

线程往往需要协调它们的 actions。最常用的协调手段是 `guarded block`。这样的代码块往往首先需要轮询一个 condition，条件为 true 时才会继续执行。要正确地做到这一点，需要遵循许多步骤。

假设有个方法叫做 `guardedJoy`，它能否运行取绝于某个线程共享变量 `joy`，只有当其他线程将该变量设置为某个值时，guardedJoy 才可以继续执行。理论上，这样的方法可以简单地循环直到满足条件，但这种循环是浪费的，因为它在等待时连续执行。

```java
public void guardedJoy() {
    // Simple loop guard. Wastes
    // processor time. Don't do this!
    while(!joy) {}
    System.out.println("Joy has been achieved!");
}
```

一个更高效的 guard 就是通过调用 `Object.wait` 方法来阻塞当前线程。在方法内部调用 wait，方法并不会立即 return 而是等待其他某个线程发出通知，告诉我们某个事件发生了 —— 虽然不一定是这个线程正在等待的事件：

```java
public synchronized void guardedJoy() {
    // This guard only loops once for each special event, which may not
    // be the event we're waiting for.
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

---

注意：记住必须要在循环内部调用 wait 方法，这样 wait 返回后又可以检查一下条件是否成立（可以避免线程虚假唤醒问题）。不要假设中断是为了您等待的特定条件,或者情况仍然是正确的。

---

就像其他可以挂起线程的方法一样，wait 也可以抛出 `InterruptedException`。在上面的例子中我们可以忽略这个异常，因为我们关心的是条件 joy 是否成立。

还有，为什么第二个方法是 synchronized？假设我们要通过对象 d 调用其 wait 方法，就必须获取和 d 关联的隐式锁 —— 否则会抛出错误。将 wait 方法方在同步方法中是一种很简单的获取隐式锁的方式。

当 wait 方法被调用时，线程会释放 d 的隐式锁然后挂起当前线程。在未来的某个时刻，另一个线程会获取相同的隐式锁并调用 `Object.notifyAll` 方法，通知所有在锁上等待的线程发生了重要的事情：

```java
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

在其他线程释放这个锁后的某个事件，第一个线程又重新获取了这个锁，然后从 wait 方法中返回。

---

注意：这里还有第二种通知方法：`notify`，它只能唤醒一个在等待的线程，但是要注意 notify 并不能指定唤醒某个线程，因此它在大型的并发程序中才会非常有用 —— 也就是说，具有大量线程的程序都在做类似的工作。在这样的应用程序中，您不关心哪个线程被唤醒。

我们可以使用 `guarded block` 来创建简单的 `Producer-Consumer` 程序（生产者-消费者模型），这样的程序在两种线程之间共享数据：生产者线程主要用于生产数据，消费者线程主要用于消费数据。这两种线程通过共享对象来进行沟通。因此线程之间的协调就非常重要：在生产者线程交付数据之前，消费者线程不能尝试检索数据，如果消费者没有检索旧数据，生产者线程也不能尝试交付新数据。

下面的例子中，数据就是文本消息序列，它们通过一个 Drop 类型的对象共享：

```java
public class ProducerConsumerModel {
    
    static class Drop {
        private String message;
        private boolean empty = true;
        
        public synchronized String take() {
            while (empty) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            empty = true;
            notifyAll();
            return message;
        }
        
        public synchronized void put(String message) {
            while (!empty) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            empty = false;
            this.message = message;
            notifyAll();
        }
    }
    
    static class Producer implements Runnable {
        private Drop drop;
        
        public Producer(Drop drop) {
            this.drop = drop;
        }

        @Override
        public void run() {
            String[] importantInfo = {
                    "Mares eat oats", "Does eat oats", "Little lambs eat ivy", "A kid will eat ivy too"
            };
            Random random = new Random();

            for (int i = 0; i < importantInfo.length; i++) {
                drop.put(importantInfo[i]);
                
                try {
                    Thread.sleep(random.nextInt(5000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            drop.put("DONE");
        }
    }
    
    static class Consumer implements Runnable {
        private Drop drop;

        public Consumer(Drop drop) {
            this.drop = drop;
        }

        @Override
        public void run() {
            Random random = new Random();
            for (String message = drop.take(); !message.equals("DONE"); message = drop.take()) {
                System.out.printf("MESSAGE RECEIVED: %s%n", message);
                try {
                    Thread.sleep(random.nextInt(5000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Drop drop = new Drop();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```



进度：

- https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html

