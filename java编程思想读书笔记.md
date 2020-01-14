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

[^1]: 想方设法不提供关联对象，进行方法引用，通过了编译，还是用不了。

----------

## 第十四章 流式编程

> 集合优化了对象的存储，而流和对象的处理有关。

**流编程示例**：

`new Random(47).ints(5, 20).distinct().limit(7).sorted().forEach(System.out::println);`

流式编程采用*内部迭代*，使得编写的代码可读性更强，也更能利用多核处理器的优势。

*外部迭代*：显式的编写迭代机制称为外部迭代。

流是懒加载的，即只在绝对必要时才计算。由于计算延迟，它能回我们表示大大大序列而不用考虑内存问题。

**Java 8 融入流的解决方案**：

在接口中添加被`default`修饰的方法。流式平滑的被嵌入到现有类中。

流操作类型有三种：创建流 、中间操作和终端操作

### 创建流

----------------

* 通过`Stream.of()`将一组元素转化为流。

  ```java
  Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!")
      .forEach(System.out::print);
  ```

* 每个集合可通过调用`stream()`产生一个流。**Map**集合通过调用`entrySet().stream()`产生流。

* 注意**Random**类只能生成基本类型，`rand.ints(((int size),(int low,int high))).boxed()`流操作会做装箱类型，从而使得`show(Stream<T> stream)`能够接受流。

* 使用 **Random** 为任意对象集合创建 **Supplier**。如用文本文件提供字符串对象。 **Stream.**`generate(Supplier<T> supplier)` ，可以接收参数  `Supplier<T>`，将其生成 `T` 类型的流。

* `IntStream` 类提供了  `range()` 方法用于生成整型序列的流。

  1. 编写循环求和时，可用`for(int i : IntStream.range(10,20).toArray())`，或者直接调用流的`sum()`求和。
  2. 利用`range( 0 , n ) . forEach( i -> action.run())`（ action 是 Runnable 对象，可通过  lambda 表达式实现单方法接口函数引用）实现小功能 `repeat()`来替代`for` 循环。

* **Stream.**`iterate()`两个参数：种子，方法。方法的结果添加到流，并存储作为种子供下次调用。

* **Stream.**`builder()`，创建`builder`对象，可以`add(T t)`，调用`builder.build()`后，构造结束。

* `Arrays`类中有一个`stream()`静态方法将**数组**转换为流。还可传入三个参数，数组，开始下标，结束下标，来传入子数组。

* 正则表达式中的流：**Pattern.**` splitAsStream(String all)`，输入只能是 **CharSequence** ，返回`Stream<String>`。

### 修改流元素（中间操作）

-------------

从一个流中获取对象，并将对象作为另一个流从后端输出，以连接到其他操作。

* 跟踪和调试，`peek(System.out::print)`(无返回值的 **Consumer** 函数式接口）允许无修改的查看流中的代码。

* 流元素排序，`sorted(Comparator.reverseOrder())`，传入**Comparator** 参数，也可以无参实现，也可以传入 Lambda 函数。

* 移除元素，

  1. `Random.java`中的`distinct()`，消除流中的重复元素。（比创建 **Set** 省事。
  2. `Stream.filter(Predicate`)，保留与传进去的过滤器函数计算结果为 **true** 的元素。

* 应用函数到元素，

  1. `map(Function)`：将函数操作应用在输入流的元素中，并将返回值传递到输出流中。
  2. `mapToInt(ToIntFunction)`：操作同上，但结果是 **IntStream**。
  3. `mapToLong(ToLongFunction)`：操作同上，但结果是 **LongStream**。
  4. `mapToDouble(ToDoubleFunction)`：操作同上，但结果是 **DoubleStream**。

  注意如果 **Function** 返回结果是数值类型的一种，则必须用`mapTo数值类型`代替。

* 想要产生元素流而非元素流的流，即将产生流的函数应用在每个元素上，然后将每个流都扁平化为元素。

  1. `flatMap(Function)`：当 `Function` 产生流时使用。

  2. `flatMapToInt(Function)`：当 `Function` 产生 `IntStream` 时使用。

  3. `flatMapToLong(Function)`：当 `Function` 产生 `LongStream` 时使用。

  4. `flatMapToDouble(Function)`：当 `Function` 产生 `DoubleStream` 时使用。

### Optional类

------------

作为流元素的持有者，即使查看的元素不存在也能友好的提示我们， **Optional** 类`Optional<String> optString`！此外，创建空流`Stream.<String>empty()` ，在等号左边说明类型信息，编译器则可以在调用时推断类型。

