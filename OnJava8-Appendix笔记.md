[TOC]

# Java 编程思想读书笔记

## 附录：流式 IO

-------

Java I/O 系统，所有与输入有关系的类都继承自`InputStream`，所有与输出有关的类继承自`OutputStream`。所有从 `InputStream` 或 `Reader` 派生而来的类都含有名为 `read()` 的基本方法，用于读取单个字节或者字节数组。同样，所有从 `OutputStream` 或 `Writer` 派生而来的类都含有名为 `write()` 的基本方法，用于写单个字节或者字节数组。

Java 的**I/O 流操作**（和`Stream`类无关）通常通过叠合多个对象来提供所期望的功能（**装饰器设计模式**）。

Java 1.0 之后，Java 的 I/O 类库发生了明显的改变，在原来面向字节的类中添加了面向字符和基于 Unicode 的类。在 Java 1.4 中，为了改进性能和功能，又添加了 `nio` 类。

### 输入流类型：InputStream

表示从不同数据源产生输入的类，数据源包括：

1. 字节数组：`ByteArrayInputStream`，构造器参数为缓冲区。
2. `String` 对象：`StringBufferInputStream`，构造器参数为字符串，底层实现实际使用`StringBuffer`。
3. 文件：`FileInputStream`，构造器参数为表示文件名、文件或`FileDescriptor`对象的字符串。
4. “管道”，工作方式与实际生活中的管道类似：从一端输入，从另一端输出，作为多线程中的数据源：`PipedInputStream`，构造器参数`PipedOutputSteam`。
5. 一个由其它种类的流组成的序列，然后我们可以把它们汇聚成一个流：`SequenceInputStream`。构造器参数两个 `InputStream` 对象或一个容纳 `InputStream` 对象的容器 `Enumeration`。
6. 其它数据源，如 Internet 连接。
7. 一个抽象类：`FilterInputStream`，作为“装饰器”的接口，其中装饰器为其他`InputStream`提供有用功能。以下列出的类构造器参数都为`InputStream`。
   * `DataInputStream`：与 `DataOutputStream` 搭配使用，按照移植方式从流读取基本数据类型（`int`、`char`、`long` 等），使用方法：包含用于读取基本数据类型的全部接口，（所有方法都以 “read” 开头，例如 `readByte()`、`readFloat()`等等）。
   * `BufferedInputStream`：“使用缓冲区”，防止每次读取时都得进行实际写操作。构造器参数可以指定缓存区大小（可选）。使用方法：本质上不提供接口，只是向进程添加缓冲功能。与接口对象搭配。
   * `LineNumberInputStream`：跟踪输入流中的行号，可调用 `getLineNumber()` 和 `setLineNumber(int)`。使用方法：仅增加了行号，因此可能要与接口对象搭配使用。
   * `PushbackInputStream`  ：具有能弹出一个字节的缓冲区，因此可以将读到的最后一个字符回退。通常组我诶编译器的扫描器，程序员不怎么用得到。

### 输出流类型：OutputStream

1. 字节数组：`ByteArrayOutputStream`，构造器参数为缓冲区初始大小（可选）。
2. 文件：`FileOutputStream`，构造器参数为表示文件名、文件或`FileDescriptor`对象的字符串。
3. “管道”，任何写入其中的信息都会自动作为相关 `PipedInputStream` 的输出：`PipedOutputStream`，构造器参数`PipedInputSteam`。指定用于多线程的数据的目的地。
4. 一个抽象类：`FilterOutputStream`，作为“装饰器”的接口，其中装饰器为其他`OutputStream`提供有用功能。以下列出的类构造器参数都为`InputStream`。
   * `DataOutputStream`：与 `DataInputStream` 搭配使用，按照移植方式向流中写入基本数据类型（`int`、`char`、`long` 等），使用方法：包含用于读取基本数据类型的全部接口，（所有方法都以 “write” 开头，例如 `writeByte()`、`writeFloat()`等等）。
   * `BufferedOutputStream`：“使用缓冲区”，防止每次发送数据时都得进行实际写操作。可以调用`flush()`清空缓冲区。构造器参数可以指定缓存区大小（可选）。使用方法：本质上不提供接口，只是向进程添加缓冲功能。与接口对象搭配。
   * `PrintStream()`：用于产生格式化输出。其中 `DataOutputStream` 处理数据的存储（置于“流”中），`PrintStream` 处理显示（内有两个重要方法：`print()` 和 `println()`）。构造器可以用 `boolean` 值指示是否每次换行时清空缓冲区（可选）。

