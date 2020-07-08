# 范型程序设计

* 为什么要使用泛型程序设计
* 定义简单泛型类
* 泛型方法
* 类型变量的限定
* 泛型代码和虚拟机
* 约束与局限性
* 泛型类型的继承规则
* 通配符类型
* 反射和泛型

## 为什么要使用泛型程序设计

泛型程序设计意味着编写的代码可以被很多不同类型的对象所重用。例如，我们并不希望为聚集String和File对象分别设计不同的类。实际上，也不需要这样做，因为一个ArrayList类可以聚集任何类型的对象。这就是一个泛型程序设计的实例。

### 类型参数的好处

在java中增加泛型类之前，泛型程序设计是用继承来实现的。ArrayList类只维护一个Object引用的数组：

```java
public class ArrayList//before generic classes
{
  private Object[] elementData;
  ...
  public Object get(int i){...}
  public void add(Object o){...}
}
```

这种方法有两个问题，当获取一个值时候，必须进行强制类型转换。

```java
ArrayList files=new ArrayList();
...
String filename=(String) files.get(0);
```

此外，这里没有错误检查。可以向数组列表中添加任何类的对象。在某些强制类型转换中就会产生一个错误。

泛型提供了一个更好的解决方案：类型参数。ArrayList类有一个类型参数用来指示元素的类型：

```java
ArrayList<String> files=new ArrayList<String>();
```

这使得代码具有更好的可读性。一眼就知道这个数组列表中包含的是String对象。编译器就不需要进行强制类型转换，因为它知道返回值类型为String，而不是Object。

```java
String filename=files.get(0);
```

而且，编译器还可以进行检查，避免插入错误类型的对象。类型参数的魅力在于：使得程序具有更好的可读性和安全性。

### 谁想成为泛型程序员

实现一个范型类很麻烦，对于类型参数，使用这段代码的程序员可能想要内置所有的类。它们希望在没有过多的限制以及混乱的错误消息的状态下，做所有的事情。因此，一个范型程序员的任务就是预测出所用类的未来可能有的所有用途。

这一任务难到什么程度呢，下面是标准类库的设计者们肯定产生争议的一个典型问题。ArrayList类有一个方法addAll用来添加另一个集合的全部元素。程序员可能想要将`ArrayList<Manager>`中的所有元素添加到`ArrayList<Employee>`中去。然而，反过来就不行了。如果只能允许前一个调用，而不能允许后一个调用，java语言的设计者发明了一个具有独创性的新概念，通配符类型，它解决了这个问题。通配符类型非常抽象，然而，它们能让库的构建者编写出尽可能灵活的方法。

## 定义简单泛型类

一个范型类就是具有一个或多个类型变量的类。本章使用一个简单的Pair类作为例子。对于这个类来说，我们只关注泛型，而不会为数据存储的细节烦恼。

```java
public class Pair<T>
{
  private T first;
  private T second;

  public Pair(){first=null;second=null;}
  public Pair(T first,T second){this.first=first;this.second=second;}

  public T getFirst(){return first;}
  public T getSecond(){return second;}

  public void setFirst(T newValue){first=newValue;}
  public void setSecond(T newValue){second=newValue;}
}
```

Pair类引入了一个类型变量T，用尖括号括起来，并放在类名的后面。泛型类可以有多个类型变量

用具体的类型替换类型变量就可以实例化泛型类型，例如`Pair<String>`，可以将结果想象成带有构造器的普通类：

```java
Pair<String>()
Pair<String>(String,String)
```

换句话说，泛型类可以看作普通类的工厂。

## 泛型方法

前面已经介绍了如何定义一个泛型类，实际上，还可以定义一个带有类型参数的简单方法。

```java
class ArrayAlg
{
  public static <T> T getMiddle(T... a)
  {
    return a[a.length/2]
  }
}
```

这个方法是在普通类中定义的，而不是在泛型类中定义的。然而，这是一个泛型方法，可以从尖括号和类型变量看出这点。注意，类型变量要放在修饰符的后面，返回类型的前面。

泛型方法可以定义在普通类中，也可以定义在泛型类中。

当调用一个泛型方法时候，在方法名前的尖括号中放入具体的类型：

```java
String middle=ArrayAlg.<String>getMiddle("John","Q.","Public");
```

在这种情况下，方法调用中可以省略<String>类型参数。编译器有足够的信息能够推断出所调用的方法。它用names的类型（即String[]）与泛型类型T[]进行匹配并推断出T一定是String。也就是说，可以调用

```java
String middle=ArrayAlg.getMiddle("john","Q.","public")
```

几乎在大多数情况下，对于泛型方法的类型引用没有问题。偶尔，编译器也会提示错误，此时需要解译错误报告。如

```java
double middle=ArrayAlg.getMiddle(3.14,1729,0);

```

## 类型变量的限定

有时候，类或方法需要对类型变量加以约束。举个例子，我们要计算数组中的最小元素

```java
class ArrayAlg
{
  public static <T> T min(T[] a)//almost correct
  {
    if (a==null || a.length==0) return null;
    T smallest=a[0];
    for (int i=1;i<a.length;i++)
      if (smallest.compareTo(a[i])>0) smallest=a[i];
    return smallest;
  }
}
```

但是，这里有一个问题，请看一下main方法的内部。变量smallest类型为T，这意味着它可以是任何一个类的对象。怎么才能确信T所属的类有compareTo方法呢？

解决这个问题的方案是将T限制为实现了Comparable接口（只含一个方法compareTo的标准接口）的类。可以通过对类型变量T设置限定实现一点：

```java
public static <T extends Comparable> T min(T[] a)...
```


实际上Comparable接口本身就是一个范型类型。这里为什么使用关键字extends而不是implements，下面的记法：

```java
<T extends BoundingType>
```

表示T应该是绑定类型的子类型。T和绑定类型可以是类，也可以是接口。选择关键字extends的原因是更接近子类的概念。

## 范型代码和虚拟机

待续
