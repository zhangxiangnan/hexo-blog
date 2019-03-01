layout: title
title: lambda表达式入门
date: 2018-02-05 19:47:07
tags:
- lambda
categories:
- lambda
---

lambda表达式入门学习

<!-- more -->

#### lambda出现的背景
如果匿名类的实现起来非常简单,如只有一个方法的接口,那么匿名类的语法使用起来看着会不清晰。

这些情形,看起来像是把功能当做参数传递到其他方法,如当点击按钮时应该执行的操作。Lambda表达式让我们能够把功能/函数/方法当做参数,或者代码当做数据来传递。

尽管匿名类比非匿名类简洁,但是对于只有一个方法的类,匿名类仍然有些烦琐。Lambda表达式让我们可以更简洁地表达单方法类的实例对象。

#### lambda表达式语法
lambda表达式如(p,q) -> System.out.print("x"), 包含如下几部分:
 - 圆括号里逗号分隔的一个或多个形参列表,注意:
    - 形参的数据数据类型可以省略不写
    - 当只有一个形参时,圆括号也可以省略不写,如p -> System.out.print("x")
 - 箭头标记
 - 一个单独表达式或者代码块,如:

```
        p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25

```

如果是一个单独的表达式,jvm计算表达式的值并返回;此外,也可以使用return语句:

```
        p -> {
            return p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25;
        }

```

return语句不是一个表达式。在lambda表达式里,必须使用{}把return的语句块包起来,但若{}包的语句块中的方法为void方法,则可以省略{},如:

```
        email -> System.out.println(email)
```

lambda表达式看着很像一个方法声明,可以将lambda表达式认为是匿名的方法调用。

lambda表达式也可以有多个形参,如:

```
        public class Calculator {
            interface IntegerMath {
                int operation(int a, int b);   
            }
            public int operateBinary(int a, int b, IntegerMath op) {
                return op.operation(a, b);
            }
            public static void main(String... args) {
                Calculator myApp = new Calculator();
                IntegerMath addition = (a, b) -> a + b;
                IntegerMath subtraction = (a, b) -> a - b;
                System.out.println("40 + 2 = " + myApp.operateBinary(40, 2, addition));
                System.out.println("20 - 10 = " + myApp.operateBinary(20, 10, subtraction));    
            }
        }
```

#### lambda表达式理想用例
##### 创建找出符合某一个特征的人员的搜索方法(找出年龄大于指定值的人?)

```
        public static void printPersonsOlderThan(List<Person> roster, int age) {
            for (Person p : roster) {
                if (p.getAge() >= age) {
                    p.printPerson();
                }
            }
        }
```

思考: 不支持泛型,改变Person类型就需要重写方法&只支持大于某个年龄,想要找出小于某个年龄的就..

##### 创建更通用的搜索方法
```
        public static void printPersonsWithinAgeRange(
            List<Person> roster, int low, int high) {
            for (Person p : roster) {
                if (low <= p.getAge() && p.getAge() < high) {
                    p.printPerson();
                }
            }
        }
```
思考:支持小于某个年龄,但是假如想找出某个指定性别、或者指定性别结合年龄条件? 假如Person类增加了关系、地域等属性呢?每种属性定义一个搜索方法也可以,但会散乱,
可以将搜索条件单独隔离到其他方法里。

##### 在局部类中指定搜索条件的代码
```
        public static void printPersons(
            List<Person> roster, CheckPerson tester) {
            for (Person p : roster) {
                if (tester.test(p)) {
                    p.printPerson();
                }
            }
        }

        interface CheckPerson {
            boolean test(Person p);
        }

        class CheckPersonSelectiveService implements CheckPerson {
            public boolean test(Person p) {
                return p.gender == Person.Sex.MALE &&
                    p.getAge() >= 18 &&
                    p.getAge() <= 25;
            }
        }
```
  调用:
```
        printPersons(
            roster, new CheckPersonEligibleForSelectiveService());
 ```
  思考:改变person类型不需要重写printPersons方法,但引入一个接口及每一种搜索一个局部实现类,可以将局部实现类改为匿名类。

##### 匿名类中指定搜索条件代码

