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





进度：

- https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html