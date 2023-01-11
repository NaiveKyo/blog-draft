# Preface

> 本文涉及到的内容：Java I/O、NIO，and NIO.2

Java 8 中关于 I/O 的在 `java.io` 和 `java.nio` 两个包下面，主要包含以下特性：

- 通过数据流、序列化以及文件系统进行输入和输出；
- 字节和 Unicode 字符之间转换时使用的工具：字符集、解码器和编码器；
- 访问文件、文件属性和文件系统；
- 用于构建可伸缩服务的 API：异步或多路复用、非阻塞 I/O；

# API Specification

简单介绍一下不同 API 的功能：

- `java.io`：支持系统输入输出和对象序列化，到文件系统；
- `java.nio`：为大容量内存操作提供缓冲区（buffers）。为了获得高性能，可以在 `direct memory`（堆外内存，不被 jvm 管理） 中申请和分配缓冲区；
- `java.nio.channels`：
  - 定义 channels（通道），它是对能执行 I/O 操作设备的一种抽象；
  - 定义了 selectors（选择器）用于多路复用（multiplexed）、非阻塞 I/O；
- `java.nio.channels.spi`：提供了 channels 的实现；
- `java.nio.file`：定义访问文件和文件系统的接口与类；
- `java.nio.file.attribute`：定义用于访问文件系统属性的接口和类；
- `java.nio.file.spi`：定义用于创建文件系统实现的类（服务发现机制）；
- `java.nio.charset`：定义字符集、解码器和编码器，在字节和 Unicode 字符转换时使用；
- `java.nio.charset.spi`：提供字符集的实现（服务发现机制）；
- `com.sum.nio.sctp`：Java 定义的关于 Stream Control Transport Protocol（流控制传输协议）的 API；

# Basic I/O

本小节介绍以下知识：

- Java 平台和基础 I/O 有关的类，它提供并强调 I/O 流的概念，I/O 流可以极大的简化 I/O 操作；
- 同时也会关注序列化操作，它允许程序将整个对象写到流中，然后再读回来；
- 最后涉及到文件 I/O 和文件系统操作，包括随机访问文件；

大部分类在 `java.io` 和 `java.nio.file` 包下。

# I/O Streams

## Byte Streams

Byte Stream 主要用于处理原始二进制数据；

程序使用字节流来执行 8 位字节的输入和输出，所有字节流都起源于 InputStream 和 OutputStream。

看下面的示例代码，注意为了减少代码量，这里直接将异常抛给 JVM，实际情况下应该手动 catch：

```java
public class UseCase1ByteStreams {
    public static void main(String[] args) throws IOException {
        // 演示复制文件
        FileInputStream in = null;
        FileOutputStream out = null;
        
        try {
            in = new FileInputStream("origin.txt");
            out = new FileOutputStream("target.txt");
            int c;
            
            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            if (in != null)
                in.close();
            if (out != null)
                out.close();
        }
    }
}
```

这段代码演示了如何进行文件的复制，从  origin.txt 源文件中一个字节一个字节的读取，然后一个字节一个字节的写出到目标文件 target.txt；显而易见，这里将会花费大量时间读取输入流和写出到输出流，因为每次只读取一个字节；

（补充：一个字节 8 位，可以表示的数字范围是 0 - 255，`read()` 方法读取到末尾会返回 -1）

> Always Close Streams

当不需要使用流时记得及时关闭它，这是很重要的一点。在上面的代码例子中我们在 finally 块中确保无论发生什么都会正常关闭流，这是为了防止资源泄露。（因为 I/O 流涉及到系统资源，不及时释放就会一直占用资源）

上面的例子看起来是 IO 流一个常规应用，但是它实际上是一种我们应该避免的低级 I/O。因为文件中会包含字符数据，更好的处理方法是使用字符流。Byte Stream 应该只用于最基本的 I/O。

为什么介绍字节流呢？因为所有的流都是建立在字节流上的。

## Character Streams

Character Streams 主要处理字符数据的 I/O，自动将 Unicode 字符数据和本地字符集之间进行转换。

Java 平台使用 Unicode 编码格式来存储字符值，Character Stream I/O 自动将这种内部格式和本地字符集进行互相转换。在西方地区，本地字符集通常是 ASCII 码的 8-bit 超集。（在中国本地字符集一般就是 UTF-8 了）

对于大多数应用程序而言，使用字符流进行 I/O 并不比使用字节流 I/O 复杂，字符流将 Unicode 字符转换为本地字符集，使用字符流代替字节流的程序可以自动适应本地字符集，并为国际化做好准备 —— 所有这些都不需要开发者关心。

如果应用不强调国际化，那么就可以更简单的使用字符流，而不必关心字符集的问题，当然，如果程序要进行国际化，那么我们的程序可以在不进行大量重新编码的情况下进行调整。

看下面的例子：

