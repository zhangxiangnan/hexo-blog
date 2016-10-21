layout: title
title: guava的不可变集合
date: 2016-10-20 13:10:12
tags:
- guava
- immutableCollections
categories:
- 译
- guava
---

guava的不可变集合讲解-Immutable*
<!-- more -->

### jdk的Collections.ImmutableXXX例子
    package cn.zxn.guava.collections;

    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;
    // jdk的方法修改原始集合会导致不可变集合发生变化
    public class ImmutableCollectionsExplained {

    public static void main(String[] args) {
        List<String> originalList = new ArrayList<String>();
        originalList.add("S");
        originalList.add("Q");

        List<String> unmodifiableList = Collections.unmodifiableList(originalList);
        System.out.println(unmodifiableList.size());// 输出2

        // 原始集合新增一个item
        originalList.add("T");
        // 不可变集合的大小变化,变为3
        System.out.println(unmodifiableList.size());// 输出3
    }
    }


### guava例子:   

    // 不可变set声明示例
    public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
      "red",
      "orange",
      "yellow",
      "green",
      "blue",
      "purple");

     class Foo {
        final ImmutableSet<Bar> immutableBars;
        Foo(Set<Bar> bars) {
          this.immutableBars = ImmutableSet.copyOf(bars);
          // 保护性拷贝，使集合immutableBars不可被删除增加元素，但是可以调用元素自身的方法进行元素的修改
        }
      }

### 使用不可变对象的原因
不可变对象有许多好处，包括：
  - 安全地被不受信任的类库使用
  - 线程安全：可以在多线程中使用而不用考虑(静态条件)加锁同步
  - 不需要支持可变性（自动扩充等机制），可以更好的节省空间、时间。所有的不可变集合实现都比对应的可变集合更能高效的使用内存。
  - 可以当做常量使用，其会保持不变

对对象objects做不可变的拷贝使一项很好的保护性编程技巧。Guava为每一个标准的集合类型提供简单、易用及不可变的版本，包含Guava自己的集合。

JDK提供Collections.unmodifiableXXX类似的方法，但在我们看来:
  - 笨拙且冗长：在你想做保护性拷贝的地方使用起来不方便
  - 不安全：返回的集合仅仅在其他地方没有持有对原始集合的引用前提下才真正不可变，guava则对原始集合的操作（删除、增加）不影响不可变集合。
  - 效率低下：内部数据结构仍然有可变集合会有的开销，包括并发修改检查，hash表中额外的空间等。

  **当你不想改变一个集合，或者希望一个集合保持不变，进行保护性拷贝原集合到一个不可变集合是一个不错的实践**

  重点：每一个Guava不可变集合实现类不能存放null值。我们做了一个彻底的对Google内部代码库调查，发现null元素仅仅在5%的情形里允许，其余95%则是若是null则会快速失败。如果你需要使用null值，考虑使用Collections.unmodifiableList及它的相关方法，可以允许null。更多细节见：[here](https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained)

### 如何使用
一个InmutableXXX集合可以通过以下几种方式创建：
  - 使用copyOf方法，例如，ImmutableSet.copyOf(set)；
  - 使用Of方法，如，ImmutableSet.of("a", "b", "c")或者ImmutableMap.of("a", "1", "b", 2);
  - 使用一个创建器Builder，如，   

        public static final ImmutableSet<Color> GOOGLE_COLORS =
           ImmutableSet.<Color>builder()
               .addAll(WEBSAFE_COLORS)
               .add(new Color(0, 191, 255))
               .build();
除了排序集合，元素顺序在构建时被保持。
如，ImmutableSet.of("a", "b", "c", "a", "d", "b") 迭代输出会得到结果： "a", "b", "c", "d".

#### 智能拷贝
  有必要记住ImmutableXXX.copyOf尝试避免拷贝数据（如果这么做安全地话）-- 确切的细节未指定，但是实现通常是智能的，如，

    ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
    thingamajig(foobar);

    void thingamajig(Collection<String> collection) {
      ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
      ...
    }
以上代码中，ImmutableList.copyOf(foobar) 将足够智能仅仅返回foobar.asList()，这是ImmutableSet的一个O(1)时间复杂度的方法。

ImmutableXXX.copyOf(ImmutableCollection)通常会在符合以下条件下来尝试来避免线性时间的拷贝：
  - 在常数时间内使用底层数据结果是可能的话：如, ImmutableSet.copyOf(ImmutableList) 不能在常数时间内完成.
  - 不会导致内存泄漏 -- 如，如果你有ImmutableList<String> hugeList（超大list）,如果执行ImmutableList.copyOf(hugeList.subList(0, 10)), 就会执行一个显示的拷贝, 以便避免意外地持有hugeList中不必要的引用
  - 不会改变语义 -- 所以ImmutableSet.copyOf(myImmutableSortedSet) 将会执行显示的拷贝, 因为ImmutableSet使用的hashCode()和equals拥有跟ImmutableSortedSet的基于comparator的行为不同的语义。
这有助于减少良好的防御性编程风格的性能开销。

#### asList

所有的不可变集合通过asList()提供一个不可变ImmutableList视图，所以 -- 如 -- 即使你有使用ImmutableSortedSet排序的数据，你也可以使用sortedSet.asList().get(k).来得到第k个最小的元素。

经常返回ImmutableList -- 不总是，但是经常 -- 一个常数级开销的视图，而不是执行显示拷贝。即，它更常常更聪明比average List -- 如，它使用支持集合的更高效的contains方法。
Details

### jdk可变集合与guava不可变集合对应关系？

接口   | 接口属于jdk或guava | 不可变版本
--------  | ---
Collection | JDK	| ImmutableCollection
List |	JDK	| ImmutableList
Set	| JDK	| ImmutableSet
SortedSet/NavigableSet | 	JDK	| ImmutableSortedSet
Map	| JDK	| ImmutableMap
SortedMap	| JDK	| ImmutableSortedMap
Multiset | 	Guava	| ImmutableMultiset
SortedMultiset	| Guava	| ImmutableSortedMultiset
Multimap	| Guava	| ImmutableMultimap
ListMultimap	| Guava	| ImmutableListMultimap
SetMultimap	| Guava	| ImmutableSetMultimap
BiMap	| Guava	| ImmutableBiMap
ClassToInstanceMap	| Guava	| ImmutableClassToInstanceMap
Table	| Guava	| ImmutableTable
