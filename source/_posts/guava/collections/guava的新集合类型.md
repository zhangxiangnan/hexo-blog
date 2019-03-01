layout: title
title: guava的新集合类型
date: 2016-10-21 13:20:15
tags:
- guava
- newCollectionsTypes
categories:
- 译
- guava
---

guava的新集合类型Multiset、Multimap、BiMap、Table、ClassToInstanceMap、RangeSet
<!-- more -->
Guava介绍了许多jdk中没有的但是很有用处的新集合类型，这些新集合类型和JDK集合框架和平共处，并且没有硬塞进任何东西到JDK集合抽象类中。
一般来说，Guava的集合实现严格遵循了JDK接口契约。

### Multiset
传统的JAVA惯例手法来统计一文档中的单词出现频率，类似如下：
```
    Map<String, Integer> counts = new HashMap<String, Integer>();
    for (String word : words) {
      Integer count = counts.get(word);
      if (count == null) {
        counts.put(word, 1);
      } else {
        counts.put(word, count + 1);
      }
    }
```
这种方式笨拙，容易出错，且不支持收集各种有用的统计信息，如单词总数，而Guava做的更好。
Guava提供一个新集合类型，Multiset，支持元素的多次添加，数学中的multiset定义为元素可以出现不止一次的set。在multiset中，类似sets和tuples，元素是无序的，如multisets {a, a, b} 和 {a, b, a} 是equal的。

有2种方式理解这个概念：
  - 类似一个没元素顺序的约束的ArrayList<E>
  - 类似一个Map<E, Integer>,key、value分别表示元素和元素出现的次数

Guava的Multiset API结合了这2种方式，如下：
  - 当以正常集合对待它，Multiset表现的更像是一个无序的ArrayList：
    - 调用add(E),则该元素出现的次数加1.
    - Multiset的iterator()方法迭代每一个元素的每一次出现。
    - size()方法是所有元素的所有出现次数的总和。
  - 额外的query操作，及性能特性，比较像一个Map<E, Integer>.
    - count(Object)方法返回指定元素的出现次数。HashMultiset的count方法效率是O(1)，TreeMultiset的count方法效率是O(log n)等。
    - entrySet() 类似Map的entrySet。
    - elementSet()返回multiset的一个去重的元素集合`Set<E>`，类似于Map的keySet()
    - 针对不重复的元素来说，Multiset实现类内存消耗（空间复杂度）随着不重复元素的数目线性增长,即重复存储相同对象。

尤其需要注意，Multiset是和JDK集合Collection的接口规范完全一致的，除了JDK自身的早期版本的极少见情形 -- 特别的，TreeMultiset，类似TreeSet，使用comparison来比较是否相等，而不是Object的equals方法. 特别地，Multiset.addAll(Collection)在集合中的每个元素每出现一次，该元素的出现次数加一，该方式比上面的使用Map+for循环更简便。

其他方法说明如下：

方法   | 描述
--------  | ---
count(E) |	对添加到multiset的元素出现次数进行计数
elementSet()	| 返回Multiset<E>的去重元素的集合(一个`Set<E>`)
entrySet() |	类似于Map.entrySet(),返回`Set<Multiset.Entry<E>>`，一个包含支持getElement()和getCount()方法的entries的set集合
add(E, int)	| 给指定元素添加指定数目的出现次数
remove(E, int) |	移除指定元素的指定数目的出现次数
setCount(E, int)	| 设置指定元素的出现次数为非负值
size()	| 返回Multiset中所有元素的总共的出现次数和。

#### Multiset不是一个Map
注意Multiset<E>不是一个Map<E, Integer>，尽管Multiset实现有这部分的功能。Multiset是一个真正的集合类型， 符合所有的相关的接口协议，其他值得注意的区别如下：

  - 一个Multiset<E>的元素只能有正数的count计数，不能是负数。计数0不认为在multiset中，在elementSet()、entrySet视图中计数为0的元素会被过滤。
  - multiset.size() 返回集合的大小，就是集合中所有元素的计数的总和。如果想要得到去重后元素的数目（不重复元素的数目），使用elementSet().size()得到。(所以，例如，add(E)方法会使multiset.size()的数目加一)
  - multiset.iterator() 会遍历每一个元素的每一次出现，所以迭代的次数等于multiset.size()
  - `Multiset<E>` 支持添加元素，移除元素，或者直接设置元素的计数count。setCount(elem, 0)等价于移除该元素的所有次数的出现。
  - multiset.count(elem) 对于不在multiset中的元素始终返回0