```java
public class UseCase2CharacterStreams {
    public static void main(String[] args) throws IOException {
        // 使用字符流复制文件
        FileReader fr = null;
        FileWriter fw = null;
        try {
            fr = new FileReader("origin.txt");
            fw = new FileWriter("target.txt");
            
            int c;
            while ((c = fr.read()) != -1) {
                fw.write(c);
            }
        } finally {
            if (fr != null)
                fr.close();
            if (fw != null)
                fw.close();
        }
    }
}
```

这个例子和前面的例子很像，无非是把 FileInputStream/FileOutputStream 换成了 FileReader/FileWriter，两个依旧是使用 int 变量进行读写，但是不同的是前者的 int 变量保存的是 后 8 位，范围从 0-255，后者保存的是后 16 位，范围从 0 - 0xFFFF。

Character Stream 其实是 Byte Stream 的一层包装，因为字符流使用字节流来执行物理层面的 I/O，字节流读取字节，然后字符流将字节转换为字符。

有两种通用的字节到字符的 "桥接" 流：

- `java.io.InputStreamReader`；
- `java.io.OutputStreamWriter`；

如果 Java I/O API 中没有字符流能够满足需求，此时可以使用它们来创建字符流。在 [Java 网络编程](https://docs.oracle.com/javase/tutorial/networking/index.html)中的 [Socket lesson](https://docs.oracle.com/javase/tutorial/networking/sockets/readingWriting.html) 会演示如何从 Socket 读取字节转换为字符流，或者将字符流写入到 Socket 字节流中。

和单个字符的读写相比 Character I/O 通常会以较大的单元出现，比如说 "行"：也就是以行结束符结尾的字符串。行结束符可以是回车/换行序列（`\r\n`）、单个回车（`\r`）或单个换行（`\n`），java  API 支持所有任意操作系统上可能的行终止符。

下面修改一下前面的例子，让它可以按行读取文件内容，此时需要使用两个新的类：

- `BufferedReader` 和 `PrintWriter`，后续会在缓存区和格式化方面深入介绍这两个类；

```java
public static void main(String[] args) throws IOException {
    BufferedReader br = null;
    PrintWriter pw = null;
    try {
        br = new BufferedReader(new FileReader("origin.txt"));
        pw = new PrintWriter(new FileWriter("target.txt"));
        
        String l;
        while ((l = br.readLine()) != null) {
            pw.println(l);
        }
    } finally {
        if (br != null)
            br.close();
        if (pw != null)
            pw.close();
    }
}
```

## Buffered Streams

Buffered Streams 通过减少 native API 的调用来优化输入和输出。

到目前为止，我们使用的大多数是无缓冲 I/O，这意味着每个读或写的请求都由底层操作系统直接处理，此时程序的处理效率十分低下，因为每个这样的请求通常都会触发磁盘访问、网络活动或其他一些相对昂贵的操作。

为了减少这种开销，Java 平台实现了缓冲的 I/O 流。

Buffered InputStream 从一个被称为缓冲区的内存区域读取数据，当缓冲区是空的时候，就会调用 native input api 从磁盘或其他地方读取数据到缓冲区。类似地，Buffered Ouputstream 将数据写入缓冲区，只有当缓存区满的时候才会调用 native output api。

在程序中可以将一个未使用缓冲区的流转换为缓冲流，在 Java I/O 体系中，这是一种很常见的包装操作（也就是设计模式中的包装器模式），只需要将未缓冲的流对象通过构造器传递给缓存流就可以了，就像前面例子中的那样：

```java
br = new BufferedReader(new FileReader("origin.txt"));
pw = new PrintWriter(new FileWriter("target.txt"));
```

有四个 buffered stream 用于包装 unbuffered stream：

- `BufferedInputStream` 和 `BufferedOutputStream` 用于创建缓冲字节流；
- `BufferedReader` 和 `BufferedWriter` 用于创建缓冲字符流；

> Flushing Buffered Streams

在特定的时候写出缓冲区而不是等待缓冲区被填满是有意义的。这就是所谓的缓冲区刷新。

一些缓冲输出类支持自动刷新，由可选的构造函数指定，启用自动刷新时，一些 key event 会触发刷新缓冲区的行为。

比如说 `PrintWriter` 就支持自动刷新，当调用它的 `println` 或者 `format` 方法时，方法调用完成后就会将缓冲区的数据全部自动写出到目的地。更多信息参考 [Formatting](https://docs.oracle.com/javase/tutorial/essential/io/formatting.html)。

想要手动刷新流，只需要调用 `flush` 方法，对任何输出流都是有效的，但是如果是没有使用缓冲区的输出流调用该方法则不会造成任何影响。

## Scanning and Formatting

允许程序以某种格式读取和写入文本。

在编程的时候通常会将数据流转换为人们易于阅读的形式，为了支持这些功能，Java 提供了两个 API：

- scanner API ：扫描器 API 将输入分解为和数据关联的单独的 token；
- formatting API：格式化 API 将数据以易于阅读的形式输出。

### Scanning

`java.util.Scanner` 类型的对象非常实用，通常有两个特性：

- 将输入按照特定的规则切分为单个 token；
- 对单个 token 进行转换；

看下面的例子：

```java
public class UseCase3ScannerAndFormatting {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        while (sc.hasNext()) {
            System.out.println("token: " + sc.next());
        }
        
        sc.close();
    }
}
```

这个例子很简单，从系统标准输入（使用开发工具比如 IDEA 时系统输入就是控制台）读取 token，默认情况下以空格分隔不同的 token，最后输出到标准输出。

别忘了最后也需要调用 close 方法，即使 Scanner 不是流，我们也需要关闭它来释放关联的底层流资源。

如果要换一种分隔符，可以调用 `sc.useDelimiter(",\\s*");` 方法设置，注意参数是模式；

另一个功能就是将 token 转换格式了：

```java
public class UseCase3ScannerAndFormatting {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        while (sc.hasNext()) {
            if (sc.hasNextInt())
                System.out.println("int: " + sc.next());
            else 
                System.out.println("token: " + sc.next());
        }
        
        sc.close();
    }
}
```

### Formatting

实现格式化的流对象是 PrintWriter（字符流）和 PrintStream（字节流）的实例。

注意：System.out 和 System.err 是 PrintStream 的实例，当需要创建格式化输出流时，才会使用 PrintWriter。

像所有的字节和字符流一样，PrintStream 和 PrintWriter 也实现了一组用于简单字节和字符输出的标准写方法，但是除此之外，它们海实现了将内部数据格式化输出的方法。

提供了两种级别的格式化：

- print 和 println 以标准的方式格式化单个 token；
- format 基于格式字符串格式化，具有很多精准的选项。

看下面的例子：

```java
public static void main(String[] args) {
    int i = 2;
    double r = Math.sqrt(i);

    System.out.print("The square root of ");
    System.out.print(i);
    System.out.print(" is ");
    System.out.print(r);
    System.out.print(".");
    
    i = 5;
    r = Math.sqrt(i);
    System.out.println("The square root of " + i + " is " + r + ".");
    
    i = 8;
    r = Math.sqrt(i);
    System.out.format("The square root of %d is %f.%n", i, r);
}
```

更多信息参考：https://docs.oracle.com/javase/tutorial/essential/io/formatting.html

## I/O from the Command Line

程序通常从命令行运行，并在命令行环境中与用户交互。Java平台以两种方式支持这种交互：Standard Streams（标准流）和 Console（控制台）。

### Standard Streams

标准流是许多操作系统的一个特性，默认情况下，它们从键盘读取输入并将输出写入显示器。它们还支持文件和程序之间的 I/O，但该功能由命令行解释器控制，而不是程序。

Java平台支持三个标准流：

（1）Standard Input：通过 `System.in` 支持；

（2）Standard Output：通过 `System.out` 支持；

（3）Standard Error：通过 `System.err` 支持；

这三个对象是默认就创建好的，不需要 new。

你可能会希望标准流是字符流，但是处于历史原因，它们都是字节流，System.out 和 System.err 是 PrintStream 对象，尽管从技术层面上看它是字节流，但是它内部还会使用一个字符流对象来模拟很多字符流的特性。

相比之下，System.in 是一个没有字符流特征的字节流。若要使用标准输入作为字符流，请使用 InputStreamReader 包装 System.in ：

```java
InputStreamReader cin = new InputStreamReader(System.in);
```

### The Console

标准流的一个更高级的替代品是控制台（Console）。这是一个单一的、预定义的 Console 类型对象，它具有标准流提供的大多数特性，以及其他特性。控制台对于安全的密码输入特别有用。Console 对象还通过其 reader 和 writer 方法提供了真正的字符流的输入和输出流。

在程序可以使用控制台之前，它必须尝试通过调用 `System.console()` 方法来检索控制台对象。如果 Console 对象可用，此方法将返回它。如果 System.console 返回 NULL，则不允许进行 console 操作，这可能是因为操作系统不支持这些操作，也可能是因为程序是在非交互环境中启动的。

Console 对象通过其 readPassword 方法支持安全的密码输入。此方法从两方面帮助保护密码输入。首先，它抑制了回显，因此密码在用户的屏幕上不可见。其次，readPassword 返回一个字符数组，而不是一个 String，因此可以覆盖密码，一旦不再需要它就从内存中删除它。

看下面的例子：

```java
public class UseCase4Console {
    public static void main(String[] args) {
        Console c = System.console();
        if (c == null) {
            System.err.println("No Console.");
            System.exit(1);
        }

        String login = c.readLine("Enter your login: ");
        char [] oldPassword = c.readPassword("Enter your old password: ");
        if (verify(login, oldPassword)) {
            boolean noMatch;
            do {
                char [] newPassword1 = c.readPassword("Enter your new password: ");
                char [] newPassword2 = c.readPassword("Enter new password again: ");
                noMatch = !Arrays.equals(newPassword1, newPassword2);
                if (noMatch) {
                    c.format("Passwords don't match. Try again.%n");
                } else {
                    change(login, newPassword1);
                    c.format("Password for %s changed.%n", login);
                }
                Arrays.fill(newPassword1, ' ');
                Arrays.fill(newPassword2, ' ');
            } while (noMatch);
        }

        Arrays.fill(oldPassword, ' ');
    }

    // Dummy change method.
    static boolean verify(String login, char[] password) {
        // This method always returns
        // true in this example.
        // Modify this method to verify
        // password according to your rules.
        return true;
    }

    // Dummy change method.
    static void change(String login, char[] password) {
        // Modify this method to change
        // password according to your rules.
    }
}
```

## Data Streams

Data Streams 处理原始数据类型和字符串的二进制数据 I/O。

原始数据类型：boolean、char、byte、short、int、long、float、double；

所有的 Data Streams 都实现了 `java.io.DataInput` 或者 `java.io.DataOutput` 接口。本小节重点介绍数据流中使用较为广泛的 `DataInputStream` 和 `DataOutputStream`。

看下面的例子：

```java
public class UseCase5DataStreams {
    
    static final String dataFile = "invoicedata";
    
    static final double[] prices = { 19.99, 9.99, 15.99, 3.99, 4.99 };
    
    static final int[] units = { 12, 8, 13, 29, 50 };
    
    static final String[] descs = {
            "Java T-shirt",
            "Java Mug",
            "Duke Juggling Dolls",
            "Java Pin",
            "Java Key Chain"
    };
    
    public static void main(String[] args) throws IOException {

        // 将数据写入到文件
        DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream(dataFile)));

        for (int i = 0; i < prices.length; i++) {
            out.writeDouble(prices[i]);
            out.writeInt(units[i]);
            out.writeUTF(descs[i]);
        }

        out.close();
        
        // 从文件中读取数据
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream(dataFile)));

        double price;
        int unit;
        String desc;
        double total = 0.0;

        try {
            while (true) {
                price = in.readDouble();
                unit = in.readInt();
                desc = in.readUTF();
                System.out.format("You ordered: %d units of %s at $%.2f%n", unit, desc, price);
                total += unit * price;
            }
        } catch (EOFException e) {
        }
        
        in.close();

        System.out.println("total: " + total);
    }
}
```

这里使用数据流将数据写入到文件，然后从文件中读取数据，有几个点需要注意：

- 创建数据流的方式依旧还是熟悉的装饰器；
- 用完流记得及时 close 释放系统资源；
- `writeUTF()` 方法将字符串以 UTF-8 格式写入到文件，UTF-8 是一种可变宽度的字符串，对于常见的英文字符，只需一个字节即可存储（汉字需要 3 个字节，具体可以网上查阅 unicode、utf 等等编码的特性和历史原因）；
- 数据流通过捕获 `EOFException` 来检测文件是否读取结束，而不是返回无效值。DataInput 方法的所有实现都使用 EOFException 而不是返回值；
- 还需要注意，DataStreams 中的每个 write 操作都有专门的 read 操作一一对应，因此我们写入什么类型的数据就要使用对应的方法读取；
- DataStreams 使用了一种非常糟糕的编程技术：它使用浮点数表示货币值，一般来说，浮点数不利于精确的值。
  - 对于货币值的正确类型是 `java.math.BigDecimal`，不幸的是，BigDecimal 是一种对象类型，因此它不能使用 DataStreams，相应的它需要使用 Object Streams。

## Object Streams

处理对象的二进制数据 I/O；

如果说 Data Streams 支持的是原始数据类型的 I/O 流操作，那么 Object Streams 支持的就是对象的 I/O 流操作了。同时大多数（但不是全部）的标准类都支持对象的序列化操作，支持序列化的类实现了 `Serializable` 接口。

一般我们说的对象流是 `ObjectInputStream` 和 `ObjectOutputStream`，这些类实现了 `ObjectInput` 和 `ObjectOutput` 接口，这两个接口又继承自 `DataInput` 和 `DataOutput` 接口，这意味这 DataStreams 中涵盖的所有基本数据的 I/O 方法在 ObjectStreams 中也会实现。因此，对象流可以包含原始数据类型和对象类型的混合。

需要注意的是对象流的 readObject 方法返回的值是 Object 类型，有时候我们需要做运行时类型转换，注意可能发生的类型转换异常（ClassNotFoundException）；

> 复杂对象的 Input 和 Output

对象流接口提供的 `writeObject` 和 `readObject` 方法非常容易使用，但是它们内部包含了一些非常复杂的对象管理逻辑。这对于像 `Calendar` 这样的类并不重要，因为它内部只是封装了原始数据类型的值，但是许多对象包含了其他对象的引用。如果 readObject 方法要从流中重构对象，它必须能够重构原始对象引用的所有对象。这些附加对象内部也可能有对其他对象的引用，以此类推。这种情况下，writeObject 会遍历整个对象引用网络，并将该网络中的所有对象写入流（这里的网络指的是对象引用关系构成的图）。因此，调用一次 writeObject 方法可能会导致大量对象写入流。

注意存在这样一种情况：同一个对象流中的两个对象 a 和 b 都持有对另一个对象 c 的引用，当 a 和 b 被读取时，引用的对象 c 是不是还是同一个？答案是会，一个流中只会包含一个对象的一个副本，尽管它可以包含任意数量的引用，因此，如果显式地将一个对象写入流两次，实际上只写入了两次引用。比如下面的代码：

```java
Object ob = new Object();
out.writeObject(ob);
out.writeObject(ob);
```

每个 writeObject 都必须与一个 readObject 相匹配，所以读取流的代码看起来是这样的：

```java
Object ob1 = in.readObject();
Object ob2 = in.readObject();
```

结果就是会产品两个引用变量：ob1 和 ob2，但是它们都指向同一个对象实例。

但是，如果一个对象被写入两个不同的流，那么就会产生两个重复对象，此时使用一个读取流来读取这两个写入流，就会获得两个对象。

# File I/O（Featuring NIO.2）

`java.nio.file` 包以及其关联的 `java.nio.file.attribute` 包为文件 I/O 和访问默认文件系统提供了全面的支持。尽管这些 API 涉及到很多类，但我们只需要关注几个入口点就可以了，其实这些 API 是非常方便使用的。

首先我们需要知道什么是 Path，`java.nio.file.Path` 是一个非常重要的 entry point，重点看 path 相关的操作；

然后了解 `java.nio.file.Files`，其中包含了与文件相关的操作方法，会介绍许多文件操作的通用概念，然后了解 checking、deleting、copying and moving files；

最后将学习一些非常强大且更高级的主题：

- 递归遍历文件树功能；
- 使用通配符搜索文件；
- 监视目录的更改；
- 介绍其他的一些不常用的方法。

## What's a Path？

文件系统在某种形式的媒体（通常是一个或多个硬盘驱动器）上存储和组织文件，使得它们易于检索。目前使用的大多数文件系统以树状（或层次结构）结构存储文件。树的顶部是一个（或多个）根节点。在根节点下面是一些文件和目录（Windows 中用问价夹表示目录），每个目录下面又可以有文件或者子目录，依次类推，几乎可以拥有无限的深度。

本小节包含三部分：

- 什么是 Path；
- 相对路径和绝对路径；
- 符号链接；

> What Is a Path?

在不同的操作系统中，文件系统的根目录也会存在差异，比如 windows 支持多个根节点，每个根节点使用盘符表示，比如 `C:\` 、`D:\`，而 Linux 或者 Solaris OS 支持单个根节点，即 `/`；

一个文件可以使用它在文件系统中的路径来标识，从根节点开始，比如说 Linux 中某个文件的路径：`/usr/local/share/file1`，换成 Windows 是这样的：`D:\share\file1.txt`，用于分隔目录名的字符（也成为分隔符）在不同的文件系统中都不一样：Linux 使用 `/`，Windows 使用 `\`。

> Relative or Absolute?

路径又分为两种：绝对路径和相对路径。

- 绝对路径中包含根元素和定位文件所需的完整目录列表，比如 `/usr/local/share/file1`；
- 相对路径需要与另一个路径组合才能访问文件，比如 `joe/foo` 就是一个相对路径，如果没有更多的信息，程序就无法根据该路径准确的定位文件。

> Symbolic Links

文件系统对象中最典型的是目录和文件，但是一些文件系统也支持符号链接的概念，符号链接也被称为软链接（在 Windows 系统中叫做快捷方式）。

符号链接是作为另一个文件引用的特殊文件，在大多数情况下，符号链接对应用程序是透明的，对符号链接的操作会自动重定向到该链接的目标（这个目标可以是文件或者目录）。

符号链接对用户来说也是透明的，像符号链接读取和写入与正常的文件或目录操作没什么区别。

而 `resolving a link` 意思是将符号链接替换为文件系统中实际的位置。

在实际场景中，大多数文件系统都可以随意使用符号链接，但是粗心创建的某些符号链接可能会导致循环引用，当链接的目标指向原始链接时，就会发生循环引用循环引用也可以是间接的，比如：目录 a 指向目录 b，目录 b 指向目录 c，目录 c 包含指向目录 a 的子目录，当程序递归遍历目录结构时，循环引用会造成巨大破坏，幸运的是，这种情况 API 中已经考虑到并处理过了。

## The Path Class

从 JDK 7 开始引入了 `java.nio.file.Path` 接口，它是 `java.nio.file` 包的一个主要的 entrypoint，能够为我们进行 File I/O 提供很多强大的功能；

顾名思义，Path 就是文件系统中某个路径在程序中的表示，Path对象包含用于构造路径的文件名和目录列表，并用于检查、定位和操作文件。

有一点需注意，当我们使用 Path 时，无需关心底层操作系统；

### Path Operations

Path 提供了很多方法用于获取 path 相关的信息、访问 path 中的元素、将 path 转换为其他格式的元素、或者提取 path 的一部分、还有匹配 path 路径字符串的方法与删除路径中冗余字符的方法。

#### 创建路径

Path 实例包含用于指定文件或目录位置的信息。

我们可以利用 `java.nio.file.Paths` 工具类的 `get` 方法来创建 Path 实例：

```java
Path p1 = Paths.get("/tmp/foo");
Path p2 = Paths.get(args[0]);
Path p3 = Paths.get(URI.create("file:///Users/joe/FileTest.java"));
```

URI 的创建方法参数可以参考源码中提供的格式，涉及到一些 RFC。

`Paths.get` 方法其实是下面代码的简写：

```java
Path p4 = FileSystems.getDefault().getPath("/users/sally");
```

假设用户的家目录是 `/home/nk` 或者 `C:\users\nk`，我们可以这样获取家目录下的某个文件，比如：`/home/nk/logs/foo.log`

```java
Path p5 = Paths.get(System.getProperty("user.home"), "logs", "foo.log");
```

#### 获取 Path 的信息

我们可以将 Path 看作某些元素 name 的序列，它们按照一定的顺序排列，拥有各自的下标，目录结构中最高级的元素下标是 0，最低级的元素下标是 n-1，n 就是 Path 中元素的 name 的总和，Path 中的方法可以使用这些索引下标来获取指定元素或子序列的信息。

```java
Path p6 = Paths.get(System.getProperty("user.dir"), "test.log");

System.out.printf("toString: %s%n", p6.toString());
System.out.printf("getFileName: %s%n", p6.getFileName());
System.out.printf("getName(0): %s%n", p6.getName(0));
System.out.printf("getNameCount: %d%n", p6.getNameCount());
System.out.printf("subpath(0, 2): %s%n", p6.subpath(0, 2));
System.out.printf("getParent: %s%n", p6.getParent());
System.out.printf("getRoot: %s%n", p6.getRoot()
```

Windows 下的输出：

```
toString: E:\IDEA Java SE Codeing\Java IO Learning\test.log
getFileName: test.log
getName(0): IDEA Java SE Codeing
getNameCount: 3
subpath(0, 2): IDEA Java SE Codeing\Java IO Learning
getParent: E:\IDEA Java SE Codeing\Java IO Learning
getRoot: E:\
```

注意上面的例子中使用的是绝对路径，也可以使用相对路径；

#### 移除 Path 中的冗余信息

一些文件系统使用 `.` 表示当前目录，使用 `..` 表示父目录，我们可能会遇到这样的情况: Path包含冗余目录信息，比如说某个服务的日志存储目录是这样的 `/dir/logs/.`，我们也会想删掉最后的 `.`：

下面两个例子都是含有冗余信息：

- `/home/./joe/foo`；
- `/home/sally/../joe/foo`；

使用 `java.nio.file.Path#normalize` 方法即可清理冗余的信息，但是需要注意该方法并不会检查文件系统，它只是一个纯粹的语法操作。

如果想要在清除路径的同时确保结果能够定位正确的文件，可以使用 `toRealPath` 方法。

#### 转换 Path

可以使用三种方法来转换 Path：

将 path 转换为 string 并且可以通过浏览器打开，可以这样：

```java
Path p6 = Paths.get(System.getProperty("user.dir"), "test.log");
System.out.printf("%s%n", p6.toUri());
```

`toAbsolutePath` 方法可以输出目标的完整路径，即使文件不存在也会输出：

```java
Path path = Paths.get(System.getProperty("user.dir"), "test.log");
System.out.println(path.toAbsolutePath());
```

`toRealPath` 方法要求文件必须是实际存在的，该方法会返回一个真实的 path：

```java
try {
    Path fp = path.toRealPath();
    System.out.println(fp);
} catch (IOException e) {
    e.printStackTrace();
}
```

该方法将一下三个操作合并为一个：

- 如果 toRealPath 接收了参数且文件系统支持符号链接，则该方法会解析符号链接；
- 如果 Path 是相对路径，该方法会返回绝对路径；
- 如果 Path 包含了冗余字符，该方法会移除冗余信息；

#### 合并两个 Path

使用 `resolve` 合并 Path。

```java
// Solaris
Path p1 = Paths.get("/home/joe/foo");
// Result is /home/joe/foo/bar
System.out.format("%s%n", p1.resolve("bar"));

or

// Microsoft Windows
Path p1 = Paths.get("C:\\home\\joe\\foo");
// Result is C:\home\joe\foo\bar
System.out.format("%s%n", p1.resolve("bar"));
```

#### 创建关联两个 Path 的 Path

在编写文件 I/O 的时候，一个常见的需求是构建一个 Path：从一个 Path 到另一个 Path，此时可以使用 `relativize` 方法，该方法从原始路径构造一个路径，并在传入路径指定的位置结束，新路径相是对于原路径的。

比如，我们有两个相对路径：

```java
Path p1 = Paths.get("joe");
Path p2 = Paths.get("sally");
```

在没有其他信息的情况下，假设 joe 和 sally 的相邻的，意味着它们在文件树结构中属于同一层次，从 joe 到 sally，我们可以从 joe 追溯到上层父节点，然后从父节点向下找到 sally，可以使用这样的代码：

```java
// Result is ../sally
Path p1_to_p2 = p1.relativize(p2);
// Result is ../joe
Path p2_to_p1 = p2.relativize(p1);
```

考虑一个复杂点的例子：

```java
Path p1 = Paths.get("home");
Path p3 = Paths.get("home/sally/bar");
// Result is sally/bar
Path p1_to_p3 = p1.relativize(p3);
// Result is ../..
Path p3_to_p1 = p3.relativize(p1);
```

这个例子中，两个 Path 共享一个节点：home。

#### 比较两个 Path

Path 类型的实例也支持 `equals()` 方法，允许您测试两条路径是否相等，`startsWith` 和 `endsWith` 可以检测路径是否以特定字符串开始或者结束，比如下面这样：

```java
Path path = ...;
Path otherPath = ...;
Path beginning = Paths.get("/home");
Path ending = Paths.get("foo");

if (path.equals(otherPath)) {
    // equality logic here
} else if (path.startsWith(beginning)) {
    // path begins with "/home"
} else if (path.endsWith(ending)) {
    // path ends with "foo"
}
```

Path 接口实现了 `java.lang.Iterable`，意味着 path 是可以遍历的，开发者可以遍历 path 中所有 name 元素，返回的第一个元素是在目录树中最接近根的元素。下面的代码段遍历一个路径，打印每个name元素:

```java
Path path = ...;
for (Path name: path) {
    System.out.println(name);
}
```

Path 也实现了 `java.lang.Comparable` 接口，意味着我们可以对 Path 实例进行排序操作；

当您想验证两个Path对象是否位于同一个文件时，可以使用 `isSameFile` 方法。

## File Operations

`java.nio.file` 包下面的另一个重要 entrypoint 就是 `java.nio.file.Files` 类，它提供了丰富的静态方法用于读取、写入以及操作 files 和 directories。同时 Files 类的方法作用于 Path 的实例。

首先我们需要了解以下知识：

- Releasing System Resources（释放操作系统资源）；
- Catching Exceptions（异常捕捉）；
- Varargs（可变参数）；
- Atomic Operations（原子操作）；
- Method Chaining（链式调用）；
- What is a Glob?（通配符语法）；
- Link Awareness（处理符号链接）；

### Releasing System Resources

Files 提供的 API 中使用了很多系统资源，比如：streams 和 channels，对于实现或者继承了 `java.io.Closeable` 接口的资源，一旦确定不再使用它，就需要立即调用 Closeable 接口的 `close()` 方法及时释放系统资源，如果不这样做会对系统性能造成一定的影响（可以调用 close 方法，也可以使用 try-with-resources 语法）。

### Catching Exceptions

使用 file I/O，我们要考虑到各种可能的突发情况：指定的文件是否存在，应用程序有没有对文件系统的访问权限，默认的文件系统实现是否支持特定的 function，等等。可能会遇到很多错误。

所以只要涉及到对文件系统的访问的方法都会声明抛出 IOException，通过 Java SE 7 提供的 trye-with-resources 语法可以很好的处理这些异常并及时释放资源。比如下面的示例代码：

```java
Charset charset = Charset.forName("US-ASCII");
String s = ...;
try (BufferedWriter writer = Files.newBufferedWriter(file, charset)) {
    writer.write(s, 0, s.length());
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
```

更多信息，参考 [The try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)。

也可以这样做，在 finally 块中调用 close 资源释放流或者通道。

```java
Charset charset = Charset.forName("US-ASCII");
String s = ...;
BufferedWriter writer = null;
try {
    writer = Files.newBufferedWriter(file, charset);
    writer.write(s, 0, s.length());
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
} finally {
    if (writer != null) writer.close();
}
```

更多信息参考：[Catching and Handling Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/handling.html)。

注意 IOException 是编译器异常，除此之外对于其他继承自 `java.nio.file.FileSystemException` 类的异常，FileSystemException 类也提供了一些有用的方法，比如可以获取到相关的文件 （getFile），获取提示信息（getMessage），获取文件系统操作失败的原因（getReason），以及可能存在的其他文件（getOtherFile）。代码如下：

```java
try (...) {
    ...    
} catch (NoSuchFileException x) {
    System.err.format("%s does not exist\n", x.getFile());
}
```

为了方便，后面的代码中省略了异常处理，但要记得实际编程时一定要异常捕获和资源释放相关操作。

### Varargs

Files 类提供的某些方法在需要声明 flag 时会接收一些任意数量的参数（可变参数），比如下面的方法声明：

```java
Path Files.move(Path, Path, CopyOption...)
```

关于可变参数的更多信息，参考：[Arbitrary Numbers of Arguments](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html#varargs)。

### Atomic Operations

Files 中的某些方法，比如 move 可以在某些文件系统中原子地执行某些操作。

原子文件操作是一种不能中断或“部分”执行的操作，要么一次成功要么就失败。当有多个进程在文件系统的同一区域上运行时，这一点非常重要，您需要保证每个进程访问一个完整的文件。

### Method Chaining

File I/O 涉及到的很多方法都支持 method chaining 这个概念。

你调用的第一个方法返回一个对象，可以立即利用这个对象继续调用方法（这个方法也会返回一个对象），这样就可以形成链式调用，这种技巧就像下面这样：

```java
String value = Charset.defaultCharset().decode(buf).toString();
UserPrincipal group =
    file.getFileSystem().getUserPrincipalLookupService().
         lookupPrincipalByName("me");
```

这种技术产生紧凑的代码，使您可以避免声明不需要的临时变量。

### What is a Glob?

Files 类中有两个方法接收 glob 参数，那么什么是 glob 呢？

我们可以使用 glob 语法去声明 pattern-matching 行为。

一个通配符模式被声明为一个字符串，并与其他字符串进行匹配，比如目录或文件的名称，通配符语法类似下面这样：

- `*`：匹配任意数量的字符（包括空字符）；
- `**`：有点类似一个星号，但是它可以跨越目录的边界，此语法常用于匹配整条 path；
- `?`：只匹配一个字符；
- 花括号指定子模式的集合，比如：
  - `{sun,moon,stars}` 匹配 "sun"、"moon" 或者 "stars"；
  - `{temp*,tmp*}`，匹配以 temp 或者 tmp 开头的字符串；
- 方括号表示一组单个字符，如果使用连字符 `-`，则表示一组字符：
  - `[aeiou]` 匹配任意小写元音字母；
  - `[0-9]`：匹配任意数字；
  - `[A-Z]`：匹配任意大写字母；
  - `[a-z,A-Z]`：匹配任意小写或者大写字母；
  - 注意：在方括号内使用 `*、?、\` 时，不具有特殊意义，只会匹配自身；
- 所有其他字符都与自己匹配；
- 如果要匹配 `*、?` 或者其他特殊字符，可以在前面加上反斜杠 `\` 表示转义，比如 `\\` 匹配单个反斜线，`\?` 匹配单个 ? 号。

下面是一些例子：

- `*.html`：匹配所有以 .html 为后缀的字符串；
- `???`：匹配任何只有三个字符的包含字母或数字的字符串；
- `*[0-9]*`：匹配任意包含一个数值的字符串；
- `*.{htm,html,pdf}`：匹配以 .htm、.html、.pdf 为后缀的字符串；
- `a?*.java`：匹配任何以 a 开头，后面至少一个字母或数字，以 .java 结尾的字符串；
- `{foo*,*[0-9]*}`：匹配任何以 foo 开头的字符串或任何包含数值的字符串

注意：如果在键盘上输入 glob 模式，并且它包含一个特殊字符，则必须将模式放在引号（比如 `"*"`）中，使用反斜杠（比如 `\`），或使用命令行中支持的任何转义机制。

通配符语法功能强大且易于使用，但是，如果它不能满足您的需要，您也可以使用[正则表达式](https://docs.oracle.com/javase/tutorial/essential/regex/index.html)。

更多通配符语法信息，可以参考 API 文档中的  `java.nio.file.FileSystem#getPathMatcher` 方法。

### Link Awareness

Files 类是 `"link aware"` 的，它提供的方法遇到符号链接时会采取默认的处理方式，当然您也可以传入相关参数来决定如何处理。

## Checking a File or Directory







进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html
- https://docs.oracle.com/javase/tutorial/essential/io/index.html
- https://docs.oracle.com/javase/tutorial/essential/io/check.html

