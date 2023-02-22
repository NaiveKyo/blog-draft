# New I/O In JDK 7

参考地址：

- https://openjdk.org/projects/nio/
- https://openjdk.org/projects/nio/presentations/TS-5686.pdf
- https://openjdk.org/projects/nio/presentations/TS-5052.pdf
- https://openjdk.org/projects/nio/presentations/TS-4222.pdf
- https://jenkov.com/tutorials/java-nio/index.html



## Preface

学习新的 File System API、Asynchronous I/O 以及其他更改后的 API。

包含以下 Topic：

- File System API；
- Channels API：
  - socket channel API；
  - Asynchronous I/O；
- Miscellaneous Topics；
- Conclusion；

## File System API

包：`java.nio.file`、`java.nio.file.attribute`；

主要的 classes：

- FileSystem：
  - 文件系统的抽象；
  - 也是一个工厂类用于创建访问文件或操作系统中的其他对象的相关对象；
- FileRef：
  - 对 file 或 directory 的引用；
  - 定义了一系列操作文件或目录的方法；
- Path：
  - 通过系统路径定位一个文件的 FileRef；
  - 由 FileSystem 创建的用于转换路径字符串或者 URI 的工具；
- FileStore：
  - 底层的存储池、设备、分区 等等......

### Hello World

```java
import java.nio.file.*;

Path home = Path.get("/home/gus");
Path profile = home.resolve(".bash_profile");
// Backup existing file
profile.copyTo(home.resolve(".bash_profile.backup"));
// Append useful stuff to the profile
WritableByteChannel ch = profile.newSeekableByteChannel(APPEND);
try {
    appendStuff(ch);
} finally {
    ch.close();
}
```

### Overview of  Path

Path 中定义了如下方法：

- 访问 components（file or directory）的方法；
- 测试和比对 component 的方法；
- 合并 path；
- 文件操作；
- 所有涉及到访问 file 的方法都会抛出 IOException；
  - 当前 API 中没有其他的 check exception；

### Opening/Creating Files

- Stream I/O：
  - `newInputStream` 和 `newOutputStream`；
  - 这两个同 java.io 交互；

- Channel I/O：
  - `java.nio.channels.SeekableByteChannel`：
    - ByteChannel 维护了一个位置；
    - Channel 等同于 `java.io.RandomAccessFile`；
    - SeekableByteChannel = ByteChannel + file position；
    - 可以转换为 FileChannel，使用更多和文件系统相关的高级特性，比如 file locking、memory mapped I/O 等等。
  - `newSeekableByteChannel` 方法用于打开或创建文件；
  - `InterruptibleChannel` 语义：
    - 异步关闭和中断；
  - 对文件的操作属性，比如：READ、WRITE、APPEND、TRUNCATE_EXISTING、CREATE 等等；
- 在创建文件时可以提供一个可选的权限属性集；
  - 文件权限是很重要的。

### Copying and Moving Files

两个方法：copyTo 和 moveTo；提供可选的参数用于决定行为；

```java
import static java.nio.file.StandardCopyOption.*;
Path source = Path.get("C:\\My Documents\\Stuff.odp");
Path target = Path.get("D:\\Backup\\MyStuff.odp");
source.copyTo(target);
source.copyTo(target, REPLACE_EXISTING, COPY_ATTRIBUTES);
```

### Symbolic Links

不同操作系统对符号链接（软链接）和硬链接的支持不一样；

```java
Path file = Path.get("/usr/spool");
// read file attributes of the link
BasicFileAttributes attrs = Attributes
    .readBasicFileAttributes(file, false);
if (attrs.isSymbolicLink()) {
    // read target of link
    Path target = file.readSymbolicLink();
    // check /usr/spool == /usr/spool/../var/spool
    assert file.isSameFile(file.resolve(target));
}
```

### Directories

- DirectoryStream：
  - 用于遍历目录中的所有 entries；
  - 支持大目录访问；
  - 提供可选的 filter 用于筛选 entries；
  - 内置的 filter 可以使用通配符或者正则表达式匹配 file name；

```java
Path dir = Path.get("mydir");
DirectoryStream stream = dir.newDirectoryStream("*.java");
try {
    for (DirectoryEntry entry: stream) {
        System.out.println(entry.getName());
    }
} finally {
    stream.close();
}
```

