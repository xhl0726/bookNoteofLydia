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

## 附录：新IO（性能瓶颈时再看不迟）



## 附录：标准IO

Java 的标准输入输出使用装饰器模式，标准错误流 `System.err` 也预先包装为 `PrintStream` 对象，但是标准输入流 `System.in` 是原生的没有经过包装的 `InputStream`。

* 标准读入，使用 `InputStreamReader` 把 `System.in` 转换成 `Reader`，再包装成`BufferedReader`。

  ```java
  TimedAbort abort = new TimedAbort(2);
          new BufferedReader(
                  new InputStreamReader(System.in))//二次包装
                  .lines() //返回类型为Stream<String>
                  .peek(ln -> abort.restart())
                  .forEach(System.out::println);
  ```

*  `System.out` 已经预先包装成了 `PrintStream` 对象，而 `PrintStream` 是一个`OutputStream`。 `PrintWriter` 有一个把 `OutputStream` 作为参数的构造器。

  ```java
  PrintWriter out =
              new PrintWriter(System.out, true);//设置第二个参数，使能自动刷新到输出缓冲区的功能
  out.println("Hello, world");
  ```

* 重定向标准 I/O，操作的是字节流而不是字符流，从而能够重定向标准输入流、标准输出流和标准错误流。因此使用 `InputStream` 和 `OutputStream`，而不是 `Reader` 和 `Writer`。

  - setIn（InputStream）
  - setOut（PrintStream）
  - setErr(PrintStream)

* 执行控制，在Java 内部直接执行操作系统的程序。

## 附录：理解equals和hashCode方法

### equals()

**Object**对象的 **equals()**函数，会比较对象的地址，所以只有在比较同一个对象时，才会获得 **true**。 创建一个新类时，自动继承自 **Object** 类，所以需要覆写 **equals()** 方法。

一个合适的 **equals()** 函数必须满足五点：

1. 反身性，**x.equals(x)** 返回 **true** 。
2. 对称性，**x.equals(y)** 返回 true 当且仅当 **y.equals(x)** 返回 true。
3. 传递性，**x.equals(x)** 且**y.equals(x)** 返回 true，**x.equals(z)** 应该返回 true。 
4. 一致性，**x.equals(y)** 多次调用应该返回 true/false。
5. 非 **null** 的 **x**， **x.equals(null)** 应返回 false。

Java 7 引入了 **Objects** 类型帮助这个覆写流程，右值需要满足：

1. 如果右值是 null，则不相等。
2. 右值是 this ， 则两个对象相等。
3. 右值不是同一个类型或子类，则两个对象不相等。`instanceof`判断类型。
4. 以上检查通过后，决定**右值**中哪些字段是重要的，然后比较这些字段，可使用`Objects.equals()`。对于子类，可调用`super.equals(target)`先判断。此调用没必要覆写，因为你不总是有权访问基类所有的必要字段。

### 不同子类的相等性

>  **旁注**： 在**hashCode()**中，如果你只能够使用一个字段，使用**Objcets.hashCode()**。如果你使用多个字段，那么使用 **Objects.hash()**。

只从基类的角度看待问题——李氏替换原则的基石。本节的代码范例（重写基类的equals()和hashcode()方法，向一个 HashSet 中放置两个同样数据的子类对象，只保存一个），完美符合替换理论，因为派生类没有添加任何额外的不在基类中的函数，派生类只是在表现上不同，而不是接口上，当然这不是常态。

 **hashCode()** 和 **equals()** 必须能够允许类型在hash数据结构中正常工作。

可以使用多个字段建立 hashCode(包含 id ) 或者，在子类中覆写 equals(不包含id ) 解决这个问题。第二种解决办法 hashCode() 依旧是独一无二的。

```java
 @Override
    public boolean equals(Object rval) {
        return rval instanceof Pig2 &&
        super.equals(rval);
    } 
```

### 哈希和哈希码

**Object** 类的 **hashCode() **生成散列码，默认使用对象的地址计算散列码。

**Object**对象的 **equals()**函数，会比较对象的地址。

**HashMap** 使用 **equals()** 判断当前的键是否与表中存在的键相同，使用 **hashCode()** 来进行查找。

因此要使用自己的类作为 **HashMap** 的键，必须同时重载 `hashcode()`和`equals()`。

