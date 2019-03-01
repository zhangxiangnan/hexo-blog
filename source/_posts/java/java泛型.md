---
title: JAVA泛型
date: 2017-12-18 20:46:15
tags:
- generics
- java
categories: java
---

Java中的泛型详解！
<!-- more -->

#### 为什么用泛型
泛型支持在定义类、接口、方法时将类型当作参数，称作类型参数/类型变量，和形参类似，形参是输入值不同，类型参数是输入类型不同。
- 编译时更强的类型检测，避免更多运行时异常，减少bug
- 消除强转
- 实现支持不同类型集合的泛型算法，更安全、易懂

#### 泛型类型
泛型是类型参数化的泛型类或接口。
不用泛型时(可放各种类型，所以运行时强转时容易异常)：
```
    public class Box {
      private Object object;
      public void set(Object object) { this.object = object; }
      public Object get() { return object; }
    }
```

泛型类的定义：
```
    class name<T1, T2, ..., Tn> { }
```
例子：
```
    public class Box<T> {
        // T stands for "Type"
        private T t;

        public void set(T t) { this.t = t; }
        public T get() { return t; }
    }  
```
类型变量T可以在类中任何地方使用，类型变量可以是任何非基本类型的类型：任意类类型、接口类型、以至另一个类型变量。