```
        printPersons(
            roster,
            new CheckPerson() {
                public boolean test(Person p) {
                    return p.getGender() == Person.Sex.MALE
                        && p.getAge() >= 18
                        && p.getAge() <= 25;
                }
            }
        );
```
  思考:改为了匿名类,省去局部类的定义,但是匿名类仍然较烦琐,由于CheckPerson接口只有一个方法,则可以考虑使用lambda表达式替换

##### 使用lambda表达式替换搜索代码
CheckPerson是一个函数接口,函数接口是仅有一个抽象方法的任何接口(可能包含多个静态方法或默认方法)。由于函数接口仅有一个方法,因为实现该接口时可以忽略方法名称,如下:
```
        printPersons(
            roster,
            (Person p) -> p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25
        );
```
思考:可以使用标准函数接口来进一步减少代码

##### 使用lambda表达式的标准函数接口
由于CheckPerson接口仅有一个方法,返回bool值,很简单,不必要在程序中自己定义,所以JDK在java.util.function中提供了若干标准函数接口,可以使用Predicate<T>替换CheckPerson:
```
        interface Predicate<T> {
            boolean test(T t);
        }
```
  这是一个泛型接口,替换CheckPerson,如下:
```
        public static void printPersonsWithPredicate(
            List<Person> roster, Predicate<Person> tester) {
            for (Person p : roster) {
                if (tester.test(p)) {
                    p.printPerson();
                }
            }
        }
```
  经过匿名类替换+使用lambda表达式,调用方式如下:
```
        printPersonsWithPredicate(
            roster,
            p -> p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25
        );
```
##### 整个程序中使用lambda表达式
 再考虑这个方法:
```
        public static void printPersonsWithPredicate(
            List<Person> roster, Predicate<Person> tester) {
            for (Person p : roster) {
                if (tester.test(p)) {
                    p.printPerson();
                }
            }
        }
```
 p.printPerson()是一个void方法,对p操作,假如不是执行printPerson操作而是执行其他类似void操作,那么就需要修改printPersonsWithPredicate方法,考虑将这个操作使用
  lambda表达式,JDK提供了Consumer<T>标准函数接口,可改进如下:

        ```
          public static void processPersons(
              List<Person> roster,
              Predicate<Person> tester,
              Consumer<Person> block) {
                  for (Person p : roster) {
                      if (tester.test(p)) {
                          block.accept(p);
                      }
                  }
          }
        ```

 调用:

    ```
        processPersons(
             roster,
             p -> p.getGender() == Person.Sex.MALE
                 && p.getAge() >= 18
                 && p.getAge() <= 25,
             p -> p.printPerson()
        );
    ```
 假如想验证人员身份或获取联系信息呢?需要一个函数接口,其有一个抽象方法,返回一个值,The Function<T,R>标准接口有一个R apply(T t)方法,应用到如下示例:

```
        public static void processPersonsWithFunction(
            List<Person> roster,
            Predicate<Person> tester,
            Function<Person, String> mapper,
            Consumer<String> block) {
            for (Person p : roster) {
                if (tester.test(p)) {
                    String data = mapper.apply(p);
                    block.accept(data);
                }
            }
        }
```
调用(获取每一个人的邮箱信息,并打印):

```
        processPersonsWithFunction(
            roster,
            p -> p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25,
            p -> p.getEmailAddress(),
            email -> System.out.println(email)
        );
```
##### 使用泛型使扩展性更强
processPersonsWithFunction方法的泛型版本如下:

    ```
        public static <X, Y> void processElements(
            Iterable<X> source,
            Predicate<X> tester,
            Function <X, Y> mapper,
            Consumer<Y> block) {
            for (X p : source) {
                if (tester.test(p)) {
                    Y data = mapper.apply(p);
                    block.accept(data);
                }
            }
        }
    ```
  调用:
```
        processElements(
            roster,
            p -> p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25,
            p -> p.getEmailAddress(),
            email -> System.out.println(email)
        );
```
#### 聚合操作
如下聚合操作:
```
        roster
            .stream()
            .filter(
                p -> p.getGender() == Person.Sex.MALE
                    && p.getAge() >= 18
                    && p.getAge() <= 25)
            .map(p -> p.getEmailAddress())
            .forEach(email -> System.out.println(email));
```
 filter、map、foreach都是聚合操作,聚会操作从stream流中获取元素,而不是从集合中。

 流是一系列元素,跟集合不同,流不是存储元素的数据结构,流从数据源如集合中通过管道/流水线传输元素。
 管道/流水线是一系列流操作,一系列聚合操作,如filter-map-foreach。聚合操作通常接受lambda表达式作为参数,来自定义行为。