> hashCode()并不需要总是能够返回唯一的标识码，但是equals() 方法必须严格地判断两个对象是否相同。

优秀代码赏析：

```java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import java.lang.reflect.*;
...//省略类定义等
public static <T extends Groundhog> //方法使用泛型参数，一定要将泛型声明放在前面
    void detectSpring(Class<T> type) { //传参一个类的类型
        try {
            Constructor<T> ghog =
                    type.getConstructor(int.class);//获取该类的一个构造器
            Map<Groundhog, Prediction> map =
                    IntStream.range(0, 10)
                            .mapToObj(i -> {
                                try {
                                    return ghog.newInstance(i);//构造器的调用方法
                                } catch(Exception e) {
                                    throw new RuntimeException(e);
                                }
                            })
                            .collect(Collectors.toMap(//
                                    Function.identity(),
                                    gh -> new Prediction()));//Prediction 是一个类 
            map.forEach((k, v) ->
                    System.out.println(k + ": " + v));
            Groundhog gh = ghog.newInstance(3);
            System.out.println(
                    "Looking up prediction for " + gh);
            if(map.containsKey(gh))
                System.out.println(map.get(gh));
            else
                System.out.println("Key not found: " + gh);
        } catch(NoSuchMethodException |
                IllegalAccessException |
                InvocationTargetException |
                InstantiationException e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args) {
        detectSpring(Groundhog.class);
    }
```

### 理解 hashCode

如果你不为你的键覆盖`hasCode()`和`equals()`，那么使用散列的数据结构（HashSet，HashMap，LinkedHashSet、LinkedHashMap）就无法正确处理你的键。

使用散列的目的在于：想要使用一个对象来查找另一个对象。

散列的价值在于速度： 散列使得查询得以快速进行，由于瓶颈位于键的查询速度，因此解决方案之一是保持键的排序状态，然后使用 `Collections.binarySearch()`进行查询。

散列更进一步，将**键的信息**保存在数组中（存储一组元素最快的数据结构是数组），但是数组不能调整容量。如果键的数量被数组的容量限制了，该怎么办呢？

答案就是：数组并不保存键本身。而是通过键对象生成一个数字，将其作为数组的下标。这个数字就是散列码，由定义在Object中的、且可能由你的类覆盖的hashCode()方法（在计算机科学的术语中称为散列函数）生成。

**查询一个值的过程**:

首先就是计算散列码，然后使用散列码查询数组。如果能够保证没有冲突（如果值的数量是固定的，那么就有可能），那可就有了一个完美的散列函数，但是这种情况只是特例。通常，冲突由**外部链接处理**：数组并不直接保存值，而是保存值的 list。然后对 list中的值使用equals()方法进行线性的查询。这部分的查询自然会比较慢，但是，如果散列函数好的话，数组的每个位置就只有较少的值。因此，不是查询整个list，而是快速地跳到数组的某个位置，只对很少的元素进行比较。这便是HashMap会如此快的原因。

**外部链接处理**方法：

为了能够自动处理冲突，使用了一个LinkedList的数组；每一个新的元素只是直接添加到list尾的某个特定桶位中。即使Java不允许你创建泛型数组，那你也可以创建指向这种数组的引用。这里，向上转型为这种数组是很方便的，这样可以防止在后面的代码中进行额外的转型。

如：`LinkedList<MapEntry<K, V>>[] buckets = new LinkedList[SIZE];`

散列表中的“槽位”（slot）通常称为桶位（bucket），因此我们将表示实际散列表的数组命名为bucket，为使散列分布均匀，桶的数量通常使用质数。

### 重写 hashCode()

首先，我们无法控制 bucket 数组的下标值的产生，这个值依赖于具体的 HashMap 对象的容量。hashCode()生成的结果，经过处理后成为桶位的下标（在SimpleHashMap中，只是对其取模，模数为bucket数组的大小）。

设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该生成同样的值。`put()`和`get()`的`hashCode()`值应当相同，`hashCode()`不应依赖于对象中易变的数据，也不应依赖于具有唯一性的对象信息，尤其是`this`，这样无法生存一个对应`put()`的新键。

因此，要想使`hashCode()`实用，

