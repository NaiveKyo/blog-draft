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

当我们获取到一个 Path 示例，它可能代表一个文件或者目录，且该 Path 是否真的存在于文件系统中吗？是可读、可写还是可执行的？

> 检验文件或目录的存在性

我们可以通过 Files 类中的方法来检验一个 Path 示例是否在文件系统中存在：

```java
Path path = Paths.get(System.getProperty("user.dir") + File.separator + "test.log");

// 检测文件是否存在
System.out.println(Files.exists(path));
System.out.println(Files.notExists(path));
```

- `java.nio.file.Files#exists`；
- `java.nio.file.Files#notExists`；

但是需要注意 `!Files.exists(path)` 不等于 `Files.notExists(path)`，当我们在校验文件的存在性时，有以下三种可能的结果：

（1）文件经校验后确认是存在的；

（2）文件经校验后确认是不存在的；

（3）不能确定文件的状态，出现此种情况一般是程序没有该文件的访问权限导致的；

如果 exists 和 notExists 方法都返回 false，说明这个文件的存在性是无法被验证的。

> 检验是否拥有对文件的访问权限

使用以下方法检验程序对文件的访问权：

- `java.nio.file.Files#isReadable`；
- `java.nio.file.Files#isWritable`；
- `java.nio.file.Files#isExecutable`；

下面的代码示例展示了如何获取指定文件存在且程序拥有对它的执行权限：

```java
Path file = ...;
boolean isRegularExecutableFile = Files.isRegularFile(file) &
         Files.isReadable(file) & Files.isExecutable(file);
```

> 校验两个 Path 定位了同一个文件

当我们使用的文件系统允许使用符号链接的时候，会出现两个 Path 示例指向同一个文件的情况，此时可以使用 `java.nio.file.Files#isSameFile` 方法来检验它们是不是指向了同一个文件：

```java
Path p1 = ...;
Path p2 = ...;

if (Files.isSameFile(p1, p2)) {
    // Logic when the paths locate the same file
}
```

## Deleting a File or Directory

你可以删除文件、目录或链接。对于符号链接而言，删除的是这个链接而不是被链接的文件，对于目录而言，这个目录必须是空目录，否则就会删除失败。

Files 类提供了两个删除相关的方法：

- `java.nio.file.Files#delete`；
- `java.nio.file.Files#deleteIfExists`；

第一个方法在删除文件时如果操作失败就会抛出异常，比如说：文件不存在会抛出 NoSuchFileException：

```java
try {
    Files.delete(path);
} catch (NoSuchFileException x) {
    System.err.format("%s: no such" + " file or directory%n", path);
} catch (DirectoryNotEmptyException x) {
    System.err.format("%s not empty%n", path);
} catch (IOException x) {
    // File permission problems are caught here.
    System.err.println(x);
}
```

第二个方法也可以删除文件，但是在文件不存在时不会抛出异常，当您有多个线程删除文件，并且您不想仅仅因为一个线程先删除文件而抛出异常时，静默失败是有用的。

## Copying a File or Directory

可以使用 `java.nio.file.Files#copy` 方法来复制文件或目录，如果目标文件存在则复制失败，除非使用 `REPLACE_EXISTING` 选项来覆盖目标文件。

目录也可以复制。但是，目录内的文件不会被复制，因此即使原始目录中有文件，新目录也是空的。

当复制链接文件时，真正被复制的其实是链接的目标文件，如果只想复制链接文件本身，而不复制被链接的文件，则可以使用 `NOFOLLOW_LINKS` 或者 `REPLACE_EXISTING` 选项。

copy 方法接收一个可变参数 `java.nio.file.CopyOption`，经常使用的是该接口的两个实现 enum：`java.nio.file.StandardCopyOption` 和 `java.nio.file.LinkOption`。

- REPLACE_EXISTING：即使目标文件已经存在，也要执行复制；如果目标是符号链接，则复制链接本身(而不是链接的目标)；如果目标是非空目录，则复制失败，并出现DirectoryNotEmptyException异常；
- COPY_ATTRIBUTES：将与该文件关联的文件属性复制到目标文件。具体支持的文件属性取决于文件系统和平台，但 `last-modified-time` 跨平台支持，并被复制到目标文件；
- NOFOLLOW_LINKS：指示不应遵循符号链接。如果要复制的文件是符号链接，则复制该链接(而不是该链接的目标)。

枚举类的信息参考：[Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)

```jav
import static java.nio.file.StandardCopyOption.*;
...
Files.copy(source, target, REPLACE_EXISTING);
```

除了文件复制，Files 类还定义了可用于在文件和流之间进行复制的方法。`copy(InputStream, Path, CopyOptions…)` 方法可用于将输入流中的所有字节复制到文件中。`copy(Path, OutputStream)` 方法可用于将文件中的所有字节复制到输出流。

