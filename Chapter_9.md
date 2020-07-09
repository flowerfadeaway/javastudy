# 集合

* java集合框架
* 具体的集合
* 映射
* 视图与包装器
* 算法
* 遗留的集合

在实现方法时候，选择不同的数据结构会导致其实现风格以及性能存在着很大差异。本章将讲述如何利用java类库帮助我们在程序设计中实现传统的数据结构。

## java集合框架

### 将集合的接口与实现分离

与现代的数据结构类库的常见情况一样，java集合类库也将接口与实现分离。首先，看一下人们熟悉的数据结构——队列是如何分离的。

队列接口指出可以在队列的尾部添加元素，在队列的头部删除元素，并且可以查找队列中元素的个数。当需要收集对象，并按照“先进先出”的规则检索对象就应该使用队列。队列接口的最简形式可能类似下面这样：

```java
public interface Queue<E>//a simplified form of the interface in the standard library
{
  void add(E element);
  E remove();
  int size();
}
```

这个接口并没有说明队列如何实现的。队列通常有两种实现方式：一种使用循环数组，另一种是使用链表。每一个实现都可以通过一个实现了Queue接口的类表示。

```java
public class CircularArrayQueue implements Queue<E>//not an actual library class
{
  private int head;
  private int tail;

  CircularArrayQueue(int capacity){...}
  public void add(E element){...}
  public E remove(){...}
  public int size(){...}
  private E[] elements;
}

public class LinkedListQueue<E> implements Queue<E>
{
  private Link head;
  private Link tail;

  LinkedListQueue(){...}
  public void add(E element){...}
  public E remove(){...}
  public int size(){...}
}
```

当在程序中使用队列时候，一旦构建类集合就不需要知道究竟使用了哪种实现。因此，只有在构建集合对象时候，使用具体的类才有意义。可以使用接口类型存放集合的引用。

```java
Queue<Customer> expressLane=new CircularArrayQueue<>(100);
expressLane.add(new Customer("Harry"));
```

利用这种方式，一旦改变了想法，可以轻松地使用另外一种不同的实现。只需要对程序的一个地方作出修改，即调用构造器的地方。如果觉得LinkedListQueue是更好的选择，就将代码修改为：

```java
Queue<Customer> expressLane=new LinkedListQueue<>();
expressLane.add(new Customer("Harry"));
```

### Collection接口

在java类库中，集合类的基本接口是Collection接口。这个接口有两个基本方法：

```java
public interface Collection<E>
{
  boolean add(E element);
  Iterator<E> iterator();
  ...
}
```

add方法用于向集合中添加元素。如果添加元素确实改变了集合就返回true，如果集合没有发生变化就返回false。例如，如果试图向集中添加一个对象，而这个对象在集中已经存在，这个添加请求就没有生效，因为集中不允许有重复的对象。

iterator方法用于返回一个实现了Iterator接口的对象。可以使用这个迭代器对象依次访问集合中的元素。

### 迭代器

Iterator接口包含4个方法：

```java
public interface Iterator<E>
{
  E next();
  boolean hasNext();
  void remove();
  default void forEachRemaining(Consumer<? super E> action);
}
```

### 集合框架中的接口

java集合框架为不同类型的集合定义了大量接口

![fig](./picture/C9/1.jpg)

集合有两个基本接口：`Collection`和`Map`

## 具体的集合

图中展示了java类库中的集合，并简要描述了每个集合类的用途，除了以Map结尾的类之外，其他类都实现了Collection接口，而以Map结尾的类实现了Map接口。

![fig](./picture/C9/2.jpg)

![fig](./picture/C9/3.jpg)

### 链表

在java中，所有链表实际都是双向链接的——即每个结点还存放着指向前驱结点的引用。

![fig](./picture/C9/4.jpg)

例如：

```java
List<String> staff=new LinkedList<>();//LinkedList implements List
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
Iterator iter=staff.iterator();
String first=iter.next();
String second=iter.next();
iter.remove();
```

### 数组列表

集合类库中提供了一种ArrayList类，这个类也实现了List接口。ArrayList封装了一个动态再分配的对象数组。

### 散列表

在java中，散列表用链表数组实现。数组的每个位置称为桶。

![fig](./picture/C9/5.jpg)

之后还有散列冲突什么的，和python讲的差不多，不赘述。

### 树集

TreeSet类与散列集十分类似，不过它比散列集有所改进。树集是一个有序集合。可以以任意顺序将元素插入到集合中。在对集合进行遍历时候，每个值将自动地按照排序后的顺序呈现。例如：

```java
SortedSet<String> sorter=new TreeSet<>();//TreeSet implements SortedSet
sorter.add("Bob");
sorter.add("Amy");
sorter.add("Carl");
for (String s:sorter) System.println(s);
```

这时候，每个值将按照顺序打印出来：Amy Bob Carl。排序是用树结构完成的（当前实现使用的是红黑树），每次将一个元素添加到树中，都被放置在正确的排序位置上。

### 队列与双端队列

在java6中引入了Deque接口，并由ArrayDeque和LinkedList类实现。这两个类都提供类双端队列，而且在必要时候，可以增加队列的长度。

### 优先级队列

优先级队列中的元素可以按照任意的顺序插入，却总是按照排序的顺序进行检索。也就是说，无论何时调用remove方法，总会获得当前优先级队列中最小的元素。然而，优先级队列并没有对所有的元素进行排序。如果用迭代的方式处理这些元素，并不需要对它们进行排序。优先级队列使用了一个优雅且高效的数据结构，堆。

## 映射

### 基本映射操作

java类库为映射提供了两个通用的实现：HashMap和TreeMap。这两个类都实现了Map接口

散列映射对键进行散列，树映射用键的整体顺序对元素进行排序，并将其组织成搜索树。例如

```java
Map<String,Employee> staff=new HashMap<>();//HashMap implements Map
Employee harry=new Employee("Harry");
staff.put("987-98-9996",harry);
...
```

每当往映射中添加对象时候，必须同时提供一个键。如果在映射中没有与给定键对应的信息，get将返回null。


## 视图与包装器

待续