* 获取**Optional** 类：

  1. `findFirst()` 返回一个包含第一个元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**
  2. `findAny()` 返回包含任意元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**
  3. `max()` 和 `min()` 返回一个包含最大值或者最小值的 **Optional** 对象，如果流为空则返回 **Optional.empty**
  4. `reduce()` 不再以 `identity` 形式开头，而是将其返回值包装在 **Optional** 中。（不存在空结果的风险）

* 对`Optional`的**解包函数**：

  1. `ifPresent(Consumer)`：当值存在时调用 **Consumer**，否则什么也不做。
  2. `orElse(otherObject)`：如果值存在则直接返回，否则生成 **otherObject**。
  3. `orElseGet(Supplier)`：如果值存在则直接返回，否则使用 **Supplier** 函数生成一个可替代对象。
  4. `orElseThrow(Supplier)`：如果值存在直接返回，否则使用 **Supplier** 函数生成一个异常。

* 自己的代码中加入 **Optional** 类，使用下面三个静态方法：

  1. `empty()`：生成一个空 **Optional**。
  2. `of(value)`：将一个非空值包装到 **Optional** 里。
  3. `ofNullable(value)`：针对一个可能为空的值，为空时自动生成 **Optional.empty**，否则将值包装在 **Optional** 中。

  不能通过传递 `null` 到 `of()` 来创建 `Optional` 对象。最安全的方法是， 使用 `ofNullable()` 来优雅地处理 `null`。

* **Optional** 对象操作

  1. `filter(Predicate)`：将 **Predicate** 应用于 **Optional** 中的内容并返回结果。当 **Optional** 不满足 **Predicate** 时返回空。如果 **Optional** 为空，则直接返回。
  2. `map(Function)`：如果 **Optional** 不为空，应用 **Function**  于 **Optional** 中的内容，并返回结果。否则直接返回 **Optional.empty**。
  3. `flatMap(Function)`：同 `map()`，但是提供的映射函数将结果包装在 **Optional** 对象中，因此 `flatMap()` 不会在最后进行任何包装。

  以上方法都不适用于数值型 **Optional**。一般来说，流的 `filter()` 会在 **Predicate** 返回 `false` 时移除流元素。而 `Optional.filter()` 在失败时不会删除 **Optional**，而是将其保留下来，并转化为空（一直 print 最后一行是“ Optional . empty” 。

* 假设你的生成器可能产生 `null` 值，那么当用它来创建流时，可以用  **Optional** 来**包装元素**。解包元素时，调用`stream.map(Optional::get)`。



###  消费流元素（终端操作）

---------------------

获取流的最终结果：

* 数组

  1. `toArray()`：将流转换成适当类型的数组。
  2. `toArray(generator)`：在特殊情况下，生成自定义类型的数组。

* 循环

  1. `forEach(Consumer)`常见如 `System.out::println` 作为 **Consumer** 函数。无序操作，仅在引入并行流时才有意义。
  2. `forEachOrdered(Consumer)`： 保证 `forEach` 按照原始流顺序操作。

   `parallel()` ，多处理器并行操作。

* 集合

  1. `collect(Collector)`：使用 **Collector** 收集流元素到结果集合中。`.collect(Collectors.toCollection(TreeSet::new))`，返回`Set<String>`

  2. `collect(Supplier, BiConsumer, BiConsumer)`：同上，第一个参数 **Supplier** 创建了一个新结果集合，第二个参数 **BiConsumer** 将下一个元素包含到结果中，第三个参数 **BiConsumer** 用于将两个值组合起来。

`.collect(Collectors.toMap(Pair::getI, Pair::getC));`，从一个`Pair`的流里生成新的对象流，**组合多个流以生成新的对象流的唯一方法**。

* 组合

  1. `reduce(BinaryOperator)`：使用 **BinaryOperator** 来组合所有流中的元素。因为流可能为空，其返回值为 **Optional**。
  2. `reduce(identity, BinaryOperator)`：功能同上，但是使用 **identity** 作为其组合的初始值。因此如果流为空，**identity** 就是结果。
  3. `reduce(identity, BiFunction, BinaryOperator)`：更复杂的使用形式（暂不介绍），这里把它包含在内，因为它可以提高效率。通常，我们可以显式地组合 `map()` 和 `reduce()` 来更简单的表达它。

* 匹配

  1. `allMatch(Predicate)` ：如果流的每个元素根据提供的 **Predicate** 都返回 true 时，结果返回为 true。在第一个 false 时，则停止执行计算。
  2. `anyMatch(Predicate)`：如果流中的任意一个元素根据提供的 **Predicate** 返回 true 时，结果返回为 true。在第一个 false 是停止执行计算。
  3. `noneMatch(Predicate)`：如果流的每个元素根据提供的 **Predicate** 都返回 false 时，结果返回为 true。在第一个 true 时停止执行计算。

  `BiPredicate<Stream<Integer>, Predicate<Integer>>`，一个二元断言，接受两个参数，测试的流和断言 **Predicate** ，适用于所有的 **Stream::Match**。