下面的例子使用了 copy 和 Files.walkFileTree 方法来递归复制。

```java
import java.nio.file.*;
import static java.nio.file.StandardCopyOption.*;
import java.nio.file.attribute.*;
import static java.nio.file.FileVisitResult.*;
import java.io.IOException;
import java.util.*;

/**
 * Sample code that copies files in a similar manner to the cp(1) program.
 */

public class Copy {

    /**
     * Returns {@code true} if okay to overwrite a  file ("cp -i")
     */
    static boolean okayToOverwrite(Path file) {
        String answer = System.console().readLine("overwrite %s (yes/no)? ", file);
        return (answer.equalsIgnoreCase("y") || answer.equalsIgnoreCase("yes"));
    }

    /**
     * Copy source file to target location. If {@code prompt} is true then
     * prompt user to overwrite target if it exists. The {@code preserve}
     * parameter determines if file attributes should be copied/preserved.
     */
    static void copyFile(Path source, Path target, boolean prompt, boolean preserve) {
        CopyOption[] options = (preserve) ?
            new CopyOption[] { COPY_ATTRIBUTES, REPLACE_EXISTING } :
            new CopyOption[] { REPLACE_EXISTING };
        if (!prompt || Files.notExists(target) || okayToOverwrite(target)) {
            try {
                Files.copy(source, target, options);
            } catch (IOException x) {
                System.err.format("Unable to copy: %s: %s%n", source, x);
            }
        }
    }

    /**
     * A {@code FileVisitor} that copies a file-tree ("cp -r")
     */
    static class TreeCopier implements FileVisitor<Path> {
        private final Path source;
        private final Path target;
        private final boolean prompt;
        private final boolean preserve;

        TreeCopier(Path source, Path target, boolean prompt, boolean preserve) {
            this.source = source;
            this.target = target;
            this.prompt = prompt;
            this.preserve = preserve;
        }

        @Override
        public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
            // before visiting entries in a directory we copy the directory
            // (okay if directory already exists).
            CopyOption[] options = (preserve) ?
                new CopyOption[] { COPY_ATTRIBUTES } : new CopyOption[0];

            Path newdir = target.resolve(source.relativize(dir));
            try {
                Files.copy(dir, newdir, options);
            } catch (FileAlreadyExistsException x) {
                // ignore
            } catch (IOException x) {
                System.err.format("Unable to create: %s: %s%n", newdir, x);
                return SKIP_SUBTREE;
            }
            return CONTINUE;
        }

        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
            copyFile(file, target.resolve(source.relativize(file)),
                     prompt, preserve);
            return CONTINUE;
        }

        @Override
        public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
            // fix up modification time of directory when done
            if (exc == null && preserve) {
                Path newdir = target.resolve(source.relativize(dir));
                try {
                    FileTime time = Files.getLastModifiedTime(dir);
                    Files.setLastModifiedTime(newdir, time);
                } catch (IOException x) {
                    System.err.format("Unable to copy all attributes to: %s: %s%n", newdir, x);
                }
            }
            return CONTINUE;
        }

        @Override
        public FileVisitResult visitFileFailed(Path file, IOException exc) {
            if (exc instanceof FileSystemLoopException) {
                System.err.println("cycle detected: " + file);
            } else {
                System.err.format("Unable to copy: %s: %s%n", file, exc);
            }
            return CONTINUE;
        }
    }

    static void usage() {
        System.err.println("java Copy [-ip] source... target");
        System.err.println("java Copy -r [-ip] source-dir... target");
        System.exit(-1);
    }

    public static void main(String[] args) throws IOException {
        boolean recursive = false;
        boolean prompt = false;
        boolean preserve = false;

        // process options
        int argi = 0;
        while (argi < args.length) {
            String arg = args[argi];
            if (!arg.startsWith("-"))
                break;
            if (arg.length() < 2)
                usage();
            for (int i=1; i<arg.length(); i++) {
                char c = arg.charAt(i);
                switch (c) {
                    case 'r' : recursive = true; break;
                    case 'i' : prompt = true; break;
                    case 'p' : preserve = true; break;
                    default : usage();
                }
            }
            argi++;
        }

        // remaining arguments are the source files(s) and the target location
        int remaining = args.length - argi;
        if (remaining < 2)
            usage();
        Path[] source = new Path[remaining-1];
        int i=0;
        while (remaining > 1) {
            source[i++] = Paths.get(args[argi++]);
            remaining--;
        }
        Path target = Paths.get(args[argi]);

        // check if target is a directory
        boolean isDir = Files.isDirectory(target);

        // copy each source file/directory to target
        for (i=0; i<source.length; i++) {
            Path dest = (isDir) ? target.resolve(source[i].getFileName()) : target;

            if (recursive) {
                // follow links when copying files
                EnumSet<FileVisitOption> opts = EnumSet.of(FileVisitOption.FOLLOW_LINKS);
                TreeCopier tc = new TreeCopier(source[i], dest, prompt, preserve);
                Files.walkFileTree(source[i], opts, Integer.MAX_VALUE, tc);
            } else {
                // not recursive so source must not be a directory
                if (Files.isDirectory(source[i])) {
                    System.err.format("%s: is a directory%n", source[i]);
                    continue;
                }
                copyFile(source[i], dest, prompt, preserve);
            }
        }
    }
}
```