### Reader和Writer

 `InputStream` 和 `OutputStream` 在**面向字节 I/O**（8 比特的字节流） 这方面仍然发挥着极其重要的作用，而 `Reader` 和 `Writer` 则提供兼容 Unicode （16比特的 Unicode 字符，Java 本身的 char 也是16 bite 的 Unicode）和**面向字符 I/O** 的功能。

有时我们必须把来自“字节”层级结构中的类和来自“字符”层次结构中的类结合起来使用。为了达到这个目的，需要用到“适配器（adapter）类”：`InputStreamReader` 可以把 `InputStream` 转换为 `Reader`，而 `OutputStreamWriter` 可以把 `OutputStream` 转换为 `Writer`。

**尽量尝试**使用 `Reader` 和 `Writer`，一旦代码没法成功编译，你就会发现此时应该使用面向字节的类库了。

**流的行为**，对于 `InputStream` 和 `OutputStream` 来说，我们会使用 `FilterInputStream` 和 `FilterOutputStream` 的装饰器子类来修改“流”以满足特殊需要。`Reader` 和 `Writer` 的类继承体系沿用了相同的思想——但是并不完全相同。比如：

1. `BufferedOutputStream` 是 `FilterOutputStream` 的子类，但 `BufferedWriter` 却不是 `FilterWriter` 的子类（尽管 `FilterWriter` 是抽象类，但却没有任何子类，把它放在表格里只是占个位置）。
2. 一旦要使用 `readLine()`，我们就不应该用 `DataInputStream`（否则，编译时会得到使用了过时方法的警告），而应该使用 `BufferedReader`。除了这种情况之外的情形中，**`DataInputStream` 仍是 I/O 类库的首选成员**。
3. 为了使用时更容易过渡到 `PrintWriter`，它提供了一个既能接受 `Writer` 对象又能接受任何 `OutputStream` 对象的构造器。`PrintWriter` 的格式化接口实际上与 `PrintStream` 相同。
4. 其中一种 `PrintWriter` 构造器还有一个执行**自动 flush** 的选项。如果构造器设置了该选项，就会在每个 `println()` 调用之后，自动执行 flush。

### 未发生改变的类

* `DataOutputStream`
* `File`
* `RandomAccessFile`适用于由大小已知的记录组成的文件。该类不是 `InputStream` 或者 `OutputStream` 继承体系中的一部分。
* `SequenceInputStream`

### IO流典型用途

* **缓冲输入文件**

  如果想要打开一个文件进行字符输入，可以使用一个 `FileInputReader` 对象，然后传入一个 `String` 或者 `File` 对象作为文件名。为了提高速度，我们希望对那个文件进行缓冲，那么我们可以**将所产生的引用传递给一个 `BufferedReader` 构造器**。`BufferedReader` 提供了 `line()` 方法，它会产生一个 `Stream<String>` 对象。

  ```java
  BufferedReader in = new BufferedReader(new FileReader("filename"));
  return in.lines().collect(Collectors.joining("\n"));
  ```

  

* **从内存中读取数据**

  从 `BufferedInputFile.read()` 读入的 `String` 被用来创建一个 `StringReader` 对象。

  ```java
  StringReader in = new StringReader(BufferedInputFile.read("xx.txt"));
  ```

  

* **从内存中读取格式化数据**

  要读取格式化数据，我们可以使用 `DataInputStream`，它是一个面向字节的 I/O 类（不是面向字符的）。这样我们就必须使用 `InputStream` 类而不是 `Reader` 类。我们可以使用 `InputStream` 以字节形式读取任何数据（比如一个文件）。

  ```java
  DataInputStream in = new DataInputStream(new ByteArrayInputStream(
                      BufferedInputFile.read("FormattedMemoryInput.java").getBytes());
  System.out.write((char) in.readByte());//逐字节读入    
                                           
  DataInputStream in = new DataInputStream(new BufferedInputStream(
                      new FileInputStream("TestEOF.java")));
  while (in.available() != 0) //在没有阻塞情况下能读取的字节数
                  System.out.write(in.readByte());
  ```

  `ByteArrayInputStream` 必须接收一个字节数组，所以这里我们调用了 `String.getBytes()` 方法。

  

* **基本文件的输出**

  `FileWriter` 对象用于向文件写入数据。实际使用时，我们通常会用 `BufferedWriter` 将其包装起来**以增加缓冲的功能**。为了提供**格式化功能**，可以用`PrintWriter`装饰`BufferedWriter`。

  ```java
  PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("filename")));
  ```

  