* 查找

  1. `findFirst()`：返回第一个流元素的 **Optional**，如果流为空返回 **Optional.empty**。

  2. `findAny()`：返回含有任意流元素的 **Optional**，如果流为空返回 **Optional.empty**。

* 信息

  1. `count()`：流中的元素个数。
  2. `max(Comparator)`：根据所传入的 **Comparator** 所决定的“最大”元素。
  3. `min(Comparator)`：根据所传入的 **Comparator** 所决定的“最小”元素。

* 数字流信息

  1. `average()` ：求取流元素平均值。
  2. `max()` 和 `min()`：数值流操作无需 **Comparator**。
  3. `sum()`：对所有流元素进行求和。
  4. `summaryStatistics()`：生成可能有用的数据。目前并不太清楚这个方法存在的必要性，因为我们其实可以用更直接的方法获得需要的数据。

  

## 第十五章 异常

  错误恢复在 Java 中格外重要—— Java 的主要目标之一就是创建供他人所使用的程序构件。Java 使用异常来提供一致的错误报告模型，**异常处理**也是 Java 中唯一官方的错误报告机制。通过强制规定的形式来消除错误处理过程中随心所欲的因素。

  *异常情形*是指阻止当前方法或作用域继续执行的问题，解决方法是从当前环境跳出并把问题提交给上一级环境。

  1. 使用 new 在堆上创建异常对象，伴随着存储空间分配和构造器的调用。所有的标准异常类的两个构造器：无参构造器和接受字符串作为参数（以便把相关信息放入异常对象）的构造器。
  2. 然后当前的执行路径被终止，并从当前环境中弹出（ **throw** ）对异常对象的引用。**异常最重要的部分就是类名**。
  3. 异常处理机制接管程序，搜寻参数与异常类型相匹配的第一个处理程序，然后运行异常处理程序。以关键字 **catch** 表示。

  *监控区域*是一段可能产生异常的代码，后面跟着处理这些异常的代码。

  异常处理理论的两种模型，终止模型（ Java 和C++ 所支持）和恢复模型（不实用，耦合度高：恢复性的处理程序需要了解异常抛出的地点，则势必要包含依赖于抛出位置的非通用性代码）。

  ### 异常机制用法：自定义和声明

---------------

* 自定义异常类，必须从已有的异常类继承，最好选择意思相近的异常类代码，最简单的方法是编译器产生无参构造器。

* 输出：

  **Exception** 从 **Throwable** 类继承。因此异常对象可调用 Throwable 类声明的 `printStackTrace()`方法，无参则信息被输出到标准错误流 System.err ；传入参数`System.out`则被输出。异常类的打印方法**getMessage（）**，有点类似于 **toString（）**。

* 记录日志：

  使用 java.util.logging 工具将输出记录到日志中。

  ```java
   private static Logger logger =
              Logger.getLogger("LoggingException");//创建与String相关联的 Logger对象
      LoggingException() {
          StringWriter trace = new StringWriter();
          printStackTrace(new PrintWriter(trace));//传入java.io.StringWriter对象给
          //java.io.PrintWriter对象作为参数。通过调用 toString()，将输出抽取为一个String
          logger.severe(trace.toString()); //向Logger写入的方式之一：直接调
         									 //用与日志记录消息的级别关联的方法
      }
  ```

* ***异常声明***：

  ```java
  void f() throws TooBig, TooSmall, DivZero { // ...
  ```

  代码必须与异常说明保持一致。当然也可以先声明抛出异常，实际上却不抛出，为异常先占个位子。在定义抽象基类和接口时这种能力很重要，这样派生类或接口实现就能够抛出这些预先声明的异常。

* 捕获所有异常：

  `catch(Exception e)`，最好放在处理程序列表的末尾，以防它在其他处理程序前把异常给捕获了。

  Exception 是与编程有关的所有异常类的基类，所以不含太多具体信息。其基类是 **Throwable**，有`getMessage();getLocallizedMessage();`获取详细信息；`toString();`返回简单描述；`printStackTrace();printStackTrace(PrintStream); printStackTrace(java.io.PrintWriter)`打印 Throwable 和 Throwable 的调用栈轨迹。

  匹配时不要求抛出的异常和程序声明的异常完全匹配，派生类的对象也可以匹配其基类的处理程序。

* **多重捕获：**

  一组具有相同基类的异常，可以直接 catch 其基类型对其进行捕获。

  如果没有相同基类型，Java 7 后可以`catch(Exception1|Exception2|Exception3 e)`。而之前只能逐个 catch 。

