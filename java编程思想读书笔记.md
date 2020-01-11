[TOC]



# JAVA编程思想读书笔记
## 第十一章 内部类

> 一个定义在另一个类中的类叫做内部类。

内部类允许把一些逻辑相关的类组合在一起，并控制位于内部的类的可见性。*but*，内部类和组合是两个概念。

内部类类似代码隐藏机制，且了解外围类并与之通信，用内部类写出来的代码更简洁优雅。

如果想从外部类的任意位置创建某个内部类的对象

* 外部类的静态方法中，指明类型 *InnerClassName*

* 外部类的非静态方法之外的任意位置/其他类中，需要指明*OuterClassName.InnerClassName*



### 链接外部类

内部类自动拥有对其外围类所有成员的访问权限。实现原理是当某个外围类的对象创建了一个内部类对象时，该内部类对象必定会秘密捕获一个指向外围类的引用（编译器自动处理，同样的，如果编译器访问不到这个引用就会报错）

内部类的对象只有在于其外围类的对象相关联的情况下才能被创建（当内部类为非**static**时）。

**用例**一：在类中内置一个内部类与一个接口对象，且该内部类**implements**该接口，内部类对外围类的字段进行操作；可通过访问接口对象，实现对外围类的操作。



### 使用*.this*和*.new*

在内部类中使用 *OuterClassName.this* 生成对外部类的引用，如` return DotThis.this`。

使用外部类的对象来创建该内部类的对象，必须在**new**表达式中提供对其他外部类对象的引用。如

```java
OuterClassName oc = new OuterClassName();
OuterClassName.InnerClassName ic = oc.new InnerClassName();
```

在拥有外部类对象前不可能创建内部类对象的。因为内部类对象会暗戳戳把自己连接到建它的外围类对象上。

但是，如果你创建的是嵌套类（静态内部类），那么它不需要对外部类的引用。



### 内部类和向上转型

内部类的访问权限可以被设置为**private**（只有外围类可以访问）或者**protected**（只有外围类及外围类的子类，还有与外围类同一个包中的类可以访问）；普通类只能被设置为**public**或**package**访问权限。

**用例一**的情况，将内部类设置为**private/protected**，将内部类向上转型为其基类尤其是转型为一个接口，（从实现了某个接口的对象，得到对该接口的引用，与向上转型为这个对象的基类，实质上效果一样），由于**内部类-某个接口的实现-能够完全不可见且不可用**，可以很好地隐藏细节，**阻止依赖类型的编码**，只把这个引用作为指向基类或接口的引用来用。



### 内部类方法和作用域

可以在一个方法里面或任意的作用域内定义内部类。

* 如用例一，实现了某类型的接口，可以创建并返回对其的引用。

* 要解决一个问题，需要一个辅助类，但不希望此类公共可用。

在方法的作用域中嵌入一个类命名为*A*，称为**局部内部类**，其可用域受到所在域的限制，且不能用访问说明符，因为它不是外围类的一部分，但是它可以访问当前代码块内的常量及此外围类的所有成员。

使用局部内部类而不是匿名内部类的原因：

* 我们需要一个已命名的构造器或重载构造器，匿名内部类只能用于实例初始化。
* 需要不止一个该内部类的对象。



### 匿名内部类

用法：将返回值的生成与表示这个返回值的类的定义结合在一起，如`return new Content(){};`，在后面的方法体内实例化**abstract**类或直接给类定义，直接返回匿名类的引用。（需要注意的是最后的分号是正常表达式的结束，只不过刚好包含了匿名内部类。）

**不带参**或者**传递外部定义参数的引用**都可以来生成匿名类对象，当使用后者时，编译器会要求参数引用是**final**的（即在初始化后不会改变，换句话说传参`final String s`或`int i`都可，其实**final**省略 Java 8 编译器也会自动给加上的但是最好不要啦哈哈）。 

匿名类中定义字段，需要执行初始化操作。

匿名类中不可能有命名构造器，但通过**实例初始化**`return new Content(){{“此处实例初始化”};};`的操作可以做一些类似构造器的行为。

匿名内部类的受限在于：扩展类和实现接口只能选其一，且只能实现一个接口。



### 嵌套类

和外围类对象之间没有联系，以**static**声明的内部类叫做嵌套类。

* 嵌套类的对象创建不需要外围类的对象。即不需要用`oc.new`。

