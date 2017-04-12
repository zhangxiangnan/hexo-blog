---
title: hashmap与HashTable源码阅读
tags:
---
### 异同
- 实现map接口定义的所以可选方法，允许key和value为null；和hashtable大致相当于hashtable,只不过hashtable是线程安全地，不允许key和value为null.
- 不保证map的顺序，特别地，不保证顺序一直不变；即不保证添加顺序和遍历顺序一致，不保证添加元素后，其他元素的遍历顺序和上次遍历顺序一致
- 针对基本的get、put操作提供常量时间的性能（hash函数能够将元素均匀分散到各个桶中的前提下）；集合视图的迭代操作耗费时间跟hashMap桶的个数及键值映射的个数成比例，因此，对迭代性能要求高时，不能把初始容量设置太大或者负载因子设置太低很重要。

影响HashMap的性能因素有2个参数：初始容量、负载因子。容量指hash表的桶个个数，初始容量指hash表刚创建时的容量。负载因子是度量hash表容量达到多少时进行自动扩容。当hash表里的entries数量超出负载因子与当前容量之乘积，hash表进行rehash（重建内部数据结构）来达到哈希表的大小是桶数量的2倍。    

一般来说，默认的负载因子为0.75，提供了时间和空间之间的良好折中。 更大的负载因子减少空间开销但是增加查找耗时（体现在HashMap的大部分操作，包括get、put操作）。
map里的预期的entries数量和负载因子大小是设置初始容量时需要考虑的，以便使rehash操作最少。如果初始容量大于(entries的数量除以负载因子），则不会发生rehash操作。

如果HashMap里存储许多键值对映射，一开始创建hashmap时指定足够大的容量相比于执行自动rehash操作来增长表使键值对映射更有效率地存储。注意许多key都有相同的hashcode值的话，会降低hahs表的性能。为了改善性能，当keys可比较comparable时，可以更改键之间的比较顺序。

注意hashmap是非同步的，若多线程并发访问同一个map，并且至少一个线程修改map的结构，必须进行外部同步。（一个结构性修改指任何添加或者删除一个或多个映射的操作；仅仅修改map中已经存在的映射的value值不算结构性修改）通常可以通过锁定封装map的对象来实现。如果没有这样的对象，可使用Collections.synchronizedMap方法，最好创建时处理，来防止偶然的非同步的map访问操作：
     Map m = Collections.synchronizedMap(new HashMap(...));

hashmap的所有集合视图方法返回的迭代器iterators都是快速失败的：如果迭代器创建之后，map任何时候发生结构性改变，用任何方式除了iterator自身的remove方法，iterator都会抛出一个ConcurrentModificationException。因此，针对并发修改，迭代器快速干净地失败而不是在未来未确定的时间冒着任意地未确定行为。

注意迭代器的快速失败行为无法保证，即，在不同步并发修改的情况下，无法做出任何硬性保证。快速失败的迭代器尽力抛出并发修改异常ConcurrentModificationException。因此，迭代器快速失败仅仅用来监测bugs，不能用来依赖异常来保证程序正确性。

Java 集合框架的一个类

- 默认初始容量16，必须是2的多少次幂
- 默认loadfactor为0.75
- 同一个桶里节点数>=8时，进行树化
- 同一个桶里的节点数<=6 ，则在resize时进行非树化操作
- 最小树化容量 64


hash方法的涉及意义

    static final int hash(Object key) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
Computes key.hashCode() and spreads (XORs) higher bits of hash to lower. Because the table uses power-of-two masking, sets of hashes that vary only in bits above the current mask will always collide. (Among known examples are sets of Float keys holding consecutive whole numbers in small tables.) So we apply a transform that spreads the impact of higher bits downward. There is a tradeoff between speed, utility, and quality of bit-spreading. Because many common sets of hashes are already reasonably distributed (so don't benefit from spreading), and because we use trees to handle large sets of collisions in bins, we just XOR some shifted bits in the cheapest possible way to reduce systematic lossage, as well as to incorporate impact of the highest bits that would otherwise never be used in index calculations because of table bounds.


- 长度总是2的次幂，允许为0，桶数组
transient Node<K,V>[] table;

- modCount字段用来表示HashMap被结构性更改的次数，包括改变映射的数目，或者改变内部结构（如rehash），该方法用来使HashMap的集合视图上的迭代器快速失败

-     int threshold; resize操作执行的阈值（loadfactor * capacity)
