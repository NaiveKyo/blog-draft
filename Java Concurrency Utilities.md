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



进度：

- https://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/index.html