* 不能从嵌套类的对象中访问非静态的外围类对象。即不能用`OuterClassName.this`。

嵌套类可作为接口的一部分，接口中的任何类都是自动**public**和**static**的，甚至可以实现外围接口。

上面一行的引用操作——使用嵌套类写`main()`方法对外围类进行测试。

内部类被嵌套多少层，都能**透明的访问它所嵌入的所有外围类的所有成员**，即使外围类成员被定义为**private**。



### 内部类的用处

内部类可以继承多个具体的或抽象类的能力，为**多重继承**的解决方案提供新思路。

用法：当必须在一个类中以某种方式实现两个接口，可以使用单一类直接**implements**两个接口，或者，**implements**一个接口并且使用内部类。如果必须实现两个类，那就只能用内部类。

* 内部类可以有多个实例，每个实例都有自己的状态信息且与其外围类对象的信息互相独立。

* 内部类就是一个独立的实体，而没有“is-a”关系。



### 闭包与回调

**闭包**是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。

内部类是面向对象的闭包，包含了外围类对象的信息，且自动拥有一个对外围类对象的引用，且有权操作所有的成员。

生成闭包的方式

* 内部类

* lambda 表达式（Java 8 之后）

闭包比指针更灵活、安全，可通过闭包实现回调——对象能够携带一些信息，这些信息允许它在稍后的某个时刻调用初始的对象（用法就是使用（闭包）内部类实现多重继承，且可以返回`OuterClassName.this`对外围类对象进行操作）。



### 内部类与控制框架

应用程序框架就是被设计用以解决某类特定问题的一个类或一组类。用法通常是继承一个或多个类并覆盖某些方法。

控制框架是一类特殊的应用程序框架，用来解决响应事件的需求。

事件驱动系统就是说主要用来响应事件的系统，比如图形用户接口（GUI）。

内部类允许：

* 控制框架的完整实现由单个类创建，从而使实现的细节被封装起来，内部类用来表示解决问题所需的各种`action()`。即创建多个内部类继承一个接口，这些内部类就实现了不同的`action()`。

* 内部类可访问外围类的任意成员。



### 继承内部类和外围类

内部类的构造器必须连接到指向其外围类对象的引用，因此在继承内部类时，必须**初始化**那个引用。

* 需要在继承类的构造器参数中传递一个外围类的引用

* 在构造器方法体中使用这一行语法：`enclosingClassReference.super();`

继承外围类时，在新类中“重写”同名内部类，两个同名类完全独立，没有被覆盖。

当然，明确的继承某个内部类也可以哒，如`public class Yolk extends Egg2.Yolk{}`。



## 第十二章 集合

**Java. util**库提供的一套*集合类*，基本类型有**List、Set、Queue、Map**。Java 集合类都可以自动调整自己的大小。



### 泛型和类型安全的集合

**Java SE 5** 增加了参数类型，即泛型。标志是尖括号内加包含类型信息。

如果一个类没有显式的声明继承哪个类，那么它就自动继承自**Object**。

Java 7 之前，两端都进行类型声明，而之后 Java 接受了*类型推断*的请求，只需左侧声明。如`ArrayList<Apple> apples = new ArrayList();`即可。

使用泛型，从**List**中获取元素不需要强制类型转换，将 RTTI 变成了编译器错误（如果不使用，则需向上转型为**Object**存储，在取出元素时向下转型，放入元素时没有检查，这样可能会出现运行时错误）。

但是，泛型参数并不仅限于只能将确切类型的对象放入集合，向上转型也可以，如`Apple`的子类对象也可以放入。

类库的两个基本接口

* **集合（Collection）**：一个独立元素的序列，**List**（以特定顺序保存一组元素）、**Set**（元素不允许重复）、**Queue**（只能在一端插入并从另一端移除）; **ArrayList、LinkedList、HashSet**（检索元素最快）、**TreeSet**（将比较结果的升序保存对象）、**LinkedHashSet**（按被添加的先后顺序保存对象）。
* **映射（Map）**：一组成对的“键值对”对象，允许用键来查找值。**ArrayList**（使用数字查找对象）、**map**； **HashMap、TreeMap、LinkedHashMap**（在保持**HashMap**查找速度的同时按键的插入顺序保存键）

可以使用向上转型的方法，创建一个具体类的对象，将其向上转型为对应的接口，然后再其余代码中用这个接口。但是这样就放弃了具体类本身拥有，接口以外的的一些方法，如`Collection<Intger> c = new ArrayList<>();`。