1. 必须速度快，且有意义，不必是独一无二的！但是通过 hashCode() 和 equals() ，必须能够完全确定对象的身份。
2. 好的hashCode() 应该产生分布均匀的散列码。如果散列码都集中在一块，那么HashMap或者HashSet在某些区域的负载会很重，这样就不如分布均匀的散列函数快。


### 调优 HashMap

我手动调优HashMap以提高其在特定应用程序中的性能时，为了理解调整HashMap时的性能问题，一些术语是必要的：

- 容量（Capacity）：表中存储的桶数量。
- 初始容量（Initial Capacity）：当表被创建时，桶的初始个数。 HashMap 和 HashSet 有可以让你指定初始容量的构造器。
- 个数（Size）：目前存储在表中的键值对的个数。
- 负载因子（Load factor）：通常表现为 size/capacity。当负载因子大小为 0 的时候表示为一个空表。当负载因子大小为 0.5 表示为一个半满表（half-full table），以此类推。**轻负载的表几乎没有冲突，因此是插入和查找的最佳选择（但会减慢使用迭代器进行遍历的过程）**。 HashMap 和 HashSet 有可以让你指定负载因子的构造器。**当表内容量达到了负载因子，集合就会自动扩充为原始容量（桶的数量）的两倍**，并且会将原始的对象存储在新的桶集合中（也被称为 rehashing）。

HashMap 中负载因子的大小为 0.75，这看起来是一个同时考虑到时间和空间消耗的平衡策略。

更高的负载因子会减少空间的消耗，但是会增加查询的消耗，查询操作是你使用的最频繁的一个操作（包括 `get()` 和 `put()` 方法）。

如果知道 HashMap 中存储的确切条目个数，直接创建一个足够容量大小的 HashMap，以避免自动发生的 rehashing 操作。

## 附录：文档注释

编写代码文档的最大问题——维护文档，解决方案——将代码链接到文档。

Java 要做的：

* 一个特殊的注释语法来标记文档
* 一个工具来将这些注释提取为有用的表单。称为 Javadoc，它是 JDK 安装的一部分。它不仅提取由这些标记所标记的信息，还提取与注释相邻的类名或方法名。通过这种方式，可以用**最少的工作量来生成合适的程序文档**。

Javadoc 输出为一个 html 文件，还可以编写自己的 Javadoc 处理程序。

### 句法规则

1. 所有的 Javadoc 指令都发生在 `/**  ...  **/`的注释中。

2. 使用 Javadoc 的两种方法：

   * 嵌入 HTML 或 doc标签。doc标签指以`@`开头，放在注释行的开头。可能会出现内联 doc 标签。
   * Javadoc 主食中的任何位置，以一个`@`开头，被花括号包围。

3. 有三种类型的注释文档，它们对应于注释前面的元素:类、字段或方法。

4. Javadoc处理注释文档仅适用于 **公共** 和 **受保护** 的成员。 

5. 要通过Javadoc处理前面的代码，命令是：

   ```cmd
   javadoc Documentation1.java //
   ```

   这将产生一组HTML文件。

6. 不要将诸如 \<h1\>或 \<hr\>之类的标题用作嵌入式HTML，因为 Javadoc 会插入自己的标题，后插入的标题将对其生成的文档产生干扰。

7. 示例标签

   * `@see` 将其他的类连接到文档中， Javadoc 将使用 @see 标记**超链接**到其他文档中，但是不检查其有效性。
   * `{@link package.class#member label}`和 @see 非常相似，不同之处在于它可以内联使用并使用标签作为超链接文本，而不是“另请参阅”。
   * `{@docRoot}` 生成文档根目录的相对路径。对于显式超链接到文档树中的页面很有用。
   * `{@inheritDoc}` 将文档从此类的最近基类集成到当前文档注释中。
   * `@version` 形式为`@version version-information`
   * `@author` 形式为 `@author author-information`可连续使用多个。
   * `@since` 此标记指示此代码的版本开始使用特定功能。
   * `@param` 可以任意使用，大概每个参数一个。
   * `@return` 描述返回值
   * `@throws` 使用形式为`@throws fully-qualified-class-name description`。 fully-qualified-class-name 给出明确的异常分类名称，并且 description （可延续到后面的行内）告诉你为什么这特定类型的异常会在方法调用后出现。
   * `deprecated`，表示已被改进的功能取代的功能，表明你不再使用此特定功能，因为将来有可能将其删除。标记为@不赞成使用的方法会导致编译器在使用时发出警告。在Java 5中，@deprecated Javadoc 标记已被 @Deprecated 注解取代。

   