管道/流水线包含以下几部分:
   - 数据源
     可以是一个集合、数组、一个生成器函数、I/O channel等
   - 0步或多步中间操作
     如filter就是中间操作,生成一个符合过滤条件的元素组成的新流
   - 一个结束操作
     如forEach生成一个非流的结果,如void、基本类型数据、一个集合。
##### 聚合操作与迭代的区别
聚合操作如forEach,与迭代器看起来类似,但是有一个根本区别:
   - 聚合操作使用内部迭代
     聚合操作使用内部迭代,只需要提供迭代集合,内部迭代可以是顺序迭代,也可以是并行迭代(并行计算,合并结果);迭代器是外部迭代,需要提供迭代集合与迭代方式,是顺序迭代方式。
   - 聚合操作从流获取数据来处理
     聚合操作从流获取数据,非集合,所以聚合操作也称为流操作。
   - 支持传递代码/行为为参数(lambada表达式作为参数)
#### 归约操作
```
        double average = roster
            .stream()
            .filter(p -> p.getGender() == Person.Sex.MALE)
            .mapToInt(Person::getAge)
            .average()
            .getAsDouble();
```
JDK有很多结束操作,如average、sum、min、max、count,这些操作通过组合流的元素内容来返回一个值,这些操作叫做归约操作(reduction operations)。
JDK也有返回一个集合的归约操作,很多归约操作都是执行一个指定的任务,如计算平均值,元素分组。JDK提供了通用的归约操作,Stream.reduce&Stream.collect

##### Stream.reduce
Stream.reduce是通用的归约操作,如下例则使用了Stream.sum 归约操作:
```
        Integer totalAge = roster
            .stream()
            .mapToInt(Person::getAge)
            .sum();
```
使用Stream.reduce实现相同的逻辑如下:
```
        Integer totalAgeReduce = roster
           .stream()
           .map(Person::getAge)
           .reduce(
               0,
               (a, b) -> a + b);
```
reduce操作有2个参数:
```
    - 第一个为标识元素
        是归约的初始值和当流中无元素时的默认返回值
    - 累加器函数
        累加器函数有2个参数,归约的部分结果值和流中的下一个要计算的元素
```
reduce操作每次都返回一个新值,累加器每次处理完一个元素后也是返回一个新值,所以如果想要归约流的元素为一个更复杂的对象,如集合,由于每次会返回一个新的集合,性能低下,可以用Stream.collect

##### Stream.collect
reduce方法每处理一个元素会新建一个值,collect方法则是修改、转换一个已经存在的值。考虑如何计算流中元素的平均值?需要两份数据,一个是所有元素的个数,一个是其和,和reduce方法以及其他所有
归约方法一样,collect也只返回一个值,可以创建一个新的数据类型,包含这两个计数,如下:
```
        class Averager implements IntConsumer {
            private int total = 0;
            private int count = 0;

            public double average() {
                return count > 0 ? ((double) total)/count : 0;
            }

            public void accept(int i) { total += i; count++; }

            public void combine(Averager other) {
                total += other.total;
                count += other.count;
            }
        }


        Averager averageCollect = roster.stream()
            .filter(p -> p.getGender() == Person.Sex.MALE)
            .map(Person::getAge)
            .collect(Averager::new, Averager::accept, Averager::combine);

        System.out.println("Average age of male members: " +
            averageCollect.average());
```
该例collect方法有三个参数:
```
    - supplier
        结果容器创建函数,工厂方法,为collect操作创建一个新实例,创建结果容器的实例,本例则是创建Averager的实例。
    - accumulator
        累加器函数合并一个流元素到结果容器中,本例中,即修改Averager的2个计数
    - combiner
        合并函数将两个结果容器的内容进行合并,本例中,为对两个Averager容器中的2个计数进行合并相加。
```
注意:
```
    - supplier是一个lambda表达式或者方法引用,而非reduce操作中的identity元素是一个值。
    - accumulator累加器和combiner合并函数不返回值
    - 可以结合并行流使用collect操作
```
尽管JDK提供了average操作来计算平均值,当我们需要计算流中元素的几个值时就可以使用collect操作+自定义类。
collect操作最适合集合,如下:
```
        List<String> namesOfMaleMembersCollect = roster
            .stream()
            .filter(p -> p.getGender() == Person.Sex.MALE)
            .map(p -> p.getName())
            .collect(Collectors.toList());
```
该例collect只需要一个Collector类型参数,该类封装了collect操作所需的三个参数(supplier、accumulator、combiner),Collectors类包含许多有用的操作,如累加元素到集合,根据不同条件分类元素等。
Collectors.toList返回的Collector会累加流元素到一个新list。