* 栈轨迹

  `printStackTrace() `方法所提供的信息可以通过` getStackTrace()` 方法来直接访问，这个方法将返回一个由栈轨迹中的元素( **StackTraceElement** )所构成的数组，其中每一个元素都表示栈中的一桢。元素 0 是栈顶元素，并且是调用序列中的最后一个方法调用（这个 Throwable 被创建和抛出之处）。数组中的最后一个元素和栈底是调用序列中的第一个方法调用（可通过`getMethodName()`获取方法名）。

* 重新抛出异常

  catch 到一个 Exception 后，可以继续 throw 出去。

  会将异常抛给上一级环境中的异常处理程序，同一个 try 块后续的 catch 子句会被忽略。

  抛出当前异常对象，想要更新调用栈信息，需要调用`fillnStackTrace()`方法，这将返回一个 Throwable 对象，通过将当前调用栈信息填入原来那个异常对象而建立。

  Java 7 之前，捕获到什么类型，丢出去什么类型。Java 7 修复了这个问题。

* **异常链**

  所有 Throwable 的子类在构造器中都可以接受一个 cause 对象作为参数。只有三种基本的异常类提供了带 cause 参数的构造器。它们是 Error（用于 Java 虚拟机报告系统错误）、Exception 以及 RuntimeException。如果要把其他类型的异常链接起来，应该使用 initCause0 方法而不是构造器。

  在一个普通方法里调用别的方法时，要考虑到“我不知道该这样处理这个异常，但是也不想把它‘吞’了，或若打印一些无用的消息”。异常链提供了一种新的思路来解决这个问题。可以直接把“被检查的异常”包装进 RuntimeException 里面。

### Java 标准异常

---------

Throwable 这个 Java 类被用来表示任何可以作为异常被抛出的类。Throwable 对象可分为两种类型（指从 Throwable 继承而得到的类型）：

1. Error 用来表示编译时和系统错误（除特殊情况外，一般不用你关心）；
2. Exception 是可以被抛出的基本类型，在 Java 类库、用户方法以及运行时故障中都可能抛出 Exception 型异常。

异常并非全是在 java.lang 包里定义的；有些异常是用来支持其他像 util、net 和 io 这样的程序包，这些异常可以通过它们的完整名称或者从它们的父类中看出端倪。

**特例：RuntimeException**：运行时异常的类型很多，自动被 JVM 抛出，都是从 RuntimeException 类继承而来，也被称为“不受检查异常”，代表的是编程错误。如果 RuntimeException 没有被捕获而直达 main()，那么**在程序退出前将调用异常的 printStackTrace() 方法**。

### finally

---------

无论异常是否被抛出， finally 子块都被执行。那么

1. 如果把 try 块放入循环内，就建立了一个“程序继续执行前必须要达到”的条件。加入 static 类型计数器，使循环必能尝试一定次数。这样就**加强了程序的鲁棒性**。
2. 对于没有垃圾回收和析构函数自动调用机制的语言来说，finally 保证了内存总得到释放。在 Java 中，finally 可以使除内存以外的资源回复到它们的初始状态，如打开的文件或网络连接等。
3. finally 和带标签的 break continue 配合使用，替代 goto 语句。
4. 何处 return 都不会影响 finally 子句的执行。
5. finally 有可能提前 return 或者抛出新异常，导致异常不被处理或被覆盖。

### 构造器和异常

--------

* 异常限制对构造器不起作用。派生类构造器不能捕获基类构造器抛出的异常。

* *异常限制*：当覆盖方法的时候，只能抛出在基类方法的异常说明里列出的那些异常。尽管在继承过程中编译器会对异常说明做强制要求，异常说明本身不属于方法类型（方法名和参数类型组成）的一部分，因此不能基于异常说明来重载方法。

* 一个出现在基类方法的异常说明中的异常，不一定会出现在派生类方法的异常说明里。

* **构造器的异常处理：**

  构造器内可能抛出异常，并要求清理的类，最安全的方法是使用嵌套 try 子句。基本规则是：在创建需要清理的对象之后，立即进入一个 try-finally 语句块。

### try-with-resources 语法

-----------

Java 7 开始，try后可以跟一个带括号的定义——括号内的部分称为资源规范头。这部分内创建的对象**必须实现**  **java.lang.AutoCloseable**  接口，这个接口有一个方法：`close()`。无论如何退出 try 块，都会执行资源清理工作。

 Java 5 的 Closeable 已被修改，继承了 AutoCloseable 接口。所以所有实现了 Closeable 接口的对象，都支持了  try-with-resources 特性。

* 资源规范头中可以包含多个定义，并且通过分号进行分割（最后一个分号是可选的）。
* 退出 try 块会调用资源的 `close()`方法，并以创建顺序相反的顺序关闭他们。
* 资源规范头中抛出异常，会被 catch 子句捕获。这部分实际上被try 块包围。

