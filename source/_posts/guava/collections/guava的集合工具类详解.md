layout: title
title: guava的集合工具类详解
date: 2016-11-22 15:18:15
tags:
- guava
- CollectionUtilities
categories:
- 译
- guava
---

JDK集合框架提供了许多工具，方便实用，Guava相比提供了更多的工具，适用于所有集合，这也是guava更流行和成熟的部分。
<!-- more -->

guava针对一个特定的接口将其方法以一种相对直观的方式进行分组，如下:

接口 |	JDK还是Guava的接口	| 对应的Guava工具类
- |
Collection|	JDK	|Collections2 (避免和 java.util.Collections冲突)
List	|JDK	|Lists
Set|	JDK|	Sets
SortedSet|	JDK|	Sets
Map|	JDK	|Maps
SortedMap	|JDK|	Maps
Queue|	JDK	|Queues
Multiset|	Guava|	Multisets
Multimap	|Guava	|Multimaps
BiMap	|Guava|	Maps
Table|	Guava	|Tables

### 静态构造方式

JDK7之前，构造新的泛型集合需要讨厌的代码重复:

    List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();

Guava则借助于泛型推断来决定右侧类型：

    List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
    Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();

JDK7的菱形操作符也去除了右侧类型：

    List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();
但是guava优化的更好，使用工厂模式，可以很方便的初始化集合。

    Set<Type> copySet = Sets.newHashSet(elements);
    List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");

此外，可以根据工厂方法的名称来提高初始化集合到指定大小的可读性：

    List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
    List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
    Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);

注意: Guava自己的新集合类型没有暴露原始构造器或初始化工具方法。相反，Guava暴露了静态工厂方法，如：

    Multiset<String> multiset = HashMultiset.create();

### Iterables

Guava的工具方法总是接受Iterable而不是Collection，这是因为遇到一个不是存在在内存中的“集合”是很常见的，如从数据库，从数据中心；如果没有抓取到所有的元素，是不支持如size方法的。

因此，你希望的支持所有Collections的许多操作都可以在Iterables中找到。此外，大多数Iterables的方法在Iterables中有访问原始iterator的对应版本。

Iterables中的绝大多数方法都是很简便的: 只在有绝对需要的时候才增强底层迭代。返回Iterables的方法返回的是简单计算的视图，而不是内存中显示构造的新集合。

Guava12中，Iterables通过FlumentIterable支持，对大多数操作进行流式语法包装。



#### 常用方法
方法	|描述	|等同于
-|
concat(`Iterable<Iterable>`)	|返回几个Iterables对象的原始级联视图|concat(Iterable...)
frequency(Iterable, Object)	|返回指定object在Iterable中出现的次数	|Collections.frequency(Collection, Object); 参考Multiset
partition(Iterable, int)|	把Iterable分割为每块含有指定的数目，并返回不可修改的视图 | Lists.partition(List, int), paddedPartition(Iterable, int)
getFirst(Iterable, T default)	| 返回Iterable的第一个元素，若为空返回指定默认元素.	|Iterable.iterator().next() FluentIterable.first()
getLast(Iterable) | 返回iterable的最后元素，或者当为空时快速失败| getLast(Iterable, T default) FluentIterable.last()
 elementsEqual(Iterable, Iterable) | 当Iterables相同的顺序上拥有相同的元素才返回true| List.equals(Object)
unmodifiableIterable(Iterable) |返回iterable的不可修改视图|  Collections.unmodifiableCollection(Collection)
limit(Iterable, int) |返回元素个数最多为指定数字的Iterable| FluentIterable.limit(int)
getOnlyElement(Iterable) | 返回Iterable中的仅有的一个元素，若Iterable为空或者含有多个元素则快速失败。| getOnlyElement(Iterable, T default)

    Iterable<Integer> concatenated = Iterables.concat(
      Ints.asList(1, 2, 3),
      Ints.asList(4, 5, 6));
    // concatenated has elements 1, 2, 3, 4, 5, 6

    String lastAdded = Iterables.getLast(myLinkedHashSet);

    String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
      // if this set isn't a singleton, something is wrong!

#### Iterables类似Collection的方法
通常，Collections天然支持如下这些操作，但是Iterables不是这样。
这些操作当参数实际上是Collection时，每个都都代表了对应的集合接口方法。如，如果Iterables.size传入的参数实际上是一个Collection，方法内部调用Collection.size替代而不是遍历Iterator。