#### 实现类
Guava提供许多Multiset的实现，大体上对应于JDK的map实现类。

Map	| Corresponding Multiset	| Supports null elements
- | - | -
HashMap	| HashMultiset | Yes
TreeMap	| TreeMultiset  | Yes (if the comparator does)
LinkedHashMap |	LinkedHashMultiset|	Yes
ConcurrentHashMap	|ConcurrentHashMultiset	| No
ImmutableMap|	ImmutableMultiset	| No


### SortedMultiset
SortedMultiset
SortedMultiset是Multiset接口的一个变种，支持高效地获取指定范围的子multisets。例如你可以使用latencies.subMultiset(0, BoundType.OPEN, 100, BoundType.OPEN).size()来获取你的网站100ms下的延迟的请求的子集，然后和latecies.size()比较来获取百分比。

TreeMultiset实现了SortedMultiset接口。ImmutableSortedMultiset目前正在测试GWT的兼容性。

### Multimap  
每一个有经验的java程序员很可能自己实现了`Map<K, List<V>>`或者`Map<K, Set<V>>`，或者直接使用那种笨拙的结构。Guava的Multimap框架使处理从keys到多个值得映射关系变得简单。一个Multimap是一个普遍的方式来关联keys和任意多values。

从概念上有2种方式来理解Multimap：单个key到单个value的映射集合：
```
    a -> 1 a -> 2 a -> 4 b -> 3 c -> 5
```
或者是唯一的单个key到values集合的映射关系：
```
    a -> [1, 2, 4] b -> [3] c -> [5]
```
一般来说, Multimap接口是理解这种观念最好的视图，但是也允许你使用asMap()以另一种方式来看，asMap()返回`Map<K, Collection<V>>`。    
最重要的是， 不会出现一个key对应一个空集合的情况：一个key要么映射到至少一个value，要么在Multimap中该key不存在。   
如果你想区分key存在但是没对应的values和key就不存在这2种情况，更适合的数据结构很可能是Graph图（支持孤立点）。

我们很少直接使用Multimap接口，更多的是，我们会使用ListMultimap或者SetMultimap，相应地keys映射到List和Set。

#### Modifying 修改API

Multimap.get(key)返回和给定key相关联的values视图，即时目前什么也没有。对于ListMultiMap返回一个list，SetMultimap返回一个set。

修改操作直接对底层的Multimap进行写。如：    
```
    Set<Person> aliceChildren = childrenMultimap.get(alice);
    aliceChildren.clear();
    aliceChildren.add(bob);
    aliceChildren.add(carol);
```
对底层的MultiMap直接进行写。

其他修改multimap的方式（更直接）包括：

方法	| 描述 |	等同于
- | - | - |
put(K, V)	| 添加key到value的映射|	multimap.get(key).add(value)
`putAll(K, Iterable<V>`) |	依次添加key到每一个value的映射	| Iterables.addAll(multimap.get(key), values)
remove(K, V)	| 移除key到value的映射，若multimap改变了则返回true，否则false	| multimap.get(key).remove(value)
removeAll(K)	| 移除所有与key映射的所有value，返回的集合可以修改，但是修改它不会影响multimap (返回适合的集合类型)|	multimap.get(key).clear()
`replaceValues(K, Iterable<V>`)	| 清空key到所有value的映射，然后设置key到每个给定value的映射。返回被清空的key关联的vlaues集合 |multimap.get(key).clear(); Iterables.addAll(multimap.get(key), values)

#### views视图