### 小结

* 异常处理的重要原则——只有在你知道如何处理的情况下才捕获异常。否则会导致无意间“吞食”异常。

* Java 被检查的异常一方面为程序“健壮性”提供了支撑，另一方面可能导致“欺骗程序”的出现。

  > 好的程序设计语言能帮助程序员写出好程序，但无论哪种语言都避免不了程序员用它写出坏程序。

* 应该在下列情况下使用异常：

  1. 尽可能使用 try-with-resource。
  2. 在恰当的级别处理问题。（在知道该如何处理的情况下才捕获异常。）
  3. 解决问题并且重新调用产生异常的方法。
  4. 进行少许修补，然后绕过异常发生的地方继续执行。
  5. 用别的数据进行计算，以代替方法预计会返回的值。
  6. 把当前运行环境下能做的事情尽量做完，然后把相同的异常重抛到更高层。
  7. 把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到更高层。
  8. 终止程序。
  9. 进行简化。（如果你的异常模式使问题变得太复杂，那用起来会非常痛苦也很烦人。）
  10. 让类库和程序更安全。（这既是在为调试做短期投资，也是在为程序的健壮性做长期投资。）

* 报告功能是异常的精髓。





## 第十七章 文件 

Java 7 对 I/O 设计的新改进，放在 **java.nio.file** 包， **non-blocking** 非阻塞  **io**。

文件操作的两个基本组件：

* 文件或者目录的路径
* 文件本身

### Path类

----------

**java.nio.file.Paths** 类包含重载方法 **static get()** ，接受一系列 **String** 字符串或一个 *统一资源标识符（URL）*作为参数，返回一个 **Path** 对象。一个 **Path** 对象表示一个文件或者目录的路径，是一个跨 OS 和文件系统的抽象，目的是在构造路径时不必关注底层操作，代码可在不修改情况下运行在不同的 OS 上。

**Files** 工具类包含了大部分我们需要的目录操作和文件操作方法。且具有目录树相关的方法。

**Path** 类的用法：

```java
 static void info(Path p) {
        System.out.println(System.getProperty("os.name"));//展示操作系统名字
     	////Path 类的操作
     	int i = p.getNameCount();//路径对象分段
     	p.endsWith(".java");//判断是否以字符串结尾
     	Path ap = p.toAbsolutePath();//变成绝对路径
     	Path rp = p.toRealPath();//返回实际情况下的Path
     	show("toString", p); //直接打印，结果为一个url
     	show("Absolute", p.isAbsolute());
        show("FileName", p.getFileName());//获取文件名
        show("Parent", p.getParent());//获取上一级文件夹名
        show("Root", p.getRoot());//获取根目录
     	URI u = p.toUri();//转为 url 对象
     
     	////File 类对 Path 的操作
        show("Exists", Files.exists(p));//是否存在
        show("RegularFile", Files.isRegularFile(p));//是否为文件
        show("Directory", Files.isDirectory(p));//是否为文件夹
        say("Executable", Files.isExecutable(p));
        say("Readable", Files.isReadable(p));
        say("Writable", Files.isWritable(p));
        say("notExists", Files.notExists(p));
        say("Hidden", Files.isHidden(p));
        say("size", Files.size(p));
        say("FileStore", Files.getFileStore(p));
        say("LastModified: ", Files.getLastModifiedTime(p));
        say("Owner", Files.getOwner(p));
        say("ContentType", Files.probeContentType(p));
        say("SymbolicLink", Files.isSymbolicLink(p));
        if(Files.isSymbolicLink(p))
            say("SymbolicLink", Files.readSymbolicLink(p));
        if(FileSystems.getDefault().supportedFileAttributeViews().contains("posix"))
            say("PosixFilePermissions", //需要先确认当前文件系统是否支持Posix 接口。
        Files.getPosixFilePermissions(p));
    }
```

```java
	//Paths 的增删操作
	pA.relativize(Path pB)// pB 相对于 pA 的路径，可达到去根目录的效果
	p.resolve(String/Path)//在 Path 后添加一个尾路径
	p.normalize()// ？
    p.resolveSibling(String/Path)//替换 Path 最后一个文件名
    Paths.get("")//获取当前文件所在文件夹
    Paths.get(".")//获取当前文件所在文件夹/.
```

### 目录树、文件系统和路径监控

----------

**Files** 工具类的目录树相关方法：