## 附录：并发底层原理



## 附录：对象序列化

### Serializable

对象序列化将实现了 `Serializable `接口的对象转换为一个**字节序列**，并能够在以后将这个字节序列完全恢复为原来的对象（**还原过程不调用构造器**）。序列化机制能自动弥补不同操作系统之间的差异。

* 一个标识接口

* 轻量级持久性
* 不仅保存了对象的“全景图”，而且能追踪对象内所包含的所有引用并保存那些对象。
* 对非 transient 和非 static 的属性进行序列化。
* 支持 Java 的两种主要特性：远程方法调用（传输参数和返回值）和 Java Bean（保存状态信息，程序启动时进行后期恢复）。
* Java 的对象序列化很OK，尽量不要自己动手。

### 控制序列化

* 实现`Externalizable`接口代替`Serializable`接口——来对序列化过程进行控制。` Externalizable` 接口继承了 `Serializable` 接口，增添了两个方法：`writeExternal()` 和` readExternal()`。这两个方法会在序列化和反序列化还原的过程中被自动调用，在方法体内写入想要序列化/反序列化的元素。**所有默认的构造器都会被调用。一定要有默认的无参构造器**。

* `transient`关键字逐个字段关闭序列化。

* 实现`Serializable`接口，并**添加而非覆盖**`writeObject()`和`readObject()`。这样一旦对象被序列化或者反序列化还原，就会自动分别调用这两个方法。

  ```java
  private void writeObject(ObjectOutputStream stream) throws IOException
  private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException
  ```

### 序列化对象和反序列化

**要序列化一个对象**

* 创建某些 OutputStream 对象

* 然后将其封装在一个 ObjectOutputStream 对象内。

* 只需调用 writeObject() 即可将对象序列化，并将其发送给 OutputStream（对象化序列是基于字节的，因要使用 InputStream 和 OutputStream 继承层次结构）。


```java
ObjectOutputStream out = new ObjectOutputStream(
                        new FileOutputStream("worm.dat"))
out.writeObject("Worm storage\n");
out.writeObject(w);//Worm w = new Worm(6, 'a');
```

**将一个序列还原为一个对象**

* 将一个 InputStream 封装在 ObjectInputStream 内，
* 调用 readObject()。和往常一样，我们最后获得的是一个引用，它指向一个向上转型的 Object，所以必须向下转型才能直接设置它们。

```java
ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("worm.dat"))
String s = (String)in.readObject();
Worm w2 = (Worm)in.readObject();
```

### XML

对象序列化：只有 Java 程序才能反序列化这种对象。一种更具互操作性的解决方案是将数据转换为 XML 格式。

类库： XOM，直观的用 java 产生和修改 XML，且强调了 XML 的正确性。

## 附录：编程指南

### 设计

1. 优雅总是会有回报。
2. 先运行起来，然后再让它变快。尽可能简单的设计先跑起来。
3. 解问题时分而治之
4. 将类创建者和类用户（客户端程序员）分开。他们不想知道内部原理！
5. 给类起个不用注释也能理解的名字。
6. 让一切自动化，编写类之前编写测试代码，使用构建工具比如 Gradle。
7. 使类尽可能原子化，每个类有一个明确的目的。如果一个类很大，那么它可能是做的事太多了，应该被拆分。建议重新设计类的线索是：
   - 一个复杂的*switch*语句：考虑使用多态。
   - 大量方法涵盖了很多不同类型的操作：考虑使用多个类。
   - 大量成员变量涉及很多不同的特征：考虑使用多个类。
   - 其他建议可以参见Martin Fowler的*Refactoring: Improving the Design of Existing Code*（重构：改善既有代码的设计）（Addison-Wesley 1999）。
8. 注意长参数列表。
9. 将重复代码放入公共方法。
10. 扩展基本功能先考虑基类扩展。
11. 对于接口，少即是多，不要试图预测类的所有使用方式。
12. 从现有类创建新类时，组合优先于继承。
13. 注意重载。
14. 注意巨型对象，除应用程序框架外，对象代表应用程序中的概念，而不是应用程序本身。
15. 对象应当持有一些数据，有明确的行为。
16. 使用继承和覆盖方法来表达行为的差异，而不是使用字段来表示状态的变化。
17. 注意协变。
18. 让别人来评判你的工作。