MultiMap支持一系列强大的视图：
- asMap方法将所有的`Multimap<K,V>当做一个Map<K,Collection>>`，其返回值支持remove、并且对返回集合的改变会影响到multimap，但是其不支持put或者putall。更关键的是，当你想要对于不存在的key返回null时你可以使用asMap().get(key)，而不是返回一个干净的空集合。(你应当强转asMap.get(key)的返回值到合适的集合类型-- 对应SetMultiMap的是set，ListMultiMap的List -- 但是JDK的类型系统不允许ListMultiMap返回`Map<K, List<V>>`)
- entries方法返回MultiMap所有的entries为一个`Collection<Map.Entry<K, V>>`（对SetMultiMap来说，Collection实际上是个Set)
- keySet将MultiMap中所有的key去重，返回一个set。
- keys方法将MultiMap中的所有keys看成一个MultiSet，并且和key相关联的values有着相同的多样性。 元素可以从MultiSet移除，但是不能添加；改变会导致原MultiMap改变。
- values()方法返回MultiMap里所有的values为一个多重集的`Collection<V>`，所有的value都在这一个Collection。类似于Iterables.concat(multimap.asMap.values())

#### Multimap不仅仅是map

A `Multimap<K, V>` 不仅仅是一个`Map<K, Colleciton<V>>`，尽管这样的一个map可能被Multimap的某个实现使用，显著区别如下：

- Multimap.get(key) 始终返回一个非null的，可能的空集合. 这并不意味着multimap花费了任何内存关联那个key，相反，返回的集合是一个允许你添加key映射的视图。
- 如果更倾向于类似Map的行为，如对于multimap中不存在的key返回null，可以使用asMap()视图来得到一个`Map<K,Collection<V>>`。（或者，从ListMultiMap得到一个`Map<K,List<v>>`，使用静态Multimaps.asMap()方法，类似方法适用于SetMultimap、SetMultimap、SortedSetMultimap)
- Multimap.containsKey(key)仅仅当有任何元素和key相关联时才返回true。特别地，如果一个k以前在multimap存在映射关系，但是后来移除映射关系了，该方法返回false
- Multimap.entries()返回multimap中的所有key的所有entries。如果想要得到所有key-collection entires，使用asMap.entrySet().
- Multimap.size()返回MultiMap中所有的entries数目，不是对key去重后的数目。使用 Multimap.keySet().size() 可以得到去重后的key数目

#### 实现类

Multimap提供很多实现，可以在大多数想使用`Map<K, Colleciton<V>>`的场合使用。

实现 |	Key行为类似	| Values行为类似
- | - |
ArrayListMultimap	| HashMap |	ArrayList，同key对应values有顺序可重复
HashMultimap	| HashMap|	HashSet,同key对应values不可重复
LinkedListMultimap	|LinkedHashMap	|LinkedList 同key对应value有顺序可重复
LinkedHashMultimap	|LinkedHashMap	|LinkedHashSet 同key对应value有顺序不重复
TreeMultimap	|TreeMap	|TreeSet，同key对应的value有顺序
ImmutableListMultimap	|ImmutableMap	|ImmutableList
ImmutableSetMultimap	|ImmutableMap	|ImmutableSet

每一个实现，除了不可变的那些，都支持key和value为null。

  - LinkedListMultimap.entries()，对于不重复的key value保持了迭代时的顺序。

  - LinkedHashMultimap，保持了映射项entries的插入顺序，包括键keys的插入顺序、和任意一个key关联的values所有值的插入顺序。

要知道以上的所有实现中并不是所有的实现都通过`Map<K, Collection<V>>`（特别的，几个MultiMap实现使用了自定义hash表来最小化开销）

如果你想要更多自定义，使用MultiMaps.newMultimap(Map, Supplier<Collection)或者list、set自定义实现来支撑自定义multimap。


### BiMap 双向Map

传统映射values到key的方式是维护2个独立的map，保持他们同步，但是这种方式当一个value已经在map里存在时令人极度困惑且感觉像是bug。如：   
```
    Map<String, Integer> nameToId = Maps.newHashMap();
    Map<Integer, String> idToName = Maps.newHashMap();

    nameToId.put("Bob", 42);
    idToName.put(42, "Bob");
    // 如果“Bob”或42已经存在，当我们刚好又忘记保持他们同步的时候，就会发生奇怪的错误。
```
A `BiMap<K, V>` 是一个`Map<K,V>`：
  - 允许你使用inverse()方法来得到反转视图BiMap<V,K>
  - 确保values是不重复的，使values类似一个Set