* 删除目录树。删除目录树的方法实现依赖于 **Files.walkFileTree(dir, new SimpleFileVisitor<Path>() {})**。*Visitor* 设计模式提供了一种标准机制来访问集合中的每个对象，然后你需要提供在每个对象上执行的操作。此操作的定义取决于实现的 **FileVisitor** 的四个抽象方法，包括：

  ```
  1.  **preVisitDirectory()**：在访问目录中条目之前在目录上运行。 
  2.  **visitFile()**：运行目录中的每一个文件。  
  3.  **visitFileFailed()**：调用无法访问的文件。   
  4.  **postVisitDirectory()**：在访问目录中条目之后在目录上运行，包括所有的子目录。
  ```

* 要获取目录树的全部内容的流，使用 **Files.walk()**。

* ```java
  	Files.copy(Paths A,Paths B)//复制
      Path tempdir = Files.createTempDirectory(Path p, "DIR_");//创建以DIR_命名开头的临时文件夹
      Files.createTempFile(tempdir, "pre", ".non");//创建以pre开头，.non 结尾命名的临时文件，可传参null，默认后缀.tmp
  ```

**查找文件系统相关的一些信息：**

* **FileSystems** 工具类获取“默认”的文件系统：`FileSystem fsys = FileSystems.getDefault()`。 

  ```java
  for(FileStore fs : fsys.getFileStores())
              show("File Store", fs);
          for(Path rd : fsys.getRootDirectories())
              show("Root Directory", rd);
          show("Separator", fsys.getSeparator());
          show("UserPrincipalLookupService",
              fsys.getUserPrincipalLookupService());
          show("isOpen", fsys.isOpen());
          show("isReadOnly", fsys.isReadOnly());
          show("FileSystemProvider", fsys.provider());
          show("File Attribute Views",
          fsys.supportedFileAttributeViews());
  ```

  一个 **FileSystem** 对象也能生成 **WatchService** 和 **PathMatcher** 对象。

* 在 **Path** 对象上调用 **getFileSystem（）** 以获取创建该 **Path **的文件系统

* 获得给定 URL 的文件系统

**路径监听**：

通过 **WatchService** 可以设置一个进程对目录中的更改做出响应。

```java
		WatchService watcher = FileSystems.getDefault().newWatchService();//创建对象
		test.register(watcher, ENTRY_DELETE);//对Path test进行注册
        Executors.newSingleThreadScheduledExecutor()
        .schedule(PathWatcher::delTxtFiles,//并行运行设置PathWatcher类的delTxtFiles方法。可替换为										//submit()并传参为线程。
        250, TimeUnit.MILLISECONDS); //并设置运行前应该等待的时间
        WatchKey key = watcher.take();//watcher.take()等待并阻塞在这里
        for(WatchEvent evt : key.pollEvents()) {
            System.out.println("evt.context(): " + evt.context() +
            "\nevt.count(): " + evt.count() +
            "\nevt.kind(): " + evt.kind());
            System.exit(0);
        }

```

从 **FileSystem** 中得到了 **WatchService** 对象，我们将其注册到 **test** 路径以及我们感兴趣的项目的变量参数列表中，可以选择 **ENTRY_CREATE**，**ENTRY_DELETE** 或 **ENTRY_MODIFY**(其中创建和删除不属于修改)。

 **WatchService** 对象只对单级文件夹内的文件进行监控，不深入下一级内。

### 文件查找和文件读写

-------------

找到文件的方法：

* 在 `path` 上调用 `toString()`，然后使用 `string` 操作查看结果。
* 在 `FileSystem` 对象上调用 `getPathMatcher()` 获得一个 `PathMatcher`，然后传入您感兴趣的模式——`glob` 或 `regex`。如`PathMatcher matcher = FileSystems.getDefault().getPathMatcher("glob:**/*.{tmp,txt}");`

* glob：开头的 `**/` 表示“当前目录及所有子目录”；单 `*` 表示“任何东西”；只使用 `*.tmp`，和 `map()` 操作配合使用，将完整路径减少到末尾的名称。

文件读写：

* 文件很小（运行的足够快且占用内存小），`Files.readAllLines(Paths.get(String))` 一次读取整个文件（因此，“小”文件很有必要），产生一个`List<String>`。 可继续调用`.stream()`进行流操作。
* `Files.write(Path p,byte[] bytes)` 被重载以写入 `byte` 数组或任何 `Iterable` 对象（它也有 `Charset` 选项）。
* 文件太大，`Files.lines()` 方便地将文件转换为行的 `Stream`。可以对该对象调用`map(一些操作).forEachOrdered((PrintWriter)output::println)`进行流与流直接的转换。



## 第十八章 字符串 20





## 第十九章 类型信息 21





## 第二十章 泛型  21



## 第二十一章 数组 22





## 第二十二章 枚举 

关键字 **enum** 可以将一组具名的值得有限集合创建为一种新的类型，每个元素作为常规的程序组件使用。**不能被继承**，可以**实现一个或多个接口**，但是通常需要一个 enum 实例才能调用其上的方法。