## Moving a File or Directory

使用 `java.nio.file.Files#move` 方法，如果目标文件存在，移动将失败，除非指定了 REPLACE_EXISTING 选项（进行覆盖操作）。

可以移动空目录。如果目录不为空，则允许在不移动目录内容的情况下移动目录。在 UNIX 系统上，在同一个分区内移动目录通常包括重命名目录，在这种情况下，即使目录中包含文件，该方法也可以工作。

这个方法也接收一个 CopyOption 的可选参数，但是它只能使用 `java.nio.file.StandardCopyOption`：

- REPLACE_EXISTING：即使目标文件已经存在，也要执行移动。如果目标是符号链接，则符号链接将被替换，但它所指向的内容不受影响；
- ATOMIC_MOVE：将移动作为原子文件操作执行。如果文件系统不支持原子移动，则抛出异常。使用 ATOMIC_MOVE，您可以将一个文件移动到一个目录中，并保证监视该目录的任何进程都可以访问一个完整的文件。

```java
import static java.nio.file.StandardCopyOption.*;
...
Files.move(source, target, REPLACE_EXISTING);
```

如上述代码所示，尽管您可以在单个目录上实现 move 方法，但该方法最常与文件树递归机制一起使用。

## Managing Metadata

元数据包括：File、File Store Attributes；

它的定义是：和数据有关的数据；

对于文件系统，数据包含在它的文件和目录中，元数据跟踪关于每个对象的信息：它是常规文件、目录还是链接？它的大小、创建日期、最后修改日期、文件所有者、组所有者和访问权限是什么？

文件系统的元数据通常称为其文件属性。Files 类包含可用于获取文件的单个属性或设置属性的方法。

