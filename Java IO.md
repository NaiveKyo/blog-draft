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

## I/O Streams

### Byte Streams

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

### Character Streams

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

### Buffered Streams

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

### Scanning and Formatting

允许程序以某种格式读取和写入文本。

在编程的时候通常会将数据流转换为人们易于阅读的形式，为了支持这些功能，Java 提供了两个 API：

- scanner API ：扫描器 API 将输入分解为和数据关联的单独的 token；
- formatting API：格式化 API 将数据以易于阅读的形式输出。

#### Scanning

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

#### Formatting

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

### I/O from the Command Line

程序通常从命令行运行，并在命令行环境中与用户交互。Java平台以两种方式支持这种交互：Standard Streams（标准流）和 Console（控制台）。

#### Standard Streams

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

#### The Console

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

### Data Streams

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

### Object Streams

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

## File I/O（Featuring NIO.2）

`java.nio.file` 包以及其关联的 `java.nio.file.attribute` 包为文件 I/O 和访问默认文件系统提供了全面的支持。尽管这些 API 涉及到很多类，但我们只需要关注几个入口点就可以了，其实这些 API 是非常方便使用的。

首先我们需要知道什么是 Path，`java.nio.file.Path` 是一个非常重要的 entry point，重点看 path 相关的操作；

然后了解 `java.nio.file.Files`，其中包含了与文件相关的操作方法，会介绍许多文件操作的通用概念，然后了解 checking、deleting、copying and moving files；

最后将学习一些非常强大且更高级的主题：

- 递归遍历文件树功能；
- 使用通配符搜索文件；
- 监视目录的更改；
- 介绍其他的一些不常用的方法。

### What's a Path？

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

### The Path Class





进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html
- https://docs.oracle.com/javase/tutorial/essential/io/index.html
- https://docs.oracle.com/javase/tutorial/essential/io/pathClass.html