#### 基本操作和组合操作

--------------

* enum 的`.values()`，返回 enum 实例的数组，且该数组元素严格保持在 enum 中生命的顺序。
* enum的实例，调用`ordinal()` 返回一个 int 值，这是每个 enum 实例在声明时的次序，从 0 开始。
* Enum 类实现了 Comparable 接口，所以它具有 compareTo() 方法。同时，它还实现了 Serializable 接口。
* enum的实例，调用`getDeclaringClass() ` 返回所属的  enum 类。
* enum的实例，调用`name() ` 返回名字。

**静态类型导入用于 enum**

**static import**，将 enum 实例的标识符带入当前的命名空间，所以无需再用 enum 类型来修饰 enum 实例。应当注意避免导致代码复杂度过高以至于令人难以理解。即`import static enums.Alarm.*;`

**方法添加**

enum中可以添加方法，甚至有 main() 方法。需要注意两点：

1. 如果你打算定义自己的方法，那么必须在 enum 实例序列的最后添加一个分号。
2. 必须先定义 enum 实例。

**方法覆盖**：如重写 `toString()`方法。

#### **switch** 和 **enum**

**switch** 和 **enum** 组合，在 enum 元素组被覆盖的前提下，可以不要 default 语句（你非要不覆盖全编译器也不报错）；此外，在 case 语句中调用 return 编译器也会报错 缺少 default。

**深究 values 方法**

* 反编译 enum 类的代码

```java
OSExecute.command(
                "javap -cp build/classes/main Explore");//Explore 是enum类名
```

* 创建 enum 类时，编译器为其

  1. 添加了一个单参数的 `valueOf()`方法。（ Enum 类也有一个同名方法，需要两个参数）

  2. 将其标记为 **final** 。

  3. 添加了一个 static 的初始化子句。 

  4. 添加了一个`values()`的 static 方法，返回 enum 实例数组。另一种获取所有 enum 实例的方式为：

     enum 实例向上转型为 **Enum**，调用 `.getClass()`获得 Class 对象，再调用 Class 类的`getEnumConstants()`获得所有实例对象。



#### 枚举的复杂操作：接口组织、EnumSet、EnumMap

-------------

在一个接口的内部，**创建实现该接口的枚举**，以此将元素进行分组，可以达到将枚举元素分类组织的目的。举例来说，假设你想用 enum 来表示不同类别的食物，同时还希望每个 enum 元素仍然保持 Food 类型。

* 对于 enum 而言，实现接口是使其子类化的唯一办法。且可以向上转型为接口引用。

* 想创建一个“校举的枚举”，那么可以创建一个新的 enum，然后用其实例包装 旧 enum 中的每一个 enum 类。
* 一种更简洁的管理枚举的办法，就是将一个 enum 嵌套在另一个 enum 内。

**使用 EnumSet 替代 Flags**

```java
		EnumSet<AlarmPoints> points = EnumSet.noneOf(AlarmPoints.class); // Empty
        points.add(BATHROOM); //添加一个
        points.addAll(
                EnumSet.of(STAIR1, STAIR2, KITCHEN));//添加多个
        points = EnumSet.allOf(AlarmPoints.class);//添加全部
        points.removeAll(
                EnumSet.of(STAIR1, STAIR2, KITCHEN));//去掉集合
        points.removeAll(
                EnumSet.range(OFFICE1, OFFICE4));//去掉OFFICE1到OFFICE2之间（包括他们两个）全部
        points = EnumSet.complementOf(points);//求当前set中的补集
```

Java SE5 引入 EnumSet，是为了通过 enum 创建一种替代品，以替代传统的基于 int 的“位标志”。

EnumSet 的设计充分考虑到了速度因素，因为它必须与非常高效的 bit 标志相竞争（其操作与 HashSet 相比，非常地快）。

可以应用于多过 64 个元素的 enum。EnumSet 的基础是 long，一个 long 值有 64 位，而一个 enum 实例只需一位 bit 表示其是否存在。

**使用 EnumMap**

一种特殊的 Map，它要求其中的键（key）必须来自一个 enum，由于 enum 本身的限制，所以 EnumMap 在内部可由数组实现。速度很快，可用于查找。



#### enum的应用场景

---------

**常量特定方法**

Java 的 enum 允许程序员为 enum 实例编写方法，从而为每个实例赋予各自不同的行为。

enum 实例不能被当做 class 类型，而是一个LikeClasses 类型的 static final实例。

因为 enum 实例是 static ，无法访问外部类的非 static 元素或方法，所以对于内部的 enum 的实例来说，其行为与一半的内部类并不相同。

enum 实例的方法可被覆盖。

**使用 enum 的职责链**

