layout: title
title: java注解详解
date: 2018-01-05 19:16:57
tags:
- 注解
categories:
- 注解
---

java注解学习

<!-- more -->

注解是元数据的一种形式,提供程序相关的数据,但不是程序本身的一部分,注解不会直接操作他们所标注的代码。

注解有很多用途:

    - 编译器所需的信息,如编译器可以用注解来检测错误或禁止警告
    - 编译时或发布时处理,如软件工具可以处理注解信息来生成代码、xml文件等
    - 运行时处理,某些注解在运行时也可以检测到

#### 注解基本点
##### 注解的形式
最简单的:

        @Entity
@符号后的信息对编译器表明这是一个注解。

以下注解的名称是Override:

        @Override
        void mySuperMethod() {  }

注解也可以包含元素,元素可以有名称,也可以没有名称,他们也有对应的值:

        @Author(
           name = "Benjamin Franklin",
           date = "3/27/2003"
        )
        class MyClass() { }

        @SuppressWarnings(value = "unchecked")
        void myMethod() { }

如果只有一个命为value的元素,那么名称可以省略,如:

        @SuppressWarnings("unchecked")
        void myMethod() {}

如果注解没有元素,那么圆括号可以省略,如@Override注解的例子。

在相同声明的位置可以使用多个注解:

        @Author(name = "Jane Doe")
        @EBook
        class MyClass { }

如果注解有相同的类型,称为可重复注解(>=Jdk8):

        @Author(name = "Jane Doe")
        @Author(name = "John Smith")
        class MyClass { }

注解可以使用java.lang或java.lang.annotation包下定义的注解,也可以使用自定义注解。

##### 注解可以用在哪儿

注解可以用在声明的地方:

    - 类/接口(含注解类型)/枚举声明
    - 方法声明
    - 字段(含枚举常量)声明
    - 形参声明
    - 构造函数声明
    - 本地变量声明
    - 注解类型声明
    - 包声明
    - 类型参数声明
    - 类型的使用

使用时注解约定占一行。

##### 类型注解
Java8中支持类型使用的注解,称为类型注解,如:
实例化对象的表达式:

        new @Interned MyObject();

类型强转:

        myString = (@NonNull String) str;
implements子句:

        class UnmodifiableList<T> implements
            @Readonly List<@Readonly T> {  }

异常声明:

        void monitorTemperature() throws
            @Critical TemperatureException {  }

#### 定义注解类型
注解可以替换注释为代码,设想:

        public class Generation3List extends Generation2List {
           // Author: John Doe
           // Date: 3/17/2002
           // Current revision: 6
           // Last modified: 4/12/2004
           // By: Jane Doe
           // Reviewers: Alice, Bill, Cindy
           // class code goes here
        }

将这些注释信息如何转化为一个注解?首先必须定一个注解类型,如下:

        @interface ClassPreamble {
           String author();
           String date();
           int currentRevision() default 1;
           String lastModified() default "N/A";
           String lastModifiedBy() default "N/A";
           // Note use of array
           String[] reviewers();
        }

注解类型的定义有点像接口的声明,但是在interface关键字前加@符号。注解是接口的一种形式,其元素的声明像方法,可以指定默认值。


        @ClassPreamble (
           author = "John Doe",
           date = "3/17/2002",
           currentRevision = 6,
           lastModified = "4/12/2004",
           lastModifiedBy = "Jane Doe",
           // Note array notation
           reviewers = {"Alice", "Bob", "Cindy"}
        )
        public class Generation3List extends Generation2List {}

注意:如果想在Javadoc生成的文档中包含@ClassPreamble中描述的信息,需要在其注解的定义上加@Documented注解。

        @Documented
        @interface ClassPreamble {
        }

#### 已有注解类型
Java预先定义了一些注解,有些被java编译器使用,有些应用到其他注解中。
##### java语言使用的注解
java.lang定义的注解为:@Deprecated, @Override, and @SuppressWarnings.

###### @Deprecated
该注解表明被标注的元素已过时,不该再被使用。使用有过时注解标注的方法、类、字段,编译器都会警告。如果元素过时,也应该使用Javadc的过时标记@deprecated来标注,两个注解一个大写,一个小写。

       // Javadoc comment follows
        /**
         * @deprecated
         * explanation of why it was deprecated
         */
        @Deprecated
        static void deprecatedMethod() { }

###### @Override
@Override注解通知编译器被标注元素覆盖了父类的元素:

       // 重写方法约定加覆盖注解
       @Override
       int overriddenMethod() { }

虽然不强制使用覆盖标记,但是使用了有助于发现错误。

###### @SuppressWarnings
@SuppressWarnings注解告诉编译器忽略可能产生的某种特定的警告,如下忽略了过时方法调用警告:

       @SuppressWarnings("deprecation")
        void useDeprecatedMethod() {
            // deprecation warning
            // - suppressed
            objectOne.deprecatedMethod();
        }