```java
Path dir = Path.get("mydir");
Files.withDirectory(dir, "*.java", new DirectoryAction() {
    public void invoke(DirectoryEntry entry) {
        System.out.println(entry.getName());
    }
});
```

- DirectoryStreamFilters
  - 提供创建常用 filter 的工厂方法；
  - newContentTypeFilter：
    - 通过文件的 MIME 类型过滤 entries；
    - 使用注册的 file type detectors；
  - 通过简单的表达式组合 filter；
- Files.walkFileTree 工具方法
  - 自顶向下遍历目录树；
  - 使用内部迭代：
    - FileVisitor 接口用于在遍历某个 file/director 时做特定处理；
    - 指定遇到符号链接时的处理策略；
  - 支持递归复制、移动、删除、chmod 等等操作；

### File Attributes

- 管理和文件相关的元数据：
  - 时间戳、owner、permissions 等等；
  - 高层次的文件/平台系统声明；
- 方法：
  - 利用 group 管理相关属性；
  - 定义了 `FileAttributeView` 用于便捷的查看文件属性；
    - 一个 view 可以继承或者覆盖某些 view；
    - 可以转换为文件系统表示或者从文件系统表示转换为 view；
  - `BasicFileAttributeView` 提供了基础的属性视图：
    - 其他视图必须继承它，因为所有的实现都要支持基础属性；
  - 定义了多种 FileAttributeView：
    - 用于支持 POSIX、DOS、ACLs 等等；
  - 其他实现支持；

获取特定的视图；

```java
BasicFileAttributeView view =
  file.newFileAttributeView(BasicFileAttributeView.class, true);
// bulk read of basic attributes
BasicFileAttributes attrs = view.readAttributes();
```

```java
PosixFileAttributes attrs =
    Attributes.readPosixFileAttributes(file, true);
String mode = PosixFilePermission.toString(attrs.permissions());
System.out.format("%s %s %s", mode, attrs.owner(), attr.group());
rwxrw-r-- alanb java
    import static java.nio.file.attribute.PosixFilePermission.*;
Attributes.setPosixFilePermissions(file,
                                   OWNER_READ, OWNER_WRITE, GROUP_WRITE, OTHERS_READ);
```

### File Change Notification

有时候应用程序需要监听文件的变化或者特定事件的发生，比如 IDE 软件。

- `WatchService`：
  - 监听注册的对象的变化；
  - 如果可能的话会使用 inotify、FEN 等等机制；
  - 监听服务会提供一个线程池用于接受各种 events；
  - 并发处理事件；
  - 很容易为一个图形化的应用构建监听器接口。

```java
WatchService watcher = FileSystems.getDefault().newWatchService();
Set<StandardWatchEventType> events =
    EnumSet.of(ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);
WatchKey key = dir.register(watcher, events);
for (;;) {
    // wait for key to be signalled
    key = watcher.take();
    // process events
    for (WatchEvent<?> ev: key.pollEvents()) {
        if (event.getType() == ENTRY_MODIFY) {
            :
        }
    }
    // reset key
    key.reset();
}
```

### Interoperability

同旧的 Java.io 交互：

- `java.io.File` 的 getFileRef 方法进行了增强；
- 对 `java.util.Scanner` 和 `java.util.Formatter` 做了增强；

### Provider Interface

Java 的 Provider 机制用于服务发现；

允许以下操作：

- 替换默认的文件系统实现；
- 可以提供定制的文件系统实现；

## Channels API

### Updates to Socket Channel API

- 设计 API 的动机：
  - Network channels 不能完全代表 network socket 的抽象；
  - Forced to mix channel 和 socket API 可以用来：
    - bind、manipulate scoket options ......
  - 可以使用 socket adapter 封装过去的 Socket api 的行为；
  - 不能充分利用平台相关的特殊 socket options；
- 方法：
  - NetworkChannel 是 network socket 的 channel；
  - 定义 bind、getLocalAddress、setOption、getOption 等等方法；
  - 提供新的 channel 实现；
- Multicasting（多点广播）：
  - MulticastChannel：
    - 可以加入 multicast groups 的 NetworkChannel；
  - Implemented by：
    - DatagramChannel；
    - AsynchronousDatagramChannel；