按性别分类:
```
            Map<Person.Sex, List<Person>> byGender =
                roster
                    .stream()
                    .collect(
                        Collectors.groupingBy(Person::getGender));
```
groupingBy方法根据指定的分类函数(getGener)来分类,返回的map的key为getGender的返回值。

获取集合中的成员姓名并按性别分类:
```
        Map<Person.Sex, List<String>> namesByGender =
            roster
                .stream()
                .collect(
                    Collectors.groupingBy(
                        Person::getGender,                      
                        Collectors.mapping(
                            Person::getName,
                            Collectors.toList())));
```
本例的groupingBy方法有2个参数,一个分类函数,一个Collector实例,Collector参数称为下游收集器,该收集器应用到另一个收集器的结果。

因此,groupingBy操作使你可以应用collect方法到groupingBy操作创建的List结果值上。

该例中,collector mapping将mapping函数应用到流中的每一个元素上,返回的结果流由会员名称组成;包含一个或多个下流收集器的流水线如本例称作多级归约。

如下例,获取每一种性别的会员的年龄总和:
```
        Map<Person.Sex, Integer> totalAgeByGender =
            roster
                .stream()
                .collect(
                    Collectors.groupingBy(
                        Person::getGender,                      
                        Collectors.reducing(
                            0,
                            Person::getAge,
                            Integer::sum)));
```
Collectors.reducing归约操作有三个参数:

- identity:
   如Stream.reduce操作一样,identity是归约的初始值和默认值(如果流中没有元素),上例中,identity为0
- mapper:
  归约操作将mapper函数应用到所有流元素,本例是获取每个成员的年龄
- operation:
  操作函数用来归约映射的值,本例是对年龄求和

下例是获取每种性别的会员的平均年龄:
```
        Map<Person.Sex, Double> averageAgeByGender = roster
            .stream()
            .collect(
                Collectors.groupingBy(
                    Person::getGender,                      
                    Collectors.averagingInt(Person::getAge)));
```
#### 并行
并行计算涉及到将一个问题分为多个子问题,同时解决子问题(并行中,每一个子问题在一个独立线程中运行),然后合并各个子问题的结果。
JavaSE提供了fork/join框架,可以让我们很方便地实现并行计算,然而用这个框架的话,必须指定如何划分问题,通过聚合操作就可以进行问题划分和结果合并。

对于使用集合的情形,并行计算的一个难点就是操作集合时的线程安全问题,虽然集合框架的同步方法是线程安全的,但是会导致线程竞争,达不到并行的目的。
聚合操作以及并行流使我们能够实现提供的非线程安全性集合的并行性,在操作集合时不需要修改集合。
注意,并行并不一定比串行快,当有足够的数据和处理器核心时可能并行更有优势。所以,即使聚合操作让我们更容易实现并行计算,但是也要考虑是否适合并行。

##### 并行执行流操作
流可以串行执行,也可以并行执行,当并行执行流时,jvm将流分为多个子流,聚合操作并行的迭代处理这些流,并把结果合并。
当创建一个流时,默认是串行流,除非使用Collection.parallelStream或者BaseStream.parallel指定为并行流。如下例,并行计算男士的平均年龄:
```
        double average = roster
            .parallelStream()
            .filter(p -> p.getGender() == Person.Sex.MALE)
            .mapToInt(Person::getAge)
            .average()
            .getAsDouble();
```
##### 并发归约
再考虑如下例子:
```
        Map<Person.Sex, List<Person>> byGender =
            roster
                .stream()
                .collect(
                    Collectors.groupingBy(Person::getGender));
```
对应的并行代码:
```
        ConcurrentMap<Person.Sex, List<Person>> byGender =
            roster
                .parallelStream()
                .collect(
                    Collectors.groupingByConcurrent(Person::getGender));

```
上例称作并发归约,jvm在以下条件都为真时会为包含collect操作的指定流水线执行并发归约:
- 流是并行流
- collect操作的collector参数的Collector.Characteristics为并发模式,通过Collector.characteristics方法设置
- 无论流式无序,或者collector设置了Collector.Characteristics.UNORDERED;确保流式无序的,可以通过BaseStream.unordered操作