### 添加元素组

在一个**Collection**中添加一组元素：

* `Arrays.asList()`：接受一个数组或者逗号分隔的元素列表（使用可变参数），并将其转换为**List**对象。
* `Collections.addAll()`：接受一个**Collection**对象，以及一个数组或是一个逗号分隔的列表，将其中元素添加到**Collection**中。该方法运行更快。
* `collection.addAll()`：只能接受另一个**Collection**作为参数，因此没有另外两个灵活。

```java
Collection<Integer> collection =
   new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
Integer[] moreInts = { 6, 7, 8, 9, 10 };
collection.addAll(Arrays.asList(moreInts));//Collection对象直接调用addAll（）
Collections.addAll(collection, moreInts); 
Collections.addAll(collection, 11, 12, 13, 14, 15);//更通用的addAll（）

List<Integer> list = Arrays.asList(16,17,18,19,20);//使用asList转换成list，底层实现是数组，没法调整大小。
list.set(1, 99); // OK -- 可修改元素
// list.add(21); // Runtime error; 不能改变大小
```

**Collection**的构造器可接受另一个**Collection**来初始化自己。因此可以使用`Arrays.asList()`来为这个构造器产生输入。

*显示类型参数说明*：

 `List<Snow> snow4 = Arrays.<Snow>asList(new Light(), new Heavy(), new Slush());`

告诉编译器`asList()`生成的结果**List**类型的实际目标类型是什么。



### 集合的打印

* **Collection**打印出的内容为[element,...]
* **Map**打印出的内容为{element A=content A，...}



### 列表List

**List**的两种类型：

* **ArrayList**，擅长随机访问元素，插入删除较慢

* **LinkedList**，插入删除操作较快，随机访问相对较慢。

**List**的各种操作：

`listName.remove()`:传入对象的引用或下标都可，`indexOf()`可查找下标。基于`equals()`

`listName.subList(int start,int end+1)`：创建切片**list**，结果回传源列表`containsAll()`结果为**true**。

`Collections.shuffle(subList,Random rand)`：*mix it up*

`Collections.sort(List sub)`：排序

`listName.retainAll(subList)`：集合交集，保留共有元素。基于`equals()`

`listName.removeAll(List sub)`:删除源链表中参数链表中的全部元素。基于`equals()`

`listName.toArray()`:重载方法可生成指定类型数组，无参版本返回一个**Object**数组，数组大小自动创建。



### 迭代器Iterators

*迭代器*（一种设计模式）实现抽象：

​		在一个序列中移动并选择该序列的每个对象，而客户端程序员不知道或不关心该序列的底层结构。

迭代器通常被称为*轻量级对象*：创建它的代价小。Java 的**Iterator**只能单向移动。

* `iterator()`方法要求集合返回一个**Iterator**。

* `next()`方法获得序列下一个元素。

* `hasNext()`方法检查序列中是否还有元素。

* `remove()`方法将迭代器最近返回的那个元素删除（若删除由`next()`生成的最后一个元素，则在调用`remove()`前必须先调用`next()`）。

**List/LinkedList/HashSet**的**Iterator**都是按存入顺序读出，**TreeSet**的**Iterator**按比较结果的升序读出。

`display()`方法的实现：传入集合的`iterator()`方法结果（`while(it.hasNext(){}`进行遍历）；或者使用`Iterable<Pet> ip`直接传入集合对象（集合实现了 Iterable 接口）（获取传入对象的Iterator对象后，继续用`while(it.hasNext(){}`进行遍历）。

**ListIterator**，**Iterator**子类型，只能由**List**类生成，可以双向移动。

* `nextIndex()`和`previousIndex()`:生成相对于迭代器在列表中指向的当前位置的后一个和前一个元素的索引。
* `hasPrevious()`：倒序判断。
* `set()`:替换它访问过最近的一个元素。
* `listName.listIterator(n)`：创建一个一开始就指向列表索引号为**n**的元素处的**ListIterator**。



### 链表LinkedList

添加了一些方法可以被作为**栈、队列、双端队列( Deque)**。

* `getFirst()`和`element()`返回列表头部，**List**为空时抛出异常，`peek()`则返回**null**。

* `removeFirst()`和`remove()`删除并返回头部元素，**List**为空时抛出异常，`poll()`返回**null**。

* `addFirst()`在列表头插入元素。