### Asynchronous I/O

- 实现目标：
  - 提供 sockets 和 files 的 Asynchronous I/O API；
  - 尽可能的使用操作系统提供的高效 I/O 工具；
- API：
  - Future Style：
    - 开始执行 I/O 操作，然后程序不会阻塞而是立即返回一个 `java.util.concurrent.Future` 实例；
    - Future 是对程序执行的未来状态的一种抽象，里面包含了用于测试或者等待程序执行完毕的方法；
  - Callback Style：
    - 当执行 I/O 操作时提供一个特殊的 CompletionHandler；
    - I/O 操作执行完毕（或者遇到问题失败）时就会立即调用这个 CompletionHandler；

Future Style：

```java
AsynchronousSocketChannel ch = AsynchronousSocketChannel.open();
// initiate connection
// wait for connection to be established or failure
Future<Void> result = ch.connect(remote);
result.get();
ByteBuffer buf = ...
    // initiate read
    Future<Integer> result = ch.read(buf);
// do something
// wait for read to complete
try {
    int bytesRead = result.get();
    :
} catch (ExecutionExecption x) {
    // failed
}
```

Callback Style：

```java
ByteBuffer buf = ...
    // CompletionHandler invoked when read completes
    ch.read(buffer, ..., new CompletionHandler<Integer,Void>() {
        public void completed(IoFuture<Integer,Void> result) {
            try {
                int bytesRead = result.getNow();
            } catch (IOException x) {
                // error handling
            }
        }
    });
```

> Wath about Threads？

（1）Who invoke the completion handler？

- 启动线程（initiating thread）；
- channel group 的线程池中的线程；

（2）Channel Group

- 封装了执行 I/O 操作所需的所有资源；
- 和一个 thread pool 关联；
- 每个 asynchronous channel 都绑定一个 group；
  - 可以是 default group；
  - 或者在创建 channel 时指定一个 group；
- 配置参数：
  - ThreadFactory；
  - 处理 I/O events 的最大线程数量；
  - .....

> Other Asynchronous I/O topics

- Timeouts；
- Asynchronous close；
- Cancellation；
- Provider Interface；
  - SPI，自定义实现；



# Asynchronous I/O

## Summary

目录：

章节一：

- Overview of Asynchronous I/O API；
- Demultiplexing I/O events and thread pools；（多路复用）
- Usage notes and other topics；

章节二：

- Grizzly Architecture；
- Thread Pool Strategies；
- Tricks；

总结；

## Concept

核心点：

- 启动 no-blocking I/O 操作；
- 在 I/O 操作执行完毕时发出通知；

- 和 no-blokcing asynchronous I/O 进行比较：
  - notification when channel ready for I/O (Selector)；（Selector 会调度处于就绪状态的 channel）
  - perform non-blocking I/O operation；
  - Reactor vs. Proactor pattern；

## Two forms

（1）正常启动非阻塞 I/O 操作时：

返回 Future 表示正在阻塞运行的线程；

```java
AsynchronousSocketChannel ch = ...
ByteBuffer buf = ...
Future<Integer> result = ch.read(buf);

AsynchronousSocketChannel ch = ...
ByteBuffer buf = ...
Future<Integer> result = ch.read(buf);
// check if I/O operation has completed
boolean isDone = result.isDone();

AsynchronousSocketChannel ch = ...
ByteBuffer buf = ...
Future<Integer> result = ch.read(buf);
// wait for I/O operation to complete
int nread = result.get();

AsynchronousSocketChannel ch = ...
ByteBuffer buf = ...
Future<Integer> result = ch.read(buf);
// wait for I/O operation to complete with timeout
int nread = result.get(5, TimeUnit.SECONDS);
```

（2）在启动非阻塞 I/O 操作时指定 CompletionHandler：

I/O 操作执行完毕（或者失败）时自动调用 Handler；

```java
interface CompletionHandler<V,A> {
    void completed(V result, A attachment);
    void failed(Throwable exc, A attachment);
}
```

- 泛型 V 表示程序执行结果；
- 泛型 A 表示 I/O 操作附加的 object 的类型；
  - 可以继续传递给 context；
  - 通常封装了 connection context；