所有，BiMap的特点是key、value都不能重复。
若尝试添加一个key到已经存在的value映射会报参数异常。若想删除预先存在的entry（指定的value），则使用BiMap.forcePut(key, value)。
```
    public static void main(String[] args) {
           BiMap<String, String> biMap = HashBiMap.create();
           biMap.put("1", "2");
           biMap.put("2", "2");
           System.out.println(biMap);
       }
       #报参数异常

       public static void main(String[] args) {
               BiMap<String, String> biMap = HashBiMap.create();
               biMap.put("1", "2");
               String s = biMap.inverse().get("1");
               System.out.println(s);
               System.out.println(biMap.inverse().get("2"));
           }
           #null
           #1
```
#### 实现

Key-Value Map实现类 |	Value-Key Map 实现类 |	对应BiMap
-|
HashMap|	HashMap|	HashBiMap
ImmutableMap|	ImmutableMap|	ImmutableBiMap
EnumMap	|EnumMap	|EnumBiMap
EnumMap	|HashMap	|EnumHashBiMap
Note: BiMap的工具方法如synchronizedBiMap在Maps里实现。

### 表格Table
```
    Table<DateOfBirth, LastName, PersonalRecord> records = HashBasedTable.create();
    records.put(someBirthday, "Schmo", recordA);
    records.put(someBirthday, "Doe", recordB);
    records.put(otherBirthday, "Doe", recordC);

    records.row(someBirthday); // returns a Map mapping "Schmo" to recordA, "Doe" to recordB
    records.column("Doe"); // returns a Map mapping someBirthday to recordB, otherBirthday to recordC
```
通常，当你想同一时间对不止一个的key检索的时候，你会想出使用类似`Map<FirstName, Map<LastName, Person>>`，这种方式丑陋难以使用。 Guava提供一个新的集合类型，表格Table，支持“行row”类型，和“column”列类型。表格Table支持很多种视图来让你可以使用从任何角度的数据，包括：

  - rowMap(), 该方法将`Table<R,C,V>`视作`Table<R,Map<C,V>>`。类似rowKeySet返回一个Set<R>.
  - row(r)返回一个非null的Map<C,V>，对这个Map的写操作将穿透到底层Table。
  - 相似的列方法提供如下： columnMap(), columnKeySet(), column(c). (基于column的访问稍微比基于行的访问效率低)
  - cellSet() 返回一个Table.Cell<R,C,V>的set集合，Cell很像Map.Entry，但是区分row和column key。

几种Table实现如下：

  - HashBasedTable, 底层使用`HashMap<R,HashMap<C,V>>`（Guava20.0使用LinkedHashMap<R, LinkedHashMap<C,V>>实现）。
  - TreeBasedTable,底层使用`TreeMap<R, TreeMap<C, V>>`.
  - ImmutableTable,底层使用`ImmutableMap<R, ImmutableMap<C, V>>`. (Note: ImmutableTable针对稀疏和稠密的数据集有单独的优化实现)
  - ArrayTable, 需要在构建的时候传递全部的row和column数据，当数据是紧密的数据时通过2维数组来提高速度、内存利用率。ArrayTable和其他实现原理有稍微不同.

### ClassToInstanceMap 类型到实例的映射Map

有时，map的key并不都是相同的类型，可能有多个类型，你可能想映射类型到values，可以使用ClassToInstanceMap.   
除了扩展Map接口，ClassToInstanceMap提供方法T `getInstance(Class<T>`) 和T putInstance(Class<T>, T)，省去了手动进行类型转换。

ClassToInstanceMap只有一个名称为B的类型参数，代表map管理的类型的上层限制，如：
```
    ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
    numberDefaults.putInstance(Integer.class,Integer.valueOf(0);
```
严格说，ClassToInstanceMap<B>实现了接口`Map<Class<? extends B>, B>` -- 或者是从B的子类型class到B的映射的一个map，这会令ClassToInstanceMap的反省类型有些令人困惑，但记住B始终是map中类型的上层限制（上界）--通常，B就是Object类。