* `offer()`、`add()`、`addLast()`在列表尾添加一个元素。第一个是**Queue**相关的方法。

* `removeLast()`删除并返回列表最后一个元素。



### 堆栈Stack

Java 6 添加了**ArrayDeque**，其中包含了直接实现堆栈功能的方法即`push()`和`pop()`。

```java
Deque<String> stack = new ArrayDeque<>();
```

可使用**ArrayDeque**和泛型组合的方法重写**Stack**类。**Java.util**包中也有**Stack**，使用`java.util.Stack<String> stack = new java.util.stack<>()`指定包名，避免发生冲突。



### 集合Set

**Set**最常见的用途是测试归属性，通常会选择**HashSet**，该实现针对快速查找进行了优化。

实际上，**Set**就是一个**Collection**，只是行为不同（继承与多态的典型应用：表现不同的行为）。

* **HashSet** 使用散列函数存储数据，无序存储
* **TreeSet **将元素存储在红黑树数据结构中，按对象的排列升序存储

* **LinkedHashSet** 因为查询速度的原因也使用了散列，但是看起来使用了链表来维护元素的插入顺序。

**产生一个每个元素都唯一的列表用例：**

`java.nio.file.Files.readAllLines()` 方法，可以打开一个文件，并将其作为一个 **List\<String>** 读取，每个 **String** 都是输入文件中的一行。

```java
List<String> lines = Files.readAllLines(Paths.get("SetOperations.java"));
Set<String> words = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);//Comparator比较器
for(String line : lines)
  for(String word : line.split("\\W+"))
    if(word.trim().length() > 0)
      words.add(word);
System.out.println(words);
```



### 映射Map

**Map**和数组和其他的**Collection**一样，可扩展到多个维度，创建一个值为**Map**的**Map**即可。

* `containsKey()`和`containsValue()`判断是否含某个键。
* `keySet()`和`values()`生成所有键组成的**Set**。



### 队列Queue

先进先出集合，在**并发编程**中尤为重要，可以安全的将对象从一个任务传输到另一个任务。

**LinkedList**实现了**Queue**接口，可向上转换为**Queue**。



### 优先级队列PriorityQueue

*队列规则*除了先进先出的典型用例外，是指在给定队列中一组元素的情况下，确定下一个弹出队列的元素的规则。

调用 `offer()` 方法来插入一个对象时，该对象会在队列中被排序，默认的排序使用队列中对象的*自然顺序*。也可以通过提供自己的 **Comparator** 来修改队列中元素的插入顺序。**PriorityQueue** 确保在调用 `peek()` 返回头部， `poll()`弹出头部 或 `remove()`删除头部  方法时，获得的元素将是队列中优先级最高的元素。

**Integer**，**String**，**Character**内置了自然排序。

如果使用自己的类和**PriorityQueue**组合，则必须包含额外的功能以产生自然排序，或提供自己的**Comparator**。



### 集合与迭代器

**Collection**是所有序列集合共有的根接口。

标准 C++ 库中的集合没有共同的基类，集合之间的所有共性都是通过迭代器实现的。

在 Java 中，迭代器和**Collection**绑定销售，实现**Collection**就意味着提供`iterator()`方法。

在`display()`时，传参选择**Collection**要更方便一点，因为是**Iterable**类型，可以使用*for-in*构造，代码更清晰。

对于非**Collection**类，需要继承**AbstractCollection**，被强制实现`iterator()`和`size()`方法 。

或者，直接实现**Iterator**的提供，花费的代价更少。

```java
public Iterator<Pet> iterator() {
    return new Iterator<Pet>() {
      private int index = 0;
      @Override
      public boolean hasNext() {
        return index < pets.length;
      }
      @Override
      public Pet next() { return pets[index++]; }
      @Override
      public void remove() { // Not implemented，没用到的方法直接跳过，抛出异常
        throw new UnsupportedOperationException();
      }
    };
  }
```

总结：生成 **Iterator** 是将序列与消费该序列的方法连接在一起耦合度最小的方式，并且与实现 **Collection** 相比，它在序列类上所施加的约束也少得多。

### for-in和迭代器

使用 *for-in* 是所有**Collection**对象的特征。原因是 Java 5 引入了**Iterable**接口。

**Iterable**接口：包含一个能够生成**Iterator**的`iterator()`方法。*for-in*使用此**Iterable**接口来遍历序列（即默认获取该类的*Iterable*对象。