### 实现

1. 遵循编码惯例。
2. 团队标准化编码风格。
3. 遵循标准的大写规则。
   - **ThisIsAClassName**
   - **thisIsAMethodOrFieldName**
   - 将 **static final** 类型的标识符的所有字母全部大写，并用下划线分隔各个单词，这些标识符在其定义中具有常量初始值。这表明它们是编译时常量。
   - **包是一个特例**，它们都是小写的字母，即使是中间词。
4. 对读取和更改私有字段的方法使用 getset。
5. 对于所创建的每个类，包含该类的JUnit 测试。
6. 有时需要继承才能访问基类的 protected 成员。如果不需要向上转型，则可以首先派生一个新类来执行受保护的访问。然后把该新类作为使用它的任何类中的成员对象，以此来代替直接继承。
7. 为了提高效率，避免使用 **final** 方法。
8. 两个类以某种功能互相关联，则尝试使一个类称为另一个类的内部类。强调了类的关联、允许在单个包中重用类名，内部类作为私有实现的一部分。
9. 类彼此之间高耦合，考虑内部类是否能获得编码和维护改进。
10. 不要成为过早优化的牺牲品。
11. 保持作用域尽可能小，以便能见度和对象的寿命尽可能小。
12. **使用标准 Java 库中的集合**。首选**ArrayList**用于序列，**HashSet**用于集合，**HashMap**用于关联数组，**LinkedList**用于堆栈（而不是**Stack**，尽管也可以创建一个适配器来提供堆栈接口）和队列（也可以使用适配器，如本书所示）。当使用前三个时，将其分别向上转型为**List**，**Set**和**Map**，那么就可以根据需要轻松更改为其他实现。
13. 编译时错误优于运行时错误。
14. 注意长方法定义。
15. 保持尽可能私有。
16. 大量使用注释，并使用*Javadoc commentdocumentation*语法生成程序文档
17. 避免直接使用数字，相反创建一个带有描述性名称的常量。
18. 创建方法时考虑异常。
19. 构造方法内部只需要将对象设置为正确的状态，主动避免调用其他方法（final 方法除外）。
20. 将对对象的清理代码放在一个明确定义的方法中。并使用像 **dispose()** 这样的名称来清楚地表明其目的。另外，在类中放置一个 **boolean** 标志来指示是否调用了 **dispose()** ，因此 **finalize()** 可以检查“终止条件”。
21. ***finalize()* 的职责只能是验证对象的“终止条件”以进行调试**。
22. **如果必须在特定范围内清理对象（除了通过垃圾收集器），请使用以下准则：** 初始化对象，如果成功，立即进入一个带有 **finally** 子句的 **try** 块，并在 **finally**中执行清理操作。
23. **在继承期间覆盖 *finalize()* 时，记得在最后一行调用 *super.finalize()***。（如果是直接继承自 **Object** 则不需要这样做。）确保基类组件在需要时仍然有效。
24. 创建固定大小的对象集合时，将他们转换为数组。
25. 优先选择接口而不是抽象类。
26. 类路径的每个名称只对有一个不在包中的类。
27. 注意意外重载。始终使用**@Override**。
28. **请注意，相比于编写代码，代码被阅读的机会更多。**



------------------------

-------------------



## 附录：集合主题（可参考代码）

## 附录：压缩（GZIP,ZIP,JAR）



## 后记：Lydia的个人私藏及灵魂发文

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

### Java 有指针么？

> 如果你之前使用类似C++的编程语言，则是“ Java是否有指针？” Java中的每个对象标识符（除原语外）都是这些指针之一，但它们的用法是不仅受编译器的约束，而且受运行时系统的约束。 换一种说法，Java有指针，但没有指针算法。 这些就是我一直所说的“引用”，您可以将它们视为“安全指针”，与小学的安全剪刀不同-它们不敏锐，因此您不费吹灰之力就无法伤害自己，但是它们有时可能很乏味。

Java 的所谓指针，其实是将引用传递给方法时，它仍指向同一对象（即地址相同）。这也是对集合操作如复制时，需要 deepCopy 的原因之一。