Guava提供MutableClassToInstanceMap、ImmutableClassToInstanceMap。

注意: 像其他Map<Class, Object>，一个ClassToInstanceMap可能包含基本类型，基本类型和对应包装类型可能对应不同的values。

### RangeSet

A RangeSet描述了一个不连续的、非空的范围集合，当往一个可变的RangeSet添加一个范围的时候，任何可连续的范围都被合并到一起，空范围被忽略，如下：
```
    RangeSet<Integer> rangeSet = TreeRangeSet.create();
     rangeSet.add(Range.closed(1, 10)); // {[1, 10]}
     rangeSet.add(Range.closedOpen(11, 15)); // disconnected range: {[1, 10], [11, 15)}
     rangeSet.add(Range.closedOpen(15, 20)); // connected range; {[1, 10], [11, 20)}
     rangeSet.add(Range.openClosed(0, 0)); // empty range; {[1, 10], [11, 20)}
     rangeSet.remove(Range.open(5, 10)); // splits [1, 10]; {[1, 5], [10, 10], [11, 20)}
```
记住想要合并范围如Range.closed(1, 10) and Range.closedOpen(11, 15), 必须首先使用Range.canonical(DiscreteDomain)预处理范围，如使用DiscreteDomain.integers().
```
    RangeSet<Integer> rangeSet = TreeRangeSet.create();
        rangeSet.add(Range.closed(1, 10).canonical(DiscreteDomain.integers())); // {[1, 10]}
        rangeSet.add(Range.closedOpen(11, 15)); // disconnected range: {[1, 10], [11, 15)}
        System.out.println(rangeSet);//[[1‥15)]

        // canonical可以理解为规范化，效果如下：
        System.out.println(Range.closed(1, 10).canonical(DiscreteDomain.integers()));[1‥11)
        System.out.println(Range.closed(1, 10));[1‥10]
```
注意: RangeSet 在GWT下和JDK1.5之下都不被支持；RangeSet需要全部NavigableMap的特征在JDK1.6中。

#### 视图
RangeSet实现支持很多视图：

  - complement(): 返回补集视图，也是一个RangeSet，它包含不连续的、非空范围.
  - subRangeSet(`Range<C>`): 返回RangeSet和指定范围的range的交叉集合，这会返回惯例的排序的headSet、subSet、tailSet视图。
  - asRanges():将RangeSet视作`Set<Range<C>>`，可以迭代遍历。
  - asSet(`DiscreteDomain<C>`) (仅仅对ImmutableRangeSet而言): 将`RangeSet<C>`当做`ImmutableSortedSet<C>`看待ranges中的元素而非range自己. (这个操作如果DiscreteDomain和RangeSet同时是无穷大或无穷小时不支持)

#### 查询

除了视图，RangeSet还直接支持几种查询操作 ，最常用的：

  - contains(C): RangeSet的最基本的方法，查询RangeSet中的任意一个range是否包含指定元素。
  - rangeContaining(C): 返回包含指定元素的Range，无返回null。
  - encloses(`Range<C>`): 测试RangeSet中的任意Range是否包围指定Range
  - span(): 返回包围RangeSet中任意的Range的最小Range。

### RangeMap

RangeMap是描述不相交、非空范围到values的映射关系。不同于RangeSet，RangeMap不合并相邻映射，即使相邻的Ranges映射到相同的values。如：
```
    RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
    rangeMap.put(Range.closed(1, 10), "foo"); // {[1, 10] => "foo"}
    rangeMap.put(Range.open(3, 6), "bar"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo"}
    rangeMap.put(Range.open(10, 20), "foo"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo", (10, 20) => "foo"}
    rangeMap.remove(Range.closed(5, 11)); // {[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"}
    Views
```
RangeMap提供2中视图：

  - asMapOfRanges(): 将RangeMap当做一个`Map<Range<K>,V>`，当想要迭代RangeMap时可以使用
  - subRangeMap(`Range<K>`) 将RangeMap和指定Range的交集当做一个RangeMap，惯例返回headMap, subMap, and tailMap.