- 方法 completed：I/O 操作成功执行后就会调用该方法；
- 方法 failed：I/O 操作失败后就会调用该方法；

```java
class Connection { … }
class Handler implements CompletionHandler<Integer,Connection> {
    public void completed(Integer result, Connection conn) {
        int nread = result;
        // handle result
    }
    public void failed(Throwable exc, Connection conn) {
        // error handling
    }
}

AsynchronousSocketChannel ch = ...
ByteBuffer buf = ...
Connection conn = ...
Handler handler = ...
ch.read(buf, conn, handler);
```

## AsynchronousSocketChannel

- Asynchronous connection；
- Asynchronous read/write；
- Asynchronous scatter/gather（multiple buffers）；
- Read/Write operations support timeout；
  - failed method invoked with timeout exception；
- Implements NetworkChannel
  - for bindin, setting socket options, etc；
- Asynchronous accept：
  - handler invoked when connection accepted；
  - Result is AsynchronousSokcetConnection；

## AsynchronousDatagramChannel

从 JDK 8 开始移除该 API，参考：

https://bugs.java.com/view_bug.do?bug_id=6993126

大概意思是：最初引进这个 API 是为了和其他 API 保持一致（其他 api 也提供了 Asynchronous 实现），但是后来发现用的很少，所以 JDK7 以后就移除了该 API，我们可以使用 DatagramChannel。

## AsynchronousFileChannel

- Asynchronous read/write；
- No global file position/offset；（这一点和常规的 Channel 不一样）
  - Each read/write specifies position in file；（在每一次读写过程中维护 position）
  - Access different parts of file concurrently；（并发访问文件不同的部分，可以借此特性实现文件分片上传）；

```java
Future<Integer> result = channel.write(buf, position);
doSomethingElse();
int nwrote = result.get();
```

- Open method specifies options：（打开文件时可以指定选项）
  - READ, WRITE, TRUNCATE_EXISTING, …
  - No APPEND；（注意和 nio.2 不一样这里没有提供向文件追加内容的操作）
  - Can specify initial attributes when creating file；
- Also supports file locking, size, truncate, ...（和 FileChannel 提供的特性差不多）

## Groups

- What threads invoke the completion handlers？
- Network oriented channels bound to a group
  - AsynchronousChannelGroup
- Group encapsulates thread pool and other shared resources
- Create group with thread pool
- Default group for simpler applications
- Completion handlers invoked by pooled threads
- AsynchronousFileChannel can be created with its own thread pool (group of one)

Creating a Group：

```java
// fixed thread pool
ThreadFactory myThreadFactory = …
int nthreads = …
AsynchronousChannelGroup group = AsynchronousChannelGroup
 .withFixedThreadPool(nThreads, threadFactory);
```

```java
// custom thread pool
ExecutorService pool = ...
AsynchronousChannelGroup group = AsynchronousChannelGroup
 .withThreadPool(pool);
AsynchronousSocketChannel channel =
 AsynchronousSocketChannel.open(group);
```

## Threads

下面了解一些线程池方面的知识：

（1）Fixed Thread Pool

- Fixed thread pool
  - Each thread waits on I/O event
  - do I/O completion
  - invoke completion handler
  - go back to waiting for I/O events（线程池复用线程）

所谓的 fixed thread pool 处理流程是这样的：

- 等待 I/O 事件；
- 接受 I/O 事件并执行 I/O 操作；
- 执行完毕后调用 completion handler；
- 释放相关资源，回到线程池继续等待下一个 I/O event；

（2）Cached Or Custom Thread Pool

- Cached or custom thread pool
  - Internal threads wait on I/O events
  - Submit tasks to thread pool to dispatch to completion handler

所谓的 Cached or custom thread pool 指：

- 程序执行 I/O 操作会开启一个线程去执行；
- I/O 操作执行完毕后提交调用 completion handler 的任务到指定线程池中；
- 指定线程池维护多个线程用来执行 handler；

## More on CompletionHandlers

- Should complete in a timely manner（必须在指定的时间内调用 handler）
  - Avoid blocking indefinitely（防止线程无线阻塞）
  - Important for fixed thread pools（对前面提到的第一种线程池很重要）