每一种编译器警告都属于一个分类,java语言规范定义了2中分类:deprecation and unchecked. 想要忽略多种分类的警告,如下:

        @SuppressWarnings({"unchecked", "deprecation"})

###### @SafeVarargs
@SafeVarargs注解当使用在方法或构造函数上时,断言了代码不可能对可变参数执行潜在的不安全操作。当使用该注解后,跟可变参数使用相关的unchecked警告会被忽略。

###### @FunctionalInterface
@FunctionalInterface在java8中引入,表明当前类型声明是一个函数接口。

##### 应用到其他注解的注解
这类注解成为元注解, java.lang.annnotation中定义的几个元注解如下:

###### @Retention
@Retention注解指明了被标记注解如何存储:

    - RetentionPolicy.SOURCE, 被标记注解仅仅在源文件级别保留,编译时被忽略
    - RetentionPolicy.CLASS ,被标记注解在编译时被编译器保留,但是在JVM运行时被忽略
    - RetentionPolicy.RUNTIME ,被标记注解被jvm保留,在运行时环境中可以使用

###### @Documented
@Documented注解表示被标注的注解在使用Javadoc工具生成文档时会被记录到文档中,默认注解是不被记录的。

###### @Target
@Target注解标记其他注解来限制注解可以应用的java元素,如:

    - ElementType.CONSTRUCTOR 可应用到构造函数上
    - ElementType.FIELD 可应用到字段或属性上
    - ElementType.LOCAL_VARIABLE 可应用到本地变量上
    - ElementType.METHOD 可应用到方法级别的注解上
    - ElementType.PACKAGE 可应用到包声明上
    - ElementType.PARAMETER 可应用到方法的参数上
    - ElementType.TYPE 可应用到类的任何元素上
###### @Inherited
@Inherited表示注解可以从父类继承,默认不可以。 当用户查询注解类型时,若当前类没有该类型的注解,会接着查询父类的该注解。该注解只可用于类声明上。

###### @Repeatable
@Repeatable在java8引入,表示被标记注解可以多次应用到相同声明或类型使用


#### 类型注解&可插拔类型系统
Jdk8以前,注解只能应用在声明上,jdk8里注解可以应用到任何类型使用上,这意味着可以在你使用一个类型的任何地方使用。
类型注解引入是为了提升java程序的分析来提供更强的类型检查,java8不提供类型检查框架,但是它允许我们来使用多个可插拔类型检查框架来配合java编译器进行更强大的类型检查。

如,想要确保某一个特定的变量不能被赋值为null,来避免NPE异常的话,可以编写一个自定义插件来检查这一点,类似如下:

        @NonNull String str;
可以加载多种插件来加强类型检查,减少错误。


#### 可重复注解
有些情形可能想要使用相同的注解多次,java8引入类型注解。
如想要定义多个时间点执行方法,如下:

        @Schedule(dayOfMonth="last")
        @Schedule(dayOfWeek="Fri", hour="23")
        public void doPeriodicCleanup() { }

可重复注解可以在任何使用标准注解的地方使用,如也可在类声明上使用,如:

        @Alert(role="Manager")
        @Alert(role="Administrator")
        public class UnauthorizedAccessException extends SecurityException { }

为了兼容,可重复注解保存在java编译器自动生成的注解容器里,需要以下两步声明:

- 一是定义可重复注解:

        import java.lang.annotation.Repeatable;
        @Repeatable(Schedules.class)
        public @interface Schedule {
          String dayOfMonth() default "first";
          String dayOfWeek() default "Mon";
          int hour() default 12;
        }

    @Repeatable元注解圆括号里定义的是注解容器的类型,java编译器用来存储可重复注解。

    这个例子里,注解容器是Scheudles,所以可重复注解@Schedule存储在@Schedules注解里。

- 二是声明容器枚举类型

    容器枚举类型必须有一个数组类型的元素,数组的数据类型必须是可重复注解的类型,如:

        public @interface Schedules {
            Schedule[] value();
        }

##### 重新获取注解
反射API中有几个方法可用来获取注解信息,返回一个单独注解的方法,如AnnotatedElement.getAnnotation(Class<T>)的行为不变,因为仅仅返回请求类型的一个注解(如果存在),如果请求类型存在多个注解,可以先得到其容器注解。

java8也引入其他方法来遍历容器注解来一次获取多个注解,如AnnotatedElement.getAnnotationsByType(Class<T>)。

##### 设计考虑
当设计注解类型时,必须考虑注解类型的使用次数,因为有了可重复注解,可使用多次;也可以通过@Target元注解来限制注解的使用元素范围。

总之,要让使用者使用起来尽可能灵活、强大。