方法	|类似集合方法|	FluentIterable等价方法
-|
addAll(Collection addTo, Iterable toAdd)|	Collection.addAll(Collection)|
contains(Iterable, Object)|	Collection.contains(Object)|	FluentIterable.contains(Object)
removeAll(Iterable removeFrom, Collection toRemove)|	Collection.removeAll(Collection)|
retainAll(Iterable removeFrom, Collection toRetain)|	Collection.retainAll(Collection)|
size(Iterable)	|Collection.size()|	FluentIterable.size()
toArray(Iterable, Class)	|Collection.toArray(T[])|	FluentIterable.toArray(Class)
isEmpty(Iterable)	|Collection.isEmpty()	|FluentIterable.isEmpty()
get(Iterable, int)	|List.get(int)|	FluentIterable.get(int)
toString(Iterable)	|Collection.toString()|	FluentIterable.toString()

#### FluentIterable

FluentIterable也有几个拷贝到不可变集合的简洁的方法:

ImmutableSet toImmutableSet()
ImmutableSortedSet toImmutableSortedSet(Comparator)

### Lists
除了静态构造方法和函数式编程方法，Lists提供了很多实用的工具方法。

方法	|  描述
-|
partition(List, int)	|返回一个list视图，将原List分为指定大小的块
reverse(List)	|返回指定List的反转（倒序）视图，若果list不可变，请使用 ImmutableList.reverse()

    List<Integer> countUp = Ints.asList(1, 2, 3, 4, 5);
    List<Integer> countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}

    List<List<Integer>> parts = Lists.partition(countUp, 2); // {{1, 2}, {3, 4}, {5}}


### Sets

#### Set理论上的操作

We provide a number of standard set-theoretic operations, implemented as views over the argument sets. These return a SetView, which can be used:
Guava支持set理论上的操作，均返回SetView，其支持：

  - 直接作为一个Set，因为SetView实现了Set接口
  - 拷贝到另一个不可变的集合通过copyInto(set)
  - 通过immutableCopy()得到一个不可变的拷贝

|方法|
- |
|union(Set, Set)
|intersection(Set, Set)
|difference(Set, Set)
|symmetricDifference(Set, Set)

如：

    Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
    Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");

    SetView<String> intersection = Sets.intersection(primes, wordsWithPrimeLength); // contains "two", "three", "seven"
    // 可以直接使用intersection作为Set，但是如果使用很多次则拷贝更高效
    return intersection.immutableCopy();

其他Set工具方法

方法	|描述	|参照
-|-|
cartesianProduct(List<Set>)	|笛卡尔乘积，返回从每个Set选取的每个元素可能组成的list | cartesianProduct(Set...)
powerSet(Set)	| 返回指定set的子set的set集合 |

    Set<String> animals = ImmutableSet.of("gerbil", "hamster");
    Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");

    Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
    // {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
    //  {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}

    Set<Set<String>> animalSets = Sets.powerSet(animals);
    // {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}

### Maps

#### uniqueIndex方法

Maps.uniqueIndex(Iterable, Function) 解决的是拥有一组对象，每个对象的某个属性是唯一的，我们想通过这个唯一的属性来找到这个对象。

如我们有一组string，每个string长度不一样，想通过指定的长度来找到某个对象：

    ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, new Function<String, Integer> () {
        public Integer apply(String string) {
          return string.length();
        }
      });

#### difference方法

Maps.difference(Map, Map)比较的是2个map的所有不同点，返回一个MapDifference对象:
  - entriesInCommon() 在2个map里都有的entry，key和values都相等。
  - entriesDiffering()	key相同，但是有不同的values，可以通过MapDifference.ValueDifference来得到左边和右边的values
  - entriesOnlyOnLeft()	key在左边的map里存在但是在右边的map里没有
  - entriesOnlyOnRight()	key在右边的map存在，左边没有

      Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
      Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
      MapDifference<String, Integer> diff = Maps.difference(left, right);

      diff.entriesInCommon(); // {"b" => 2}
      diff.entriesDiffering(); // {"c" => (3, 4)}
      diff.entriesOnlyOnLeft(); // {"a" => 1}
      diff.entriesOnlyOnRight(); // {"d" => 5}

### BiMap的工具方法

The Guava针对BiMap的工具方法在Maps里，因为BiMap也是一个Map

BiMap utility |	Corresponding Map utility
-|-|
synchronizedBiMap(BiMap) |Collections.synchronizedMap(Map)
unmodifiableBiMap(BiMap)	|Collections.unmodifiableMap(Map)

### Multisets
标准的集合操作，如containsAll忽略了multiset中元素出现的次数，只关心元素是否在multiset中存在与否。Multisets则提供了许多重视元素多样性的操作：