- May be invoked directly by initiating thread（I/O 启动线程可能会直接执行 handler）
  - when I/O operation completes immediately, and initiating thread is pooled thread；
  - – may have several handler frames on thread stack，implementation limit to avoid stack overflow
- Termination due to uncaught error or runtime exception causes pooled thread to exit



## ByteBuffers

- Not safe for use by multiple concurrent threads（多线程下使用 ByteBuffer 是不安全的）
- When I/O operation is initiated then must take great care not to access buffer until I/O operation completes（在 I/O 操作刚启动时千万注意不要 access buffer，除非 I/O 操作执行完毕才可以访问）
- Memory requirements for buffers depends on number of outstanding I/O operations（buffer 需要的内存空间取决于未完成 I/O 操作的数量）
- Heap buffers incur additional copy per I/O（如果使用的是 heap buffer，那么每次 I/O 都会复制一份 buffer）
  - As per SocketChannel API, compare performance 
  - Copy performance and temporary direct buffer usage improved



## Other topics

- Queuing not supported on stream connections（在常规 stream I/O 中不支持队列）
  - A short-write would corrupt the stream
  - Handlers not guaranteed to be invoked in order
  - Read/WritePendingException to catch bugs
- Asynchronous close
  - Causes all outstanding I/O operations to fail
- Cancellation
  - Future interface defines cancel method
  - Forceful cancel allows to close channel



## Grizzly Architecture



## Which Thread Pool strategy？

- With AIO, you can configure the thread pool (ExecutorService) used by both the AIO kernel and your application or use the preconfigured/built in Thread Pool that comes by default

```java
AsynchronousChannelGroup.withCachedThreadPool
 (ExecutorService, initialSize)
AsynchronousChannelGroup.withThreadPool
 (ExecutorService)
AsynchronousChannelGroup.withFixedThreadPool
 (nThread, ThreadFactory)
```

### FixedThreadPool

- An asynchronous channel group associated with a fixed thread pool of size N creates N threads that are waiting for already processed I/O events.
- The kernel dispatches events directly to those threads:
  - Thread first completes the I/O operation (like filling a ByteBuffer during a read operation). 
  - Next invoke the `CompletionHandler.completed()` that consumes the result.
  - When the CompletionHandler terminates normally then the thread returns to the thread pool and wait on a next event. 
- Avoid blocking/long lived operations inside a completion handler.
- If not possible, either use a CachedThreadPool or another ExecutorService that can be used from a completion handler.

### Deadlock

- What about if all threads "dead lock" inside a CompletionHandler？
  - you entire application can hang until one thread becomes free to execute again.
  - The kernel is no longer able to EXECUTE anything！
- Hence this is critically important CompletionHandler complete in a timely manner and avoid blocking.
- If all completion handlers are blocked, any new event will be queued until one thread is 'delivered' from the lock.
- Avoid blocking operations inside a completion handler. If not possible, either use a  CachedThreadPool or another ExecutorService that can be used from a completion handler.

### CachedThreadPool

- An asynchronous channel group associated with a cached thread pool submits events to the thread pool that simply invoke the user's completion handler. 
- Internal kernel's I/O operations are handled by one or more internal threads that are not visible to the user application. 
- That means you have one `hidden thread pool` that dispatches events to a cached thread pool, which in turn invokes completion handler
- 注意，这样做有一个好处： **a thread's context switch for free!**

### OOM

- Probability of suffering the hang problem compared with the FixedThreadPool is lower.（和 FixedThreadPool 相比，CachedThreadPool 出现挂起问题的概率要小一些）
- Still might grow infinitely…（但是也会出现）
- At least you guarantee that the kernel will be able to complete its I/O operations (like reading bytes).（但是能够确保操作系统内核能够完成 I/O 操作）
-  Oops…CachedThreadPool must support unbounded queuing to works properly.（但是 CachedThreadPool 需要支持无界队列，比如 LinkedBlockingQueue 无参构造）
- **So you can possibly lock all the threads and feed the queue forever until OOM happens.** （可以锁定所有线程，然后后续的工作任务就会不断加入到队列中，直到 OOM）
- Avoid blocking/long lived operations inside a completion handler.
- Possibility of OOM if the queue grow indefinitively => monitor the queue

