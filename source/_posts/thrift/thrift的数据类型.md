layout: title
title: thrift的数据类型
date: 2016-07-31 18:01:47
tags:
- thrift
- thrift type

categories: thrift
---
本文介绍thrift的数据类型。
<!-- more -->

#### 简介
apache的thrift框架，facebook开发，用于可扩展的跨语言的服务开发，把软件栈和代码生成引擎结合，来构建高效的、在C++、JAVA、Python、PHP、Ruby、Erlang、Perl、Haskell、C#、Cocoa、JavaScript、Node.js、SmallTalk、Ocami、Dephi及他语言之间无缝的服务。

#### thrfit的数据类型
thrift的类型系统目的在于允许开发人员尽可能使用本地类型，不用管他们使用的是何种编程语言。thrift的IDL提供类型的描述，用来生成每一个目标语言对应的代码。

##### 基本类型
简单类型以简单明了作为目标，而不是提供大量简单类型，集中在所有语言共有的关键类型上。
- bool：一个布尔值，true或者false，对应java的boolean
- byte：一个8字节的有符号整数，对应java的byte
- i16：16位的有符号整数，对应java的short
- i32：32位有符号整数，对应java的int
- i64: 一个64位的有符号整数，对应java的long
- double：一个64位浮点数，对应java的double
- string：一个使用utf8编码的字符串，对应java的string   

注意：没有无符号整数，因为大多数语言没有这种类型。

##### 特殊类型
二进制：未编码的字节序列
注意：这个类型目前仅仅是上面string类型的特殊形式，为了和java更好的交互性，某些点上考虑也可认为是基本类型。

##### 结构体structs
结构体定义了公共对象，本质上等同于OOP语言中的类，但是没有继承的概念。结构体有一系列强类型的字段，每一个都有唯一的命名标识符。字段可能有不同的注解（数字的字段IDs，可选的默认值等。），这些在IDL中描述。

##### 容器类型Containers
thrift的容器类型是强类型容器，对应大部分语言中常用和公共的容器类型。
有3种容器类型：
- list：有序的元素列表，被转换为STL vector，java arraylist，脚本语言中的数组等。
- set：唯一元素的无序集合，被转换为STL set，java hashset，Python set等，php无sets，转成类似于list对待。
- map：严格的唯一键到值得映射，转换为STL map，java hashmap，PHP associative array，Python/ruby的dictionary等。默认时，类型映射未明确绑定，自定义代码生成器可以替换为目标语言的自定义类型（不懂。）

容器类型支持任何有效的thrift type。
N.B.: 为了最大的兼容通用性，map的key只能使用基本类型，不能使用容器或者结构体类型，因为有些语言的原始map类型不支持key使用复杂类型，如json协议的key只支持使用基本类型。

##### 异常Exceptions
异常功能等同于结构体，除了异常继承自每种语言的原始异常的基类，以便和每个语言的原始本地异常类无缝衔接。

##### 服务Services
服务使用thrift的类型定义，一个service的定义语义等同于定义一个接口（或者纯粹的抽象类）在面向对象的编程中。thrift编译器根据服务可生成功能齐全的客户端及实现了服务接口的server存根。

一个service包含一系列命名函数，每一个包含一系列参数和一个返回类型。

注意： void是一个有效的方法返回类型，除了其他定义的thrift类型。
此外，一个单向的修饰符关键字可以修饰一个void函数，生成的代码不用等server端返回响应。一个纯粹的void函数是会返回一个响应到客户端的，来保证这个操作已在服务端处理完成。单向方法调用仅仅保证强求在传输层调用成功。同一客户端的单向方法多次调用可能在服务端是并行或者无序执行。