方法|	描述	|和Collection方法的区别
-|-|
containsOccurrences(Multiset sup, Multiset sub)	|如果sub.count(o)<=super.count(o)对于所有的o成立，则返回true，否则false|	Collection.containsAll忽略了出现次数，仅仅测试元素是否被包含
removeOccurrences(Multiset removeFrom, Multiset toRemove)|	移除removeFrom中的一次出现针对toRemove元素的每一次出现|	Collection.removeAll移除了所有元素的所有出现次数，虽然toremove中仅仅出现一次
retainOccurrences(Multiset removeFrom, Multiset toRetain)|确保对于所有的o，removeFrom.count(o) <= toReatin.count(o)	|Collection.retainAll保留了所有元素的所有出现次数，虽然toReatin中仅仅出现一次。
intersection(Multiset, Multiset)	|返回2个multisets的交集视图，retainOccurrences的一种非破坏性方法|

    Multiset<String> multiset1 = HashMultiset.create();
    multiset1.add("a", 2);

    Multiset<String> multiset2 = HashMultiset.create();
    multiset2.add("a", 5);

    multiset1.containsAll(multiset2); // returns true: all unique elements are contained,
      // even though multiset1.count("a") == 2 < multiset2.count("a") == 5
    Multisets.containsOccurrences(multiset1, multiset2); // returns false

    multiset2.removeOccurrences(multiset1); // multiset2 now contains 3 occurrences of "a"

    multiset2.removeAll(multiset1); // removes all occurrences of "a" from multiset2, even though multiset1.count("a") == 2
    multiset2.isEmpty(); // returns true

其他Multisets中的工具方法包括：

方法 | 说明
-|
copyHighestCountFirst(Multiset)	|返回出现频次倒序的Multiset的不可变拷贝
unmodifiableMultiset(Multiset)|	返回Multiset的不可变视图
unmodifiableSortedMultiset(SortedMultiset)	|返回排序Multiset的不可变视图
    Multiset<String> multiset = HashMultiset.create();
    multiset.add("a", 3);
    multiset.add("b", 5);
    multiset.add("c", 1);

    ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);

    // highestCountFirst, 类似它的entrySet和elementSet, 以顺序{"b", "a", "c"}遍历元素
### Multimaps
#### index
对比Maps.uniqueIndex, Multimaps.index(Iterable, Function)解决的是当你想要找到某个属性上相同的所有对象时，也有可能是唯一的单独对象.
假如想通过长度来进行分组：
    ImmutableSet<String> digits = ImmutableSet.of("zero", "one", "two", "three", "four",
      "five", "six", "seven", "eight", "nine");
    Function<String, Integer> lengthFunction = new Function<String, Integer>() {
      public Integer apply(String string) {
        return string.length();
      }
    };
    ImmutableListMultimap<Integer, String> digitsByLength = Multimaps.index(digits, lengthFunction);
    `/*
     * digitsByLength maps:
     *  3 => {"one", "two", "six"}
     *  4 => {"zero", "four", "five", "nine"}
     *  5 => {"three", "seven", "eight"}
     */`

#### invertFrom
由于Multimap可以映射许多key到某个value，以及一个key到多个values，所有反转一个multimap会很有用。Guava提供invertFrom(Multimap toInvert, Multimap dest)来实现该功能。

NOTE: 如果使用ImmutableMultimap，则考虑使用ImmutableMultimap.inverse()替换.

    ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
    multimap.putAll("b", Ints.asList(2, 4, 6));
    multimap.putAll("a", Ints.asList(4, 2, 1));
    multimap.putAll("c", Ints.asList(2, 5, 3));

    TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap.<String, Integer> create());
    // 注意我们自己选择实现，所有如果用TreeMultimap会得到如下有序的结果：
    `/*
     * inverse maps:
     *  1 => {"a"}
     *  2 => {"a", "b", "c"}
     *  3 => {"c"}
     *  4 => {"a", "b"}
     *  5 => {"c"}
     *  6 => {"b"}
     */`

#### forMap方法

假如想对一个Map使用Multimap的方法该怎么办？forMap(map)视图将一个Map当做SetMultiMap。这会很有用，如，和Multimaps.invertFrom结合使用：

    Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
    SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
    // multimap maps ["a" => {1}, "b" => {1}, "c" => {2}]
    Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.<Integer, String> create());
    // inverse maps [1 => {"a", "b"}, 2 => {"c"}]

#### Wrappers包装

### Tables
与Multimaps.newXXXMultimap(Map, Supplier)工具方法类似， Tables.newCustomTable(Map, Supplier<Map>)允许你指定一个你喜欢的row或者column map的Table实现。

    // use LinkedHashMaps instead of HashMaps
    Table<String, Character, Integer> table = Tables.newCustomTable(
      Maps.<String, Map<Character, Integer>>newLinkedHashMap(),
      new Supplier<Map<Character, Integer>> () {
        public Map<Character, Integer> get() {
          return Maps.newLinkedHashMap();
        }
      });
#### transpose变换
The transpose(Table<R, C, V>) 方法允许你将Table<R, C, V>变换成Table<C, R, V>.

#### Wrappers
大多数情形使用ImmutableTable