* *职责链设计模式*：

  程序员以多种不同的方式来解决一个问题，然后将它们链接在一起。当一个请求到来时，它遍历这个链，直到链中的某个解决方案能够处理该请求。

**使用 enum 的状态机**

* *状态机*：

  一个状态机可以具有有限个特定的状态，它通常根据输入，从一个状态转移到下一个状态，不过也可能存在瞬时状态（transient states），而一旦任务执行结束，状态机就会立刻离开瞬时状态。

  由于 enum 对其实例有严格限制，非常适合用来表现不同的状态和输入。

**多路分发 & Enum**



### 本章小结

优雅清晰的代码，比什么都重要。



## 第二十三章 注解

注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方法，Java 5 引入，提供了 Java 无法表达的但是需要被程序员完整表述程序所需的信息。注解使得我们可以

1. **以编译器验证的格式存储程序的额外信息**。
2. 属于**语言层级的概念**，在源代码级别保存所有的信息。
3. 创建涉及重复工作的类或接口时，可使用注解来自动化和简化流程。如 EJB 中的很多额外功作通过注解来消除。
4. 注解可替代一些现有的系统，如 XDoclet （一种独立的文档化工具，专门设计用来生成注解风格的文档）。
5. 从语法角度来看，注解的使用方法与修饰符的使用方法一致。

注解的[语法]()十分简单，主要是在现有语法中添加 @ 符号。

Java 5 引入了前三种定义在 **java.lang** 包中的注解：

- **@Override**：表示当前的方法定义将覆盖基类的方法。如果你不小心拼写错误，或者方法签名被错误拼写的时候，编译器就会发出错误提示。
- **@Deprecated**：如果使用该注解的元素被调用，编译器就会发出警告信息。
- **@SuppressWarnings**：关闭不当的编译器警告信息。
- **@SafeVarargs**：在 Java 7 中加入用于禁止对具有泛型varargs参数的方法或构造函数的调用方发出警告。
- **@FunctionalInterface**：Java 8 中加入用于表示类型声明为函数式接口

### 定义注解







## 第二十四章 并发编程 23



## 第二十五章 设计模式 

模式的基本概念可以看做程序设计的基本概念，添加抽象层。即：将易变的事物与不变的事物分开。开发一个优雅且易维护设计中最困难的部分即发现“变化”的载体。设计模式的目的即隔离代码中的更改。

设计模式的三种类别：

1. **创建型**：如何创建对象。 比如单例模式（Singleton）
2. **构造型**：设计对象以满足特定的项目约束。它们处理对象与其他对象连接的方式，以确保系统中的更改不需要更改这些连接。
3. **行为型**：处理程序中特定类型的操作的对象。比如观察者和访问者模式。

设计模式：

* **单例模式：**提供一个且只有一个对象实例的方法。

  嵌套私有类在首次引用之前不会加载。创建单例的关键是防止客户端程序员直接创建对象。

* **模板方式模式：**构造应用程序框架时常用。通常隐藏在底层，在基类中定义且不能更改。

  它有时是一个 **private** 方法，但实际上总是 **final**。它调用其他基类方法(您覆盖的那些)来完成它的工作,但是它通常只作为初始化过程的一部分被调用(因此框架使用者不一定能够直接调用它)。

* **代理模式和桥接模式：**以某种方式“代表”具体实现的方法的调用就行。

  比如把接口的实现放入内部类中，并在内部类里重载接口，在外围类的重载中调用内部类的重载方法。

  在结构上，代理模式和桥接模式的区别很简单:代理模式只有一个实现，而桥接模式有多个实现。在设计模式中被认为是不同的:代理模式用于控制对其实现的访问，而桥接模式允许您动态更改实现。

  设计模式中描述的代理模式的常见用途如下:

  1. 远程代理。它在不同的地址空间中代理对象。

  2. 虚拟代理。这提供了“懒加载”来根据需要创建“昂贵”的对象。

  3. 保护代理。当您希望对代理对象有权限访问控制时使用。

  4. 智能引用。要在被代理的对象被访问时添加其他操作。例如，跟踪特定对象的引用数量，来实现写时复制用法，和防止对象别名。一个更简单的例子是跟踪特定方法的调用数量。您可以将Java引用视为一种保护代理，因为它控制在堆上实例对象的访问(例如，确保不使用空引用)。

* **状态模式：** 

  代理模式中的例子，将内部类放到外面，并实现多个，在接口中加一个 change() 方法，实现**代理对象的生命周期内从一个实现切换到另一种实现的方法**。

* **状态机：**

  桥接模式允许程序员更改实现，状态机利用一个结构来自动地将实现更改到下一个。当前实现表示系统所处的状态，系统在不同状态下的行为不同(因为它使用桥接模式)。