* **文本文件输出快捷方式**

  Java 5 在 `PrintWriter` 中添加了一个辅助构造器，可以不必手动添加缓冲。

  ```java
  PrintWriter out = new PrintWriter("filename");
  ```

  

* **存储和恢复数据**

  要输出可供另一个“流”恢复的数据，我们可以用 `DataOutputStream` 写入数据，然后用 `DataInputStream` 恢复数据。注意 `DataOutputStream` 和 `DataInputStream` 是面向字节的，因此要使用 `InputStream` 和 `OutputStream` 体系的类。

  ```java
  DataOutputStream out = new DataOutputStream(new BufferedOutputStream(
                      new FileOutputStream("Data.txt")))
  out.writeDouble(3.14159);
  out.writeUTF("That was pi");
  DataInputStream in = new DataInputStream(new BufferedInputStream(
                      new FileInputStream("Data.txt")))
  System.out.println(in.readDouble());
  System.out.println(in.readUTF());
  ```

  使用 `DastaOutputStream` 时，写字符串并且让 `DataInputStream` 能够恢复它的唯一可靠方式就是使用 UTF-8 编码，在这个示例中是用 `writeUTF()` 和 `readUTF()` 来实现的。但是这两个方法使用的是一种适用于 Java 的 UTF-8 变体（见JDK 文档），因此如果我们用一个非 Java 程序读取用 `writeUTF()` 所写的字符串时，必须编写一些特殊的代码才能正确读取。

  使用`DataOutputStream`的方法进行读取及存储时：

  * 要么为文件中的数据采用固定的格式；
  * 要么将额外的信息保存到文件中，通过解析额外信息来确定数据的存放位置。
  * 注意，对象序列化和 XML 是存储和读取复杂数据结构的更简单的方式。

  

* **读写随机访问文件**

   `RandomAccessFile` 实现了接口：`DataInput` 和 `DataOutput`）。使用前必须清楚文件的结构。有一套专门的方法来读写基本数据类型的数据和 UTF-8 编码的字符串：

  ```java
  RandomAccessFile rf = new RandomAccessFile(file, "r");//只读
  RandomAccessFile rf = new RandomAccessFile(file, "rw");//读写
  rf.writeUTF("The end of the file");
  rf.seek(5 * 8);
  rf.writeDouble(47.0001);
  rf.close();
  ```

   `double` 总是 8 字节长，所以如果要用 `seek()` 定位到第 5 个（从 0 开始计数） `double` 值，则要传入的地址值应该为 `5*8`。

### 本章小结

理解“装饰器”模式，I/O流类库有用且具有可移植性。

务必先试试**Files**，**Path**... 本章前面的技术，再考虑 I/O 流库。

## 附录：新IO





## 附录：标准IO



## 附录：理解equals和hashCode方法



## 附录：文档注释



## 附录：并发底层原理



## 附录：对象序列化





## 附录：编程指南



------------------------

-------------------



## 附录：集合主题

## 附录：压缩

## 后记：Lydia的个人私藏

> 除非你准备致力于终身学习，否则请不要从事这项业务。有时，编程似乎是一份报酬丰厚，值得信赖的工作，但确保这一点的唯一方法是，始终使自己变得更有价值。

- 将学习作为一种生活方式。例如，学习一种以上的语言；没有什么比学习另一种语言更能吸引你的眼球。
- 知道在哪里以及如何获得新知识。
- 研究现有技术。
- 我们是工具使用者，即要善于利用工具。
- 学习做最简单的事情。
- 了解业务（阅读杂志。从 *fast company*（国外一家商业杂志）开始，该公司的文章非常简短有趣。然后你就会知道是否要阅读其他的）
- 应对错误负责。 “我用着没事”是不可接受的策略。查找自己的错误。
- 成为领导者：那些沟通和鼓舞别人的人。
- 你在为谁服务？
- 没有正确的答案……但总是更好的方法。展示和讨论你的代码，不要有情感上的依恋。你不是你的代码。
- 这是通往完美的渐进旅程。

承担一切可能的风险，最好的风险是那些可怕的风险，但是在尝试时你会比想象中的更加活跃。最好不要刻意去预测某个特定的结果，因为如果你过于重视某个结果，就会经常错过真正的可能性。应该“让我们做一点实验，看看会把我们带到哪里”。这些实验是我最好的冒险。