| Methods                                                      | Comment                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`size(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#size-java.nio.file.Path-) | 返回指定文件的字节数；                                       |
| [`isDirectory(Path, LinkOption)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isDirectory-java.nio.file.Path-java.nio.file.LinkOption...-) | 如果 Path 实例在文件系统中对应的文件是一个目录，则返回 true； |
| [`isRegularFile(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isRegularFile-java.nio.file.Path-java.nio.file.LinkOption...-) | 如果 Path 实例在文件系统中对应的是一个常规文件，则返回 true； |
| [`isSymbolicLink(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isSymbolicLink-java.nio.file.Path-) | 如果 Path 实例在文件系统中对应的是一个符号链接，则返回 true； |
| [`isHidden(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isHidden-java.nio.file.Path-) | 如果 Path 实例在文件系统中对应的是一个隐藏文件，则返回 true； |
| [`getLastModifiedTime(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getLastModifiedTime-java.nio.file.Path-java.nio.file.LinkOption...-) [`setLastModifiedTime(Path, FileTime)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setLastModifiedTime-java.nio.file.Path-java.nio.file.attribute.FileTime-) | 返回或者设置指定 Path 文件的最后修改时间；                   |
| [`getOwner(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getOwner-java.nio.file.Path-java.nio.file.LinkOption...-) [`setOwner(Path, UserPrincipal)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setOwner-java.nio.file.Path-java.nio.file.attribute.UserPrincipal-) | 获取或者设置指定 Path 文件的拥有者；                         |
| [`getPosixFilePermissions(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getPosixFilePermissions-java.nio.file.Path-java.nio.file.LinkOption...-) [`setPosixFilePermissions(Path, Set)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setPosixFilePermissions-java.nio.file.Path-java.util.Set-) | Returns or sets a file's POSIX file permissions.             |
| [`getAttribute(Path, String, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getAttribute-java.nio.file.Path-java.lang.String-java.nio.file.LinkOption...-) [`setAttribute(Path, String, Object, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setAttribute-java.nio.file.Path-java.lang.String-java.lang.Object-java.nio.file.LinkOption...-) | Returns or sets the value of a file attribute.               |

如果一个程序同时需要多个文件属性，使用检索单个属性的方法可能效率很低。重复访问文件系统以检索单个属性可能会对性能产生不利影响。Files类提供了两个 readAttributes 方法，用于在一次批量操作中获取文件的属性。

| Method                                                       | Comment                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`readAttributes(Path, String, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#readAttributes-java.nio.file.Path-java.lang.String-java.nio.file.LinkOption...-) | Reads a file's attributes as a bulk operation. The `String` parameter identifies the attributes to be read. |
| [`readAttributes(Path, Class, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#readAttributes-java.nio.file.Path-java.lang.Class-java.nio.file.LinkOption...-) | Reads a file's attributes as a bulk operation. The `Class<A>` parameter is the type of attributes requested and the method returns an object of that class. |

在展示 readAttributes 方法的示例之前，我们需要注意一点：不同的文件系统对于应该跟踪哪些属性有不同的概念。出于这个原因，相关的文件属性被分组到 `view` 中。一个 `view`  映射到特定的文件系统实现，比如 POSIX 或 DOS，或者一个常见的功能，比如 file ownership。

支持以下 views：

- `java.nio.file.attribute.BasicFileAttributeView`：
  - 提供所有文件系统实现必须支持的基本属性的视图。
- `java.nio.file.attribute.DosFileAttributeView`：
  - 将基本属性视图扩展到支持 DOS 属性的文件系统上的 the standard four bits。
- `java.nio.file.attribute.PosixFileAttributeView`：
  - 将基本属性视图扩展到支持标准的 POSIX 家族 (如 UNIX ) 的文件系统上的属性。这些属性包括文件所有者、组所有者和 9 个相关的访问权限。
- `java.nio.file.attribute.FileOwnerAttributeView`：
  - 任何支持文件所有者概念的文件系统实现都支持。
- `java.nio.file.attribute.AclFileAttributeView`：
  - 支持读取或更新文件的访问控制列表 (ACL)。支持 NFSv4 ACL 模型。还可能支持与 NFSv4 模型有良好定义映射的任何 ACL 模型(例如Windows ACL模型)。 
- `java.nio.file.attribute.UserDefinedFileAttributeView`：
  - 启用对用户定义元数据的支持。该视图可以映射到系统支持的任何扩展机制。例如，在 Solaris 操作系统中，您可以使用这个视图存储文件的 MIME 类型。

特定的文件系统实现可能只支持基本的文件属性视图，也可能支持这些文件属性视图中的几个。文件系统实现也可能支持此 API 中未包含的其他属性视图。

在大多数情况下，您不应该直接处理任何 `FileAttributeView` 接口。（f you do need to work directly with the `FileAttributeView`, you can access it via the [`getFileAttributeView(Path, Class, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getFileAttributeView-java.nio.file.Path-java.lang.Class-java.nio.file.LinkOption...-) method.）

readAttributes 方法使用泛型，可用于读取任何文件属性视图的属性。

下面介绍的内容包括以下几个主题：

- Basic File Attributes：基础文件属性；
- Setting Time Stamps：设置时间戳；
- DOS File Attributes：DOS 文件属性；
- POSIX File Permissions：可移植操作系统文件权限；
- Setting a File or Group Owner：设置文件或组的所有者；
- User-Defined File Attributes：用户自定义文件属性；
- File Store Attributes：文件存储属性。

### 基础文件属性

如前所述，要读取文件的基本属性，可以使用 Files.readAttributes 方法，它在一个批量操作中读取所有基本属性。这比分别访问文件系统来读取每个单独的属性要高效得多。varargs 参数目前支持 LinkOption enum, NOFOLLOW_LINKS。当您不希望使用符号链接时，请使用此选项。

关于时间戳：基础属性中包含三种时间戳：creationTime、lastModifiedTime、lastAccessTime。这些时间戳中的任何一个在特定的实现中都可能不受支持，在这种情况下，相应的访问器方法返回一个特定于实现的值。在支持的情况下，时间戳将作为 `FileTime` 对象返回。

下面的代码展示了读取指定文件的基础文件属性，通过 `java.nio.file.attribute.BasicFileAttributes` 接口获取：

```java
Path file = ...;
BasicFileAttributes attr = Files.readAttributes(file, BasicFileAttributes.class);

System.out.println("creationTime: " + attr.creationTime());
System.out.println("lastAccessTime: " + attr.lastAccessTime());
System.out.println("lastModifiedTime: " + attr.lastModifiedTime());

System.out.println("isDirectory: " + attr.isDirectory());
System.out.println("isOther: " + attr.isOther());
System.out.println("isRegularFile: " + attr.isRegularFile());
System.out.println("isSymbolicLink: " + attr.isSymbolicLink());
System.out.println("size: " + attr.size());
```

除了本例中显示的法外，还有一个 fileKey 方法，该方法返回唯一标识文件的对象，如果没有可用的文件键，则返回 null。

### 设置时间戳

比如说修改 last modified time：

```java
Path file = ...;
BasicFileAttributes attr =
    Files.readAttributes(file, BasicFileAttributes.class);
long currentTime = System.currentTimeMillis();
FileTime ft = FileTime.fromMillis(currentTime);
Files.setLastModifiedTime(file, ft);
```

### DOS 文件属性

DOS 文件属性在其他非 DOS 系统上也受支持，比如 Samba。下面的代码演示使用 DosFileAttributes：

```java
Path file = ...;
try {
    DosFileAttributes attr =
        Files.readAttributes(file, DosFileAttributes.class);
    System.out.println("isReadOnly is " + attr.isReadOnly());
    System.out.println("isHidden is " + attr.isHidden());
    System.out.println("isArchive is " + attr.isArchive());
    System.out.println("isSystem is " + attr.isSystem());
} catch (UnsupportedOperationException x) {
    System.err.println("DOS file" +
        " attributes not supported:" + x);
}
```

但是，也可以使用 `setAttributes(Path, String, Object, LinkOption...)` 方法：

```java
Path file = ...;
Files.setAttribute(file, "dos:hidden", true);
```

### POSIX 文件权限

POSIX 是 Portable Operating System Interface for UNIX 的缩写，是一组 IEEE 和 ISO 标准，旨在确保不同 UNIX 风格之间的互操作性。如果一个程序符合这些 POSIX 标准，它应该很容易移植到其他 POSIX 兼容的操作系统上。

除了 file owner 和 group owner，POSIX 还支持9种文件权限：read, write, and execute permissions for the file owner, members of the same group, and "everyone else."

下面演示使用 PosixFileAttributes ：

```java
Path file = ...;
PosixFileAttributes attr =
    Files.readAttributes(file, PosixFileAttributes.class);
System.out.format("%s %s %s%n",
    attr.owner().getName(),
    attr.group().getName(),
    PosixFilePermissions.toString(attr.permissions()));
```

`java.nio.file.attribute.PosixFilePermissions` 工具类提供了一些好用的方法：

- `toString` 方法：用字符串描述权限，比如 `rw-r--r--`；
- `fromString` 方法：接收一个权限字符串将其转换为一组 PosixFilePermission 集合；
- `asFileAttribute` 方法：将一个文件权限的 Set 构建为一个文件属性，可以传递给 `Path.createFile` 或 `Path.createDirectory` 方法。

下面的代码片段从一个文件中读取属性并创建一个新文件，将原始文件中的属性分配给新文件：

```java
Path sourceFile = ...;
Path newFile = ...;
PosixFileAttributes attrs =
    Files.readAttributes(sourceFile, PosixFileAttributes.class);
FileAttribute<Set<PosixFilePermission>> attr =
    PosixFilePermissions.asFileAttribute(attrs.permissions());
Files.createFile(file, attr);
```

asFileAttribute 方法将一组 permission 包装为一个 FileAttribute，上述代码尝试使用这个文件属性去创建新的文件，注意 umask 也适用新创建的文件，这就意味着通过这种方式创建的文件是安全的。

要将文件的权限设置为硬编码字符串表示的值，您可以使用以下代码:

```java
Path file = ...;
Set<PosixFilePermission> perms =
    PosixFilePermissions.fromString("rw-------");
FileAttribute<Set<PosixFilePermission>> attr =
    PosixFilePermissions.asFileAttribute(perms);
Files.setPosixFilePermissions(file, perms);
```

[Chmod](https://docs.oracle.com/javase/tutorial/essential/io/examples/Chmod.java) 例子的功能和 chmod 命令类似，可以递归地修改文件权限。

### 设置 File or Group Owner

如果想根据字符串找到对应的 file owner 或 group owner，可以使用 `java.nio.file.attribute.UserPrincipalLookupService` 服务，该服务接收一个字符串（user name 或者 group name），并返回一个 `UserPrincipal` 实例。可以使用 `FileSystems.getDefault().getUserPrincipalLookupService();` 获取默认文件系统的 user principal look-up service。

下面展示如何设置文件所有者：

```java
Path file = ...;
UserPrincipal owner = file.GetFileSystem().getUserPrincipalLookupService()
        .lookupPrincipalByName("sally");
Files.setOwner(file, owner);
```

在 Files 类中没有专门用于设置 group owner 的方法，但是一个更好更安全的方式是通过 POSIX 文件属性 view 去做：

```java
Path file = ...;
GroupPrincipal group =
    file.getFileSystem().getUserPrincipalLookupService()
        .lookupPrincipalByGroupName("green");
Files.getFileAttributeView(file, PosixFileAttributeView.class)
     .setGroup(group);
```

### 用户自定义文件属性

如果文件系统提供的 file attributes 不能够满足需求，我们就可以使用 `java.nio.file.attribute.UserDefinedFileAttributeView` 创建和跟踪自己的文件属性。

一些实现将此概念映射到特殊文件系统的某些特性上，比如 NTFS 的可选数据流（Alternative Data Streams），ext3 和 XFS 的扩展属性。大多数实现对这些值的大小施加限制，例如，ext3 将扩展属性大小限制为 4KB。

也可以利用文件的 MIME type 来存储用户自定义属性：

```java
Path file = ...;
UserDefinedFileAttributeView view = Files
    .getFileAttributeView(file, UserDefinedFileAttributeView.class);
view.write("user.mimetype",
           Charset.defaultCharset().encode("text/html");
```

使用下列代码读取 MIME type：

```java
Path file = ...;
UserDefinedFileAttributeView view = Files
.getFileAttributeView(file,UserDefinedFileAttributeView.class);
String name = "user.mimetype";
ByteBuffer buf = ByteBuffer.allocate(view.size(name));
view.read(name, buf);
buf.flip();
String value = Charset.defaultCharset().decode(buf).toString();
```

[Xdd](https://docs.oracle.com/javase/tutorial/essential/io/examples/Xdd.java) 代码示例展示了如何 get、set、delete 用户自定义属性。

注意：在 Linux 中，您可能必须为用户定义的属性启用扩展属性才能工作。如果在尝试访问用户定义属性视图时收到 UnsupportedOperationException 异常，则需要重新挂载文件系统。下面的命令为 ext3 文件系统重新挂载具有扩展属性的根分区：

```shell
$ sudo mount -o remount,user_xattr /
```

如果上述命令不适用当前使用的 Linux 系统，请参考相关文档。注意上述命令不是永久性的，要想永久生效，需要添加条目：`/etc/fstab`

### File Store Attributes

可以使用 `java.nio.file.FileStore` 学习关于 file store 的更多信息，比如还有多少空间是可用的，`getFileStore(Path)` 方法获取指定文件的存储空间。

下面的代码演示如何获取指定文件的占用存储空间大小：

```java
Path file = ...;
FileStore store = Files.getFileStore(file);

long total = store.getTotalSpace() / 1024;
long used = (store.getTotalSpace() -
             store.getUnallocatedSpace()) / 1024;
long avail = store.getUsableSpace() / 1024;
```

[DiskUsage](https://docs.oracle.com/javase/tutorial/essential/io/examples/DiskUsage.java) 示例使用这个 API 打印默认文件系统中所有存储的磁盘空间信息。本例使用 FileSystem 类中的 getFileStores 方法获取文件系统的所有文件存储。

## Reading, Writing, and Creating Files

本小节讨论读取、写入、创建和打开文件。有许多文件相关的 I/O API 方法可以使用。

下表按照复杂程度由低到高列出一些常用 API：

| 方法                                        | 使用场景                                            | 复杂程度（低 -> 高） |
| ------------------------------------------- | --------------------------------------------------- | -------------------- |
| `readAllBytes`<br/>`readAllLines`           | 经常使用的方法，适用小文件                          |                      |
| `newBufferedReader`<br/>`newBufferedWriter` | 适合文本文件（txt 后缀）                            |                      |
| `newInputStream`<br/>`newOutputStream`      | 流、不使用缓存的，通常和已有的 API 一起使用         |                      |
| `newByteChannel`                            | channels 和 ByteBuffers（NIO 中的管道和堆外缓冲区） |                      |
| `FileChannel`                               | 增强特性，file-locking，memory-mapped I/O           |                      |

上表前几行的是一些工具方法，如 `readAllBytes`、`readAllLines`，以及 write 方法，适用于常用的简单的场景；上表中间几行的则是用于遍历文本流或文本行，如 `newBufferedReader`、`newBufferedWriter`；然后就是 `newInputStream` 和 `newOutputStream`；这几个方法可以与 `java.io` 包下的工具协同使用。

最后几行的则是用于同 `ByteChannels`、`SeekableByteChannels` 以及 `ByteBuffers` 一起使用，比如 `newByteChannel` 方法。

最后面是 `FileChannel` 它主要用于更高级的应用场景，比如文件锁（file locking）或者内存映射 I/O（memory-mapped I/O）。

> 注意：创建新的文件允许我们使用一组初始属性。比如在支持 POSIX 标准（如 UNIX）的文件系统上，我们可以在创建文件时声明 file owner、group owner 以及其他权限，上一章节关于文件元数据管理中已经提到了如何设置和访问文件。

本节则介绍以下知识：

-  `OpenOptions` 参数；
- 小文件常用的方法；
- 文本文件的 Buffered I/O；
- Methods for Unbuffered Streams and Interoperable with `java.io` APIs；
- Channels 和 `ByteBuffers` 相关的方法；
- 创建普通和临时文件的方法；

### OpenOptions 参数

本小节介绍的不少方法都会接收一个可选的 OpenOptions 类型的参数，如果未提供该参数则采取默认处理策略。

`java.nio.file.OpenOption` 有几个实现类，其中 `java.nio.file.StandardOpenOption` 是标准实现，它提供几个常量：

- `WRITE`：以 **写** 的形式访问该文件；
- `APPEND`：向文件末尾追加数据，它可以和 WRITE、CREATE 一起使用；
- `TRUNCATE_EXISTING`：将文件截断为 0 bytes，它和 WRITE 一起使用；
- `CREATE_NEW`：创建一个新文件，当该文件已存在时抛出异常；
- `CREATE`：文件存在则打开该文件，文件不存在则创建新文件；
- `DELETE_ON_CLOSE`：在流关闭时删除该文件，这个属性非常适合临时文件；
- `SPARSE`：暗示将要创建的文件是 sparse （稀疏）的。某些操作系统支持这个增强选项，比如 NTFS，如果某些大文件存在 data "gaps" （数据空白）的情况，文件系统能够以更好的方式存储它，避免造成磁盘空间的浪费；
- `SYNC`：保持文件（包括内容和元数据）与底层存储设备同步；
- `DSYNC`：仅保持文件内容和底层存储设备同步；



### 小文件常用的方法

（1）从文件中读取所有字节或所有行；

如果您有一个较小的文件，并且希望一次性读取其全部内容，此时可以使用 `readAllBytes(Path)` 或者 `readAllLines(Path, Charset)` 方法。这些方法为您处理大部分工作，例如打开和关闭流，但不适用于处理大型文件。

```java
Path file = ...;
byte[] fileArray;
fileArray = Files.readAllBytes(file);
```

（2）向一个文件写入所有字节或行；

可以使用其中一种写入方法将字节或行写入文件。

- `write(Path, byte[], OpenOption...)`；
- `write(Path, Iterable< extends CharSequence>, Charset, OpenOption...)`；

```java
Path file = ...;
byte[] buf = ...;
Files.write(file, buf);
```

### 文本文件的缓冲 I/O 方法

`java.nio.file` 包支持 channel I/O，它可以在缓冲区中移动数据，绕过一些可能会阻碍 stream I/O的层。

（1）使用 Buffered Stream I/O 读取文件；

`newBufferedReader(Path, Charset)` 方法用于打开文件并进行读取，返回一个 `BufferedReader` 用于更高效的读取文本信息； 

下面展示如何使用 `newBufferedReader` 方法读取文件：

```java
Charset charset = Charset.forName("US-ASCII");
try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
    String line = null;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
```

（2）使用 Buffered Stream I/O 写出文件；

`newBufferedWriter(Path, Charset, OpenOption...)` 方法使用 `BufferedWriter` 写出数据到文件：

```java
Charset charset = Charset.forName("US-ASCII");
String s = ...;
try (BufferedWriter writer = Files.newBufferedWriter(file, charset)) {
    writer.write(s, 0, s.length());
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
```

### Unbuffered Streams 方法协同 java.io API

（1）使用 Stream I/O 读取文件；

可以使用 `newInputStream(Path, OpenOption...)` 方法打开并读取文件，它返回一个未使用缓冲的字节输入流：

```java
Path file = ...;
try (InputStream in = Files.newInputStream(file);
    BufferedReader reader =
      new BufferedReader(new InputStreamReader(in))) {
    String line = null;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException x) {
    System.err.println(x);
}
```

（2）使用 Stream I/O 创建并写出文件；

可以使用 `newOutputStream(Path, OpenOption...)` 方法创建文件、追加数据到文件、写入数据到文件。该方法打开或创建文件并写出字节数据，它返回一个未被缓冲的字节输出流；

该方法接收 OpenOption 参数，如果未提供该参数并且目标文件不存在就创建文件，如果文件已存在，就会截断文件，等同于使用 CREATE 和 TRUNCATE_EXISTING 调用方法。

下面演示打开一个日志文件，如果该文件不存在则创建，已存在就追加数据：

```java
import static java.nio.file.StandardOpenOption.*;
import java.nio.file.*;
import java.io.*;

public class LogFileTest {

  public static void main(String[] args) {

    // Convert the string to a
    // byte array.
    String s = "Hello World! ";
    byte data[] = s.getBytes();
    Path p = Paths.get("./logfile.txt");

    try (OutputStream out = new BufferedOutputStream(
      Files.newOutputStream(p, CREATE, APPEND))) {
      out.write(data, 0, data.length);
    } catch (IOException x) {
      System.err.println(x);
    }
  }
}
```

### Channels 和 ByteBuffers 相关的方法

（1）使用 Channel I/O 读写文件；

stream I/O 一次读取一个字符，而 channel I/O 一次读取一个缓冲区。

`java.nio.channels.ByteChannel` 接口提供了基础的 read 和 write 功能，`java.nio.channels.SeekableByteChannel` 则在 ByteChannel 的基础上提供了维持 channel 中的位置以及改变位置的能力，还支持截断和 channel 关联的文件、查询文件大小的功能。

这种能力可以找到文件中的某个位置，然后从该位置开始读取或者写入数据，从而实现对文件的随机访问。

有两个方法用于读或写 channel I/O：

- `newByteChannel(Path, OpenOption...)`；
- `newByteChannel(Path, Set<? extends OpenOption>, FileAttribute<?>...)`；

> 注意：`newByteChannel` 方法返回的是 `SeekableByteChannel` 实例。结合默认的文件系统，你可以将其转换为 `FileChannel` 实例，以便使用更高级的特性，例如将文件的某个区域直接映射到内存中以实现更快的访问，锁定文件的某个区域使得其他进程无法访问它，或者从某个绝对位置读取和写入字节而不影响 channel 的当前位置。

`newByteChannel` 方法也接受一个可选的 OpenOption 参数，支持 newOutputStream 方法使用的相同的选项，另外还有一个选项：READ 是必需的，因为 SeekableByteChannel 同时支持读和写。

指定 READ 将打开读取通道。指定 WRITE 或 APPEND 打开写入通道。如果没有指定这些选项，则打开通道以供读取。

下面的代码演示了读取文件并将内容打印到标准输出：

```java
public static void readFile(Path path) throws IOException {

    // Files.newByteChannel() defaults to StandardOpenOption.READ
    try (SeekableByteChannel sbc = Files.newByteChannel(path)) {
        final int BUFFER_CAPACITY = 10;
        ByteBuffer buf = ByteBuffer.allocate(BUFFER_CAPACITY);

        // Read the bytes with the proper encoding for this platform. If
        // you skip this step, you might see foreign or illegible
        // characters.
        String encoding = System.getProperty("file.encoding");
        while (sbc.read(buf) > 0) {
            buf.flip();
            System.out.print(Charset.forName(encoding).decode(buf));
            buf.clear();
        }
    }    
}
```

下面的示例是为 UNIX 和其他 POSIX 文件系统编写的，它使用一组特定的文件权限创建一个日志文件。代码创建一个日志文件，如果日志文件已经存在，则追加到日志文件，创建日志文件时，owner 具有读写权限，group 具有只读权限。

```java
import static java.nio.file.StandardOpenOption.*;
import java.nio.*;
import java.nio.channels.*;
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.io.*;
import java.util.*;

public class LogFilePermissionsTest {

  public static void main(String[] args) {
  
    // Create the set of options for appending to the file.
    Set<OpenOption> options = new HashSet<OpenOption>();
    options.add(APPEND);
    options.add(CREATE);

    // Create the custom permissions attribute.
    Set<PosixFilePermission> perms =
      PosixFilePermissions.fromString("rw-r-----");
    FileAttribute<Set<PosixFilePermission>> attr =
      PosixFilePermissions.asFileAttribute(perms);

    // Convert the string to a ByteBuffer.
    String s = "Hello World! ";
    byte data[] = s.getBytes();
    ByteBuffer bb = ByteBuffer.wrap(data);
    
    Path file = Paths.get("./permissions.log");

    try (SeekableByteChannel sbc =
      Files.newByteChannel(file, options, attr)) {
      sbc.write(bb);
    } catch (IOException x) {
      System.out.println("Exception thrown: " + x);
    }
  }
}
```

### 创建常规或临时文件的方法

（1）创建常规文件；

您可以使用 `createFile(Path, FileAttribute<?>)` 方法创建带有初始属性集的空文件。例如，如果在创建时希望文件具有特定的文件权限集，则使用 createFile 方法来实现此目的。

如果不指定任何属性，则使用默认属性创建文件。如果文件已经存在，createFile 将抛出异常。

下面演示创建具有默认权限的文件：

```java
Path file = ...;
try {
    // Create the empty file with default permissions, etc.
    Files.createFile(file);
} catch (FileAlreadyExistsException x) {
    System.err.format("file named %s" +
        " already exists%n", file);
} catch (IOException x) {
    // Some other sort of failure, such as permissions.
    System.err.format("createFile error: %s%n", x);
}
```

（2）创建临时文件；

您可以使用以下 createTempFile 方法之一创建临时文件:

- `createTempFile(Path, String, String, FileAttribute<?>)`；
- `createTempFile(String, String, FileAttribute<?>)`；

第一个方法允许代码为临时文件指定一个目录，第二个方法在默认的临时文件目录中创建一个新文件。这两种方法都允许您为文件名指定后缀，第一个方法还允许您指定前缀。下面的代码片段给出了第二个方法的示例:

```java
try {
    Path tempFile = Files.createTempFile(null, ".myapp");
    System.out.format("The temporary file" +
        " has been created: %s%n", tempFile)
;
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
```





进度：

- https://docs.oracle.com/javase/8/javase-books.htm

- https://docs.oracle.com/javase/8/docs/

- https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html

- https://docs.oracle.com/javase/tutorial/essential/io/index.html

- https://docs.oracle.com/javase/tutorial/essential/io/rafs.html

  