因此，所有实现了**Iterable**接口的类，都可以将其用于*for-in*语句，需要注意数组直接作为**Iterable**参数传递会失败，必须手动执行这种转换`Arrays.asList(strings)`(`strings`是一个`String[]`。

### 适配器方法惯用法

一个**Iterable**类，想在*for-in*语句中一个或多个使用这个类的方法，若直接继承这个类并覆盖`iterator()`方法，则只能替换而非添加。解决方案是*适配器方法*的惯用法。需要提供特定的接口满足*for-in*语句，如果有一个接口且需要另一个接口，就编写一个适配器。即添加一个能生成**Iterable**对象的方法，该对象可以用于*for-in*语句。

```java
class ReversibleArrayList<T> extends ArrayList<T> {
  ReversibleArrayList(Collection<T> c) {
    super(c);
  }
  public Iterable<T> reversed() {
    return new Iterable<T>() {
      public Iterator<T> iterator() {
        return new Iterator<T>() {
          int current = size() - 1;
          public boolean hasNext() {
            return current > -1;
          }
          public T next() { return get(current--); }
          public void remove() { // Not implemented
            throw new UnsupportedOperationException();
          }
        };
      }
    };
  }
}

 ReversibleArrayList<String> ral =
      new ReversibleArrayList<String>(
        Arrays.asList("To be or not to be".split(" ")));
    // Grabs the ordinary iterator via iterator():
    for(String s : ral)
      System.out.print(s + " ");
    System.out.println();
    // Hand it the Iterable of your choice
    for(String s : ral.reversed())
      System.out.print(s + " ");
```

要注意 `Arrays.asList()` 生成一个 **List** 对象，该对象使用**底层数组**作为其物理实现。如果执行的操作会修改这个 **List** ，并且不希望修改原始数组，那么就应该在另一个集合中创建一个副本。

### 小结

* 不要在新代码中使用遗留类**Vector，Hashtable**和**Stack**，用**LinkedList**。
* 集合不能保存基本类型，但自动装箱机制会负责执行基本类型和包装类型之间的双向转换。
* 大量随机访问用**ArrayList**，大量删除插入用**LindedList**。
* **HashSet** 提供最快的查询速度，而 **TreeSet** 保持元素处于排序状态。 **LinkedHashSet** 按插入顺序保存其元素，但使用散列提供快速访问的能力。

**Java 集合简图**

![simple collection taxonomy](E:/md/gitOnline/OnJava8/docs/images/simple-collection-taxonomy.png)

可以看到，实际上只有四个基本的集合组件： **Map** ， **List** ， **Set** 和 **Queue** ，它们各有两到三个实现版本（**Queue** 的 **java.util.concurrent** 实现未包含在此图中）。最常使用的集合用黑色粗线线框表示。

虚线框表示接口，实线框表示普通的（具体的）类。带有空心箭头的虚线表示特定的类实现了一个接口。实心箭头表示某个类可以生成箭头指向的类的对象。例如，任何 **Collection** 都可以生成 **Iterator** ， **List** 可以生成 **ListIterator** （也能生成普通的 **Iterator** ，因为 **List** 继承自 **Collection** ）。

下面是译者绘制的 Java 集合框架简图，黄色为接口，绿色为抽象类，蓝色为具体类。虚线箭头表示实现关系，实线箭头表示继承关系。



![collection](E:/md/gitOnline/OnJava8/docs/images/collection.png)
![map](E:/md/gitOnline/OnJava8/docs/images/map.png)



---------------



## 第十三章  **函数式编程**

> 函数式编程语言操纵代码片段就像操作数据一样容易。 虽然 Java 不是函数式语言，但 Java 8 Lambda 表达式和方法引用 (Method References) 允许你以函数式编程。

**自修改代码**：程序员通过修改内存中的代码来节省代码空间，以便程序执行时执行不同的操作（棘手而神秘的汇编代码）。

**函数式编程（FP）**：通过合并现有代码来生成新功能而不是从头开始编写所有内容——通过代码以某种方式操纵其他代码。为“并行编程”、“代码可靠性”、“代码创建和库复用”。

面向对象是抽象数据，函数式编程是抽象行为。

**如何传递功能**：（通常传递给方法的数据不同，结果不同，那么如果希望方法调用时行为不同，则需要**将代码传递给方法**来控制它的行为）

* **[1]**传统方法，定义一个类，实现一个接口，复写函数体。
* **[2]**创建匿名内部类`new Strategy(){}`
* **[3]**Java 8 的 Lambda 表达式：`(String) msg->msg.substring(0,5);`，箭头左侧是参数，右侧是从 Lambda 返回的表达式，即函数体。
* **[4]**Java 8 的方法引用：`ClassName::FunctionName`，`::`左侧是类或对象的名称，右边是方法名称但没有参数列表。这个语法来自 C++ 。

### Lambda表达式

----------------

* 产生函数，而不是类。尽管 JVM 上一切都是一个类，但在幕后各种操作使之看起来像函数。
* 语法尽可能少。基本语法是：
  1. 参数，当自用一个参数可以不需要`()`；正常情况用`()`包裹参数（列表）。
  2. `->`，可视为“产出”。
  3. `->`之后的内容都是方法体，若是单行，则该表达式是结果自动成为 Lambda 表达式的返回值，此时使用  **return** 是非法的；如果是多行则用`{}`，并使用 **return** 。

#### Lambada和递归

递归的 Lambda 表达式，递归方法必须是实例变量或者静态变量（ static ）。

**使用方法**：

1. 通常对包含一个方法的接口，创建一个静态接口对象`static IntCall target`，对此对象使用 Lambda 表达式`target = n->n==0 ? 1: n * target.func(n-1)`，调用此对象`.func()`。
2. 同 1 ，创建接口对象` IntCall target`，在对象所在外围类的构造函数中实现 Lambda表达式`fib = n -> n == 0 ? 0 : n == 1 ? 1 : target.func(n - 1) + target.func(n - 2);`，并创建外围类实例变量`rf`，调用` rf.target.func`



### 方法引用

----------------------------

* 基本语法是：

  类名或对象名，后跟`::`，然后跟方法名称。

* 以单一方法接口开始，只要

  1. **方法引用的方法**的签名（参数类型和返回类型）符合该接口的方法的签名
  2. 对已实例化对象的方法进行绑定引用（有时称为*绑定方法引用*），或引用**静态方法**。
  3. 用法为 `c = StaticClassName::func2()`，然后调用`c`本身的方法`c.call()`。

#### Runnable 接口：方法引用和Lambda实现

 **Thread** 对象将 **Runnable** 作为其构造函数参数，并具有会调用 `run()` 的方法  `start()`。 **注意**，只有**匿名内部类**才需要具有名为 `run()` 的方法。

```java
class Go {
  static void go() {
    System.out.println("Go::go()");
  }
}

new Thread(new Runnable() { //匿名内部类的方法
      public void run() {
        System.out.println("Anonymous");
      }
    }).start();

    new Thread(  // lambda 表达式的方法
      () -> System.out.println("lambda")
    ).start();

    new Thread(Go::go).start(); //方法引用
```

#### 未绑定的方法引用

未绑定的方法引用值没有关联对象的普通（非静态）方法。**使用前，必须先提供对象**[^1]。也就是说，这是另一种形式的方法引用，使用未绑定的引用时，函数方法的签名（接口中的单个方法）不再与方法引用的签名完全匹配。 理由是：你需要一个对象来调用方法。那么在函数方法的参数中必须要留出来**第一个位置**给`this`，并在其上调用方法。

#### 构造函数引用

构造函数也可以被捕获，然后通过引用调用它~

```java
MakeNoArgs mna = Dog::new;
Make1Arg m1a = Dog::new;
Dog dn = mna.make();
Dog d1 = m1a.make("Comet");//构造函数几个参数都可以。
```



### 函数式接口

----------

方法引用和 Lambda 表达式必须被赋值，且编译器需要识别类型信息以确保类型正确。虽然 Lambda 表达式自带类型推导，但是对于多参数编译器如何确定正确类型呢？

Java 8 引入 `java.util.function`包，它包含一组接口，这些接口是 Lambda 表达式和方法引用的目标类型。 每个接口只包含一个抽象方法，称为函数式方法。这个包的目的在创建一组完整的目标接口，使得我们一般情况下不需要再定义自己的接口。

编写接口时，可以使用 `@FunctionalInterface` 注解强制执行此“函数式方法”模式：接口中如果有多个方法则会产生编译时错误消息。

> 这里左侧是接口，右侧是方法， Java 8 在这里做的手脚： 如果将方法引用或 Lambda 表达式赋值给函数式接口（类型需要匹配）， Java 会适配你的赋值到目标接口。编译器会自动包装方法引用或 Lambda表达式到实现目标接口的类的实例中。
>
> 使用函数接口时，名称无关紧要——只要参数类型和返回类型相同， Java 将程序员的方法映射到接口方法，要调用方法，则调用接口的函数式方法名即可。
>
> Java 8 允许以简便的语法为接口赋值函数，但是直接接口对象 = 某类对象的操作依旧不可行！

**多参数函数式接口**

觉得`java.util.function`中的接口不够用，可以自创接口，前面加上`@FunctionalInterface`。

**基本类型的函数式方法**

注意参数传递的是包装类还是基本类型，基本数据类型可以进行自动拆装箱和类型转换，但是包装类之间不能自动类型转换（**Integer**无法转换为**Double**）。



###  高阶函数

-----------------

一个消费或产生函数的函数。

**1.**  产生函数

* 继承一个接口，为专用接口创建了一个别名，如`interface FuncSS extends Function<String, String> {}`。
* 使用 Lambda 表达式，在方法中创建和返回一个函数，`static FuncSS produce() {return s -> s.toLowerCase();}`。这里`produce()`是高阶函数。

**2.** 消费函数

* 在函数的参数中传入函数类型。

```java
static Two consume(Function<One,Two> onetwo) {
    return onetwo.apply(new One()); //传入参数new One（）
  }
  public static void main(String[] args) {
    Two two = consume(one -> new Two()); //传入参数one，函数体（返回值）为 new Two（）
  }
```

* 或者，基于消费函数生成新函数。

这个用例非常有趣，有趣到脑壳疼。

```java
class I {
  @Override
  public String toString() { return "I"; }
}

class O {
  @Override
  public String toString() { return "O"; }
}

public class TransformFunction {
  static Function<I,O> transform(Function<I,O> in) {
    return in.andThen(o -> {
      System.out.println(o);
      return o;
    });
  }
  public static void main(String[] args) {
    Function<I,O> f2 = transform(i -> {
      System.out.println(i);
      return new O();
    });
    O o = f2.apply(new I());
  }
}
```

 `transform()` 产生的是一个新函数，它将 `in` 的动作与 `andThen()` 参数的动作结合起来，这是一个**函数组合**用法。



### 闭包

--------

Java 8 提供了有限但合理的闭包支持。

从 Lambda 表达式引用的**局部变量**必须是 `final` 或者是**等同 `final` **效果（虽然没有明确声明变量为`final`，但是因变量初始值没改变过而有了等同效果，这是在 Java 8 才放宽的规则）的。即使在 return 前先“++”，也会被编译器识别出来，从而报错。

但是，应用于对象引用的 `final` 关键字仅表示不会重新赋值引用。 它并不代表你不能修改对象本身。比如`ArrayList`声明了`final`后，也可以增删元素！

从 Lambda 表达式引用的外围类成员，没有`final`要求。

总结来说就是要考虑**捕获的变量**是否是**等同 final 效果**，如果是对象中的字段，那么它拥有独立的生存周期，也不需要任何特殊捕获，当然就没有`final`要求。

#### 函数组合

```java
import java.util.function.*;
import java.util.stream.*;

public class PredicateComposition {
  static Predicate<String>
    p1 = s -> s.contains("bar"),
    p2 = s -> s.length() < 5,
    p3 = s -> s.contains("foo"),
    p4 = p1.negate().and(p2).or(p3);
  public static void main(String[] args) {
    Stream.of("bar", "foobar", "foobaz", "fongopuckey")
      .filter(p4)
      .forEach(System.out::println);
  }
}
```

`p4` 获取到了所有断言并组合成一个更复杂的断言。解读：如果字符串中不包含 `bar` 且长度小于 5，或者它包含 `foo` ，则结果为 `true`。

#### 柯里化和部分求值

 柯里化意为：将一个多参数的函数，转换为一系列单参数函数。

`Function<String, Function<String, String>> sum = a -> b -> a + b; `

也可以多级化。

### 本章小结

---------

Lanbda 表达式和方法引用提供了对函数式编程的支持，流式编程更容易了。

没有泛型 Lambda，所以 Lambda 在 Java 中并非一等公民哈哈。

**第十三章脚注：**

[^1]:想方设法不提供关联对象，进行方法引用，通过了编译，还是用不了。

----------

## 第十四章 流式编程

> 集合优化了对象的存储，而流和对象的处理有关。