### Kernel/default thread pool

- Hybrid of the above configurations：
  - Cached thread pool that creates threads on demand；（使用 CachedThreadPool 时根据需要创建线程）；
  - N threads that dequeue events and dispatch directly to CompletionHandler
- N defaults to the number of hardware threads.
- In addition to N threads, there is one additional internal thread that dequeues events and submits tasks to the thread pool to invoke completion handlers.

### AsynchronousSocketChannel.read()

Once a connection has been accepted, it is now time to read some bytes:

```java
AsynchronousSocketChannel.read(ByteBuffer b,
 Attachement a,
CompletionHandler<> c);
```

看到这个方法，可以对比 NIO 中的 SelectionKey.attach() 方法。

考虑以下问题：

- Let’s say you get 10 000 accepted connections
- Hence 10 000 ByteBuffer created, and the read operations get invoked
- – Now we are waiting, waiting, waiting, waiting for the remote client(s) to send us bytes (slow clients/ network)
- Another 10 000 requests comes in, and we are again creating 10 000 ByteBuffer and invoke the read() operations.
- OOM

## Use ByteBuffer pool & Throttle

- Let’s not be too negative here. So far we have tested with more than 20 000 clients without any issues
- But this is still something you have to keep in mind!!
- Might want to throttle the read() operation to avoid the creation of too many ByteBuffer
-  We strongly recommend the use of a ByteBuffer pool, specially if you are using Heap ByteBuffer (more on this later). 
-  Get a ByteBuffer before invoking the read() method, and return it to the pool once the read operations complete.



## Blocking AsynchronousSocketChannel.read()

- When invoking the read operation, the returned value is a Future:

```java
Future readOp = AsynchronousSocketChannel.read(…);
readOp.get(30, TimeUnit.SECONDS);
```

- The Thread will block until the read operation complete or times out
- **Be careful as you might lock your ThreadPool (specially with FixedThreadPool)**

## AsynchronousSocketChannel.write()

```java
AsynchronousSocketChannel.write(ByteBuffer b,
 Attachement a,
CompletionHandler<> c);
```

- Since we are asynchronous, invoking write(..) will not block, so the calling thread can continue its execution.
- What happens when the calling thread invokes the write method again and the CompletionHandler has not yet been invoked by the previous write call?

- You get a **WritePendingException**
- Hence when invoking the write operation, make sure the CompletionHandler.complete() has been invoked before initiating another write. 
- Better, store ByteBuffer inside a queue and execute write operations only when the previous one has completed (will show code soon)
- As for read, we strongly recommend the use of a ByteBuffer pool for executing write operations. Get one before writing, put it back to the pool after.



## Damned ByteBuffer!

- If you are using Heap ByteBuffer, be aware the kernel will copy the bytes into a direct ByteBuffer during every write operation:
- Direct ByteBuffer performance have significantly improved with JDK 7, so use them all the time.
- Scattered ByteBuffer write operations still offer you free copy, using direct ByteBuffer or not!

## AsynchronousFileChannel.open()

Before, the nightmare:

```java
File f = new File();
FileOutputStream fis = new
FileOutputStream(f);
FileChannel fc = fis.getChannel();
fc.write(…);
```

 Now, the paradise:

```java
afc = AsynchronousFileChannel.open(Path
file, OpenOption... options);
afc.write(…);
```



# Extension：Java NIO Framework

Java NIO 框架：

- Mina；Apache 项目

- Grizzly；Sun 公司项目，是 NIO 的一层简单封装；
- Netty；Jboss 项目，目前最流行；



# Tutorial

https://jenkov.com/tutorials/java-nio/index.html

# Conclusion

总结 Java NIO 的几个核心概念（加粗为核心组件）：

（1）**Channels**；

（2）**Buffers**；

（3）Scatter - Gather；

（4）Channel to Channel Transfers；

（5）**Selectors**；

（6）FileChannel；

（7）SocketChannel；

（8）ServerSocketChannel；

（9）Non-blocking Server Design；

（10）DatagramChannel；

（11）Pipe；

（12）NIO vs IO；

（13）Path；

（14）Files；

（15）AsynchronousFileChannel；