##### 类型参数命名规范
约定俗成，类型参数单字母、大写，常用的如下：
- E 元素 (java集合框架常用）
- K Key
- N Number
- T Type
- V Value
- S,U,V等

##### 泛型调用及实例化
泛型类型的调用(也称参数化类型)：Box<Integer> integerBox;
实例化：
```
    Box<Integer> integerBox = new Box<Integer>();
    Box<Integer> integerBox = new Box<>();//>=jdk7
```
##### 参数化类型
类型参数也可以是参数化类型：
```
    OrderedPair<String, Box<Integer>> p = new OrderedPair<>("primes", new Box<Integer>(...));
```
#### 泛型方法
泛型方法指引入类型参数的方法，在方法的返回类型之前声明，使用<>，可以有多个类型参数，适用于静态、非静态、构造方法。
```
    public class Util {
        public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
            return p1.getKey().equals(p2.getKey()) &&
                   p1.getValue().equals(p2.getValue());
        }
    }
    public class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
  }

    Pair<Integer, String> p1 = new Pair<>(1, "apple");
    Pair<Integer, String> p2 = new Pair<>(2, "pear");
    boolean same = Util.<Integer, String>compare(p1, p2);
    boolean same = Util.compare(p1, p2);//类型推导
```
#### 有界（受限）类型参数
  有些情况想限制类型参数的参数类型为某个类型或及其子类型，用extends表示.
  extends此处表示类的继承和接口的实现。
```
    public <U extends Number> void inspect(U u){
          System.out.println("T: " + t.getClass().getName());
          System.out.println("U: " + u.getClass().getName());
    }
```
有界类型参数可以调用界限类型里定义的方法，如：
```
    public class NaturalNumber<T extends Integer> {
      private T n;
      public NaturalNumber(T n)  { this.n = n; }
      public boolean isEven() {
          return n.intValue() % 2 == 0;
      }
      // ...
    }
```
界限可以有多个，用&连接，多个界限中最多只能有一个类；如果有一个类，多个接口，则该类必须在最左，如<T extends B1 & B2 & B3> （B1为类）

有界类型参数是泛型算法实现的关键，如：
```
    public interface Comparable<T> {
      public int compareTo(T o);
    }
    public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
        int count = 0;
        for (T e : anArray)
            if (e.compareTo(elem) > 0)
                ++count;
        return count;
    }
```
#### 泛型&继承&子类型
```
      Box<Number> box = new Box<Number>();
      box.add(new Integer(10));   // OK
      box.add(new Double(10.1));  // OK

      public void boxTest(Box<Number> n) { }
      boxTest(new Box(10))
      //error,Box<Integer>、Box<Double>都不是Box<Number>的子类型。
```
A继承于B，但是不代表Class<A>继承于Class<B>

泛型的继承可以通过extends或implements来实现：
```
    ArrayList<String> -> List<String> -> Collection<String>
```
#### 类型推导
Java编译器从方法调用传入的类型以及对应的方法声明的参数类型来推断出使方法调用最合理的参数类型。
```
    static <T> T pick(T a1, T a2) { return a2; }
    Serializable s = pick("d", new ArrayList<String>())
```
有了类型推导，在泛型方法调用、实例化泛型类、泛型类/非泛型类的泛型构造方法调用都可以省略。
类型推导只通过调用参数、目标类型、返回值类型来推导，而不是用程序后续的结果类型。

#### 通配符
泛型中，问号标记“？”叫通配符，表示一种未知类型，通配符可以用在以下情形：参数的类型、字段、本地变量、返回值；不用于泛型方法调用的类型参数、泛型类的创建、父类型。

##### 上限通配符
上限通配符用来放宽变量的限制，比如想定义一个方法，适用于 List<Integer>, List<Double>, and List<Number>类型。
```
    public static void process(List<? extends Number>) {}
```
List<Number>限制所有类型只能为Number，但List<? extends Number>可匹配Number及Number子类型，并且每个元素都可以调用Number类里的方法。

##### 无界通配符
使用?定义，如List<?>，称作未知类型的list，适用场合：
  - 如果在写一个可以使用Object类中的功能实现的方法
  - 当代码在泛型中使用不依赖类型参数的方法，如List.size List.clear，实际上Class<?>最常用，因为Class<T>里的多数方法不依俩T。
```
    // 不适用于List<Integer>、List<String>等
    public static void printList(List<Object> list) {
      for (Object elem : list)
          System.out.println(elem + " ");
      System.out.println();
    }

    // 打印，适合任务类型
    public static void printList(List<?> list) {
      for (Object elem: list)
          System.out.print(elem + " ");
      System.out.println();
    }
```

List<?>与List<Object>不一样，List<Object>可以往里面添加Object、及Object的任务子类型，但是List<?>声明的变量只能往里添加null。

##### 下界通配符
下届通配符用 ? super Foo 表示只能是Foo或Foo的父类型，不能同时声明下届和上界。

List<Integer>和List<? super Integer>不一样，List<Integer>匹配Integer的列表，但是List<? super Integer>匹配Integer及Integer的父类型。

##### 通配符和子类型
A extends B，但是Class<A>和Class<B>没有继承关系，如何让两者有关系？
```
    List<? extends Integer> intList = new ArrayList<>();
    List<? extends Number>  numList = intList;  // OK. List<? extends Integer>是List<? extends Number>的子类型
```
List<?>是List<Integer>和List<Number>的父类型。

List<Integer> -> List<? extends Integer> -> List<? extends Number> -> List<?>

List<Number> -> List<? super Number> -> List<? super Integer> -> List<?>

List<Number> -> List<? extends Number>

List<Integer> -> List<? super Integer>

##### 通配符捕获和辅助方法
有些情况下，编译器可以捕获到通配符的类型，这种叫通配符捕捉。
```
  public class WildcardError {

    void foo(List<?> i) {
        i.set(0, i.get(0));//报包含capture of ?的错误
      }
    }
```
添加辅助方法来解决：
```
    public class WildcardFixed {
      void foo(List<?> i) {
          fooHelper(i);
      }
      // 创建辅助方法以便可以通过类型推导来进行通配符类型捕获
      private <T> void fooHelper(List<T> l) {
          l.set(0, l.get(0));
      }
    }
```
##### 通配符的使用场合
何时使用上界通配符、何时使用下界通配符？首先通配符主要用在方法的形式参数声明上，应避免使用在方法返回类型上（调用者需要处理通配符）
- 输入变量(In)就是一个提供数据给代码使用的变量，如拷贝方法copy(src, dst)，src就是输入变量，因为src提供了用来拷贝的数据（producer）；
- 输出变量(Out)就是一个保存数据以便在其他地方使用的变量，如copy方法的dst变量，数据拷贝到了dst接受了拷贝的数据(consumer)
- 又用于输入且用于输出的变量

使用指南（方法的参数声明）：
  - 输入变量使用上界通配符，extends，此时变量只读
  - 输出变量使用下界通配符，super，此时变量只可写
  - 针对输入变量可以使用Object类里定义的方法访问时，使用无界通配符
  - 当代码同时需要将变量当作输入、输出变量时，不使用通配符

以上描述总结出PECS－Producer extends, consumer super.

List<? extends ...>定义的变量可以认为只读，但是不是严格意义的只读，只是说不能修改已有元素、不能新增元素，但是仍可以执行如下：
 - clear
 - 插入null
 - 迭代与删除
 - 通过通配符捕捉来写入元素

#### 类型擦除
Java引入泛型来加强编译器类型检查以及支持泛型编程，java使用类型擦除来实现泛型：
 - 替换泛型类型中所有的类型参数为他们的边界类型或者为Object（类型参数无界时）,因为生成的字节码都是普通的类、接口、方法
 - 如果需要保持类型安全，插入类型转换指令
 - 生成桥接方法来保持泛型类型继承中的多态性

java针对参数化类型不会生成新的类，所以泛型不会产生运行时开销。

##### 泛型类型擦除
类型擦除时，编译器擦除所有的类型参数，并每个都使用最左边界类型或者Object(无界类型)来替换 ，Node<T>中T是无界通配符。

##### 泛型方法擦除
泛型方法擦除时，和类型擦除一样。

##### 桥接方法
如下：
```
    public class Node<T> {
    public T data;
    public Node(T data) { this.data = data; }
      public void setData(T data) {
          System.out.println("Node.setData");
          this.data = data;
      }
    }
    public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }
      public void setData(Integer data) {
          System.out.println("MyNode.setData");
          super.setData(data);
      }
    }
```
类型擦除后：
```
    public class Node {
      public Object data;
      public Node(Object data) { this.data = data; }
      public void setData(Object data) {
          System.out.println("Node.setData");
          this.data = data;
      }
    }

    public class MyNode extends Node {
      public MyNode(Integer data) { super(data); }
      public void setData(Integer data) {
          System.out.println("MyNode.setData");
          super.setData(data);
      }
    }
```
擦除后setData方法不能覆盖父类的setData方法，为了保持泛型类型在擦除后的多态特性，java编译器自动生成桥接方法如下：
```
    class MyNode extends Node {
      //桥接方法
        public void setData(Object data) {
            setData((Integer) data);
        }
        public void setData(Integer data) {
            System.out.println("MyNode.setData");
            super.setData(data);
        }
    }
```
#### 泛型的限制
##### 不能使用基本类型实例化泛型类型
```
    Pair<int, char> p = new Pair<>(8, 'a');//error
    Pair<Integer, Character> p = new Pair<>(8, 'a');// 8，'a'会自动装箱
```
##### 不能创建类型参数的实例
```
    public static <E> void append(List<E> list) {
      E elem = new E();  // compile-time error
      list.add(elem);
    }

    public static <E> void append(List<E> list, Class<E> cls) throws Exception {
      E elem = cls.newInstance();   // 反射可以实现
      list.add(elem);
    }
```
##### 不能声明类型为类型参数的静态字段
```
    public class MobileDevice<T> {
      private static T os;//编译错误
    }
```
因为静态字段是所有类的实例共享，多个实例实例化时传入不同的T类型，则无法确定os字段属于哪一个类型。

##### 针对参数化类型不能使用cast或instanceof
```
    public static <E> void rtti(List<E> list) {
      if (list instanceof ArrayList<Integer>) { //编译错误
      }
    }

    public static void rtti(List<?> list) {
      if (list instanceof ArrayList<?>) {  // 无界通配符OK; instanceof需要一个具体化的类型
      }
    }

    List<Integer> li = new ArrayList<>();
    List<Number>  ln = (List<Number>) li;  //编译错误

    List<String> l1 = ...;
    ArrayList<String> l2 = (ArrayList<String>)l1;  // OK
```
##### 不能创建参数化的数组类型
```
    List<Integer>[] arrayOfLists = new List<Integer>[2];  //编译错误，数组声明时需要具体化的类型
```
##### 不能创建、捕获、抛出序列化类型的对象
```
    // 不能隐式地继承Throwable
    class MathException<T> extends Exception {  }    // compile-time error

    // 不能显式地继承Throwable
    class QueueFullException<T> extends Throwable { } // compile-time error

    // 不能创建类型参数的实例
    public static <T extends Exception, J> void execute(List<J> jobs) {
      try {
          for (J job : jobs)
              // ...
      } catch (T e) {   // compile-time error
          // ...
      }
    }

    可以在throws子句中使用类型参数
    class Parser<T extends Exception> {
      public void parse(File file) throws T {     // OK
      }
    }
```
##### 形式类型参数擦除后类型一样的不能重载
```
    public class Example {
      public void print(Set<String> strSet) { }
      public void print(Set<Integer> intSet) { }
    }
```
#### 非具体化类型
 具体化类型是指运行时完全可以获取到其类型信息的类型，如基本类型、非泛型类型、裸（原始）类型、或者无界通配符的调用。
 非具体化类型是指编译时类型信息被擦除,如泛型类型的调用（无界通配符除外），非具化类型运行时没有足够的类型信息。如JVM运行时无法区分List<String>、List<Integer>。非具化类型不能用于：instaceof表达式、或者作为数组的元素。

##### 堆污染
当参数化类型的变量引用非参数化类型的对象时，就会发生堆污染。在编译或运行时如果无法验证包含参数话类型当操作如转换、调用的正确性，就会报未经检查的警告，需要引起注意。
