## 快速索引区——OnJava8

### 一、内部类

* 内部类的创建方式和访问权限

* 内部类自动获取一个对所在外部类的引用，`OuterClassName.this`。

* 内部类的访问权限可为`private`或`protected`。普通类只能为`public`或包访问权限

* 内部类向上转型为基类/接口，可隐藏细节，阻止依赖类型的编码

* **局部内部类**的创建方式和访问权限

  （方法）作用域中，无访问说明符，有外围类成员访问权限

* **匿名内部类**的创建方式和访问权限

  使用与定义在一起（或实例化 `abstract` 类），不可能有命名构造器，扩展类和实现接口只能选其一，且接口只能实现一个，参数引用外部对象需显式`final`，类中定义字段需初始化。

* **嵌套类**的创建方式和访问权限（`static` 类）

* 内部类和多重继承（继承两个类的实现方法）

* **闭包**的两种生成方式（内部类和 Lambda 表达式）

  闭包引用的局部变量必须是`final`

* 闭包和指针（闭包更灵活更安全）

* 内部类被继承时的注意事项（外围类的引用和调用？）

### 二、集合

* 分类

  **List**（以特定顺序保存一组元素）

  * `ArrayList`（随机访问快）
  * `LinkedList`（插入删除快，可被作为栈、队列、双端队列）

  **Set**（元素不允许重复）

  * `HashSet`（检索元素最快），**散列**
  * `TreeSet`（将比较结果的升序保存对象），**红黑树**
  * `LinkedHashSet`（按被添加的先后顺序保存对象，且提供快速访问），链表加散列

  **Queue**（只能在一端插入并从另一端移除，并发编程/安全性）

  * `Deque`
  * `LinkedList`

  * `PriorityQueue`（先进先出且排序）

  **Map**（用键查找值，可生成 **Set**）

  * `ArrayList`
  * `HashMap`
  * `TreeMap`
  * `LinkedHashMap`（在保持`HashMap`查找速度的同时按键的插入顺序保存键）

* 添加元素组的三种方法

  `collection.addAll()`；`Collections.addAll()`（更灵活）；`Arrays.asList()`。

* `Iterable` 接口：`iterator()`方法

  * `listIterator`

* 工具类

  * `Collections`
    * `Collections.sort(list,new Comparator<Integer>(){});`
  * `Arrays`
    * `Arrays.sort` 对于很小的数组（小于27）使用插入排序，其他使用双轴快排。轴点p1 比p2 小。

### 三、函数式编程

* 面向对象是抽象数据，函数式编程是抽象行为。

* 合并现有代码来生成新功能，将代码传递给方法。

* 传统方法

  类，接口，覆写函数体。

* **匿名内部类**

  此处的方法签名（方法名称和参数列表）

* **lambda  表达式**

  * 单一方法接口

* **方法引用**

  * 单一方法接口

  * 此处的方法签名（参数类型和返回类型）

  * 方法来源（必须静态或实例化对象的绑定引用）

  * 未绑定的方法引用，此时的方法签名规则改变，参数列表留出第一个位置给`this`。

* **函数式方法**（单方法接口中的抽象方法）

  *  引用`java.util.function`包中的接口
  *  `@FunctionalInterface` 注解自己的接口，强制执行“函数式方法”模式

* **高阶函数**（消费或产生函数的函数）

  * 消费函数
* 产生函数
  
* **闭包**

### 四、流式编程

* **创建流**

  * `Stream.of(Element ...element).forEach(System.out:print)`
  * **Random** 类的 `.boxed()`
  * **Stream.**`generate(Supplier<T> supplier)` 
  * `IntStream.range()`
  * **Stream.**`iterate()`
  * **Stream.**`builder()`
  * `Arrays.stream()`
  * **Pattern.**` splitAsStream(String all)`

* **修改流元素**

* **Optional 类**

* **消费流元素**

### 五、字符串

* 字符串的不可变性
* Java 只有两个重载操作符（用于 `String` 的 `+` 与 `+=` ）
* `StringBuilder`和`StringBuffer`
* **CharSequence** 接口：：`CharBuffer`、`String`、`StringBuffer`、`StringBuilder` 
* `Pattern` 和 `Matcher`

### 六、泛型

* **多态**

  一种面向对象思想的泛化机制。将方法的参数类型设为基类，则该方法可接受任何派生类作为参数，包括暂时还不存在的类。此外，如果方法以接口而不是类作为参数，就更宽松了，接口可以突破继承体系的限制。

* **泛型**格式：使用**类型参数**

* `Supplier<T>`接口：`get()`

* **泛型擦除**

  * 在泛型代码内部，无法获取任何有关**泛型参数类型**的信息。编译器无法确定类型`T`的边界（比如是否有方法 f()）。
  * 任何基本类型都不能作为类型参数
  * 一个类不能实现同一个泛型接口的两种变体，由于擦除，这两个变体会变成相同的接口
  * 使用带有泛型类型参数的转型或 **instanceof** 不会有任何效果

  * 由于擦除，重载方法可能会产生一样的类型签名。（编译器可检测到）
  * **catch** 语句不能捕获泛型类型的异常

###  七、一些接口

* `Supplier<T>`接口：`get()`

* `Iterable` 接口：`iterator()`

* `java.lang.Comparable`接口：`compareTo()`

  实现了该接口的对象数组可直接调用`Arrays.sort()`进行排序。

* `Comparator`接口：两个方法`compare()`和`equals()`

  * `Collections.reverseOrder()`
* `Arrays.sort(a, new CompTypeComparator());`
  
* 





## 按常考点整理

### 1. String，StringBuilder和StringBuffer

在对字符串进行`toString()`或各种增删改查工作时，编译器自动引入了`java.lang.StringBuilder`类。显式使用`StringBuilder`避免性能问题，且可以预先指定大小，避免频繁的重新分配缓存。`StringBuilder `是 Java SE5 引入的，在这之前用的是 `StringBuffer`。后者是线程安全的，因此开销也会大些。使用 `StringBuilder` 进行字符串操作更快一点。

### 2. Java 类初始化顺序

- 父类静态属性，静态代码块
- 子类静态属性，静态代码块
- 父类普通属性，普通代码块，构造函数
- 子类普通属性，普通代码块，构造函数

3、