##### 排序
pipeline处理流中元素的顺序依赖于流是串行执行还是并行执行,以及流的数据源本身和中间操作,如下例:
```
        Integer[] intArray = {1, 2, 3, 4, 5, 6, 7, 8 };
        List<Integer> listOfIntegers =
            new ArrayList<>(Arrays.asList(intArray));

        System.out.println("listOfIntegers:");
        listOfIntegers
            .stream()
            .forEach(e -> System.out.print(e + " "));
        System.out.println("");

        System.out.println("listOfIntegers sorted in reverse order:");
        Comparator<Integer> normal = Integer::compare;
        Comparator<Integer> reversed = normal.reversed();
        Collections.sort(listOfIntegers, reversed);  
        listOfIntegers
            .stream()
            .forEach(e -> System.out.print(e + " "));
        System.out.println("");

        System.out.println("Parallel stream");
        listOfIntegers
            .parallelStream()
            .forEach(e -> System.out.print(e + " "));
        System.out.println("");

        System.out.println("Another parallel stream:");
        listOfIntegers
            .parallelStream()
            .forEach(e -> System.out.print(e + " "));
        System.out.println("");

        System.out.println("With forEachOrdered:");
        listOfIntegers
            .parallelStream()
            .forEachOrdered(e -> System.out.print(e + " "));
        System.out.println("");


        listOfIntegers:
        1 2 3 4 5 6 7 8
        listOfIntegers sorted in reverse order:
        8 7 6 5 4 3 2 1
        Parallel stream:
        3 4 1 6 2 5 7 8
        Another parallel stream:
        6 3 1 5 7 8 4 2
        With forEachOrdered:
        8 7 6 5 4 3 2 1
```
#### 副作用
一个方法或一个表达式除了返回值及生成一个值,还有一个副作用就是会修改计算机的状态,如可变的reductions操作(collect操作)及System.out.println。
JDK对某些副作用处理的很好,特别地,collect方法就是设计针对并行安全有副作用的场景下来执行最常见的stream操作。类似forEach、peek操作可以消除副作用;lambda表达式即使返回void,但如果内部调用了System.out.println,
也会产生副作用。

##### 延迟计算/惰性计算
所有中间操作都是延迟执行的,表达式、方法或算法如果仅仅在需要的时候才计算其值,则就是延迟计算。中间操作是延迟计算因为当终结操作开始时,才会处理流中的数据。延迟处理流的机制让java编译器及运行时
能够优化处理流的方式。比如filter-mapToInt-average流水线操作average可以先从mapToInt操作创建的流中获取几个数,mapToInt则是从filter操作获取元素,average操作则重复着这个流程直至获取所有的数据,然后计算平均值。

##### 干扰
流操作中的lambda表达式不应该有干扰。干扰发生在一个流水线处理流时,流的数据源被修改,如下例:
```
        try {
            List<String> listOfStrings =
                new ArrayList<>(Arrays.asList("one", "two"));

            String concatenatedString = listOfStrings
                .stream()

                // 不要在处理流时,对流的数据源进行修改,会发生干扰,导致报错
                .peek(s -> listOfStrings.add("three"))
                .reduce((a, b) -> a + " " + b)
                .get();

            System.out.println("Concatenated string: " + concatenatedString);
        } catch (Exception e) {
            System.out.println("Exception caught: " + e.toString());
        }
```
##### 有状态的lambda表达式
在流操作中避免使用有状态的lambda表达式当做参数,有状态的lambda表达式是指其结果依赖于在流水线执行过程中可能改变的任何状态,如下例:
```
        List<Integer> serialStorage = new ArrayList<>();
        System.out.println("Serial stream:");
        listOfIntegers
            .stream()
            // 不要这么做,其使用了有状态的lambda表达式
            .map(e -> { serialStorage.add(e); return e; })

            .forEachOrdered(e -> System.out.print(e + " "));
        System.out.println("");

        serialStorage
            .stream()
            .forEachOrdered(e -> System.out.print(e + " "));
        System.out.println("");


        System.out.println("Parallel stream:");
        List<Integer> parallelStorage = Collections.synchronizedList(
            new ArrayList<>());
        listOfIntegers
            .parallelStream()

            // 使用了有状态的lambda表达式
            .map(e -> { parallelStorage.add(e); return e; })

            .forEachOrdered(e -> System.out.print(e + " "));
        System.out.println("");

        parallelStorage
            .stream()
            .forEachOrdered(e -> System.out.print(e + " "));
        System.out.println("");
```

lambda表达式 e -> { parallelStorage.add(e); return e; } 是有状态的lambda表达式,因为每次代码运行其结果都会不同:
```
        Serial stream:
        8 7 6 5 4 3 2 1
        8 7 6 5 4 3 2 1
        Parallel stream:
        8 7 6 5 4 3 2 1
        1 3 6 2 4 5 8 7
```
若想要可预测性及确定性结果,要确保流操作中的lambda表达式参数是无状态的。

注意:ArrayList是非线程安全的,即多个线程不能同时访问指定集合,否则会导致不可预料结果,如下:
```
        Parallel stream:
        8 7 6 5 4 3 2 1
        null 3 5 4 7 8 1 2
```

#### 访问外部范围的局部变量
 和局部和匿名类类似,lambda表达式也能够捕捉变量,也可以访问外部范围的局部变量,但是不会覆盖外部范围的局部变量。

 这是因为lambda表达式从词法上被隔离,不会从超类继承任何变量,也不会引入新的变量作用范围,
 lambda表达式里声明的变量被封闭在lambda表达式里, 如下例:
```
        import java.util.function.Consumer;
        public class LambdaScopeTest {
            public int x = 0;

            class FirstLevel {
                public int x = 1;
                void methodInFirstLevel(int x) {

                    Consumer<Integer> myConsumer = (y) ->
                    {
                        System.out.println("x = " + x); // Statement A
                        System.out.println("y = " + y);
                        System.out.println("this.x = " + this.x);
                        System.out.println("LambdaScopeTest.this.x = " + LambdaScopeTest.this.x);
                    };
                    myConsumer.accept(x);
                }
            }

            public static void main(String... args) {
                LambdaScopeTest st = new LambdaScopeTest();
                LambdaScopeTest.FirstLevel fl = st.new FirstLevel();
                fl.methodInFirstLevel(23);
            }
        }

        x = 23
        y = 23
        this.x = 1
        LambdaScopeTest.this.x = 0
```

假如把lambda表达式声明替换为如下:
```
        Consumer<Integer> myConsumer = (x) -> {
        }
```
则会报x已定义的错,这是因为lambda表达式不会引入新的变量作用范围,因此在lambda表达式内部可以直接访问外部范围的字段、方法、局部变量,比如可以直接访问methodInFirstLevel方法形参x,但是要访问FirstLevel
的字段x,则需要使用this.

#### 目标类型
lambda表达式的类型如何确定呢?是通过目标类型,即如public static void printPersons(List<Person> roster, CheckPerson tester)需要CheckPerson类型,则lambda表达式就是该类型(方法声明需要的参数类型)。

java编译器根据lambda表达式使用的上下文或场景里找到目标类型当做lambda表达式的类型,也就是lambda只能用在java编译器能确定目标类型的场景里:
```
    - 变量声明
    - 赋值
    - return 语句
    - 数组初始化
    - 方法或者构造函数参数
    - lambda表达式体
    - ? : 三元表达式
    - 转换语句
```
对于方法参数,java编译器确定lambda表达式类型通过2个特征:重载解析&类型参数推导

#### 序列化
如同非静态内部类一样,强烈建议禁止lambda表达式的序列化。


#### 方法引用
我们使用lambda表达式来创建匿名方法时,有时lambda表达式啥也没做只是调用了一个已经存在的方法,针对这些情形,通过方法名称来引用已经存在的方法更简洁直观,方法引用可以做到这点。
```
        Arrays.sort(rosterAsArray, Person::compareByAge);
```
##### 方法引用的种类
- 引用静态方法
    ContainingClass::staticMethodName
- 引用非静态实例方法
  containingObject::instanceMethodName
- 引用某个类型的任意对象的实例方法
    ContainingType::methodName,如String::compareToIgnoreCase
- 引用构造方法
    ClassName::new
