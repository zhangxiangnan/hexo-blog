layout: title
title: spring如何保证线程安全的
date: 2016-06-14 13:41:03
tags:
- spring
- framework
- threadsafe
- concurrent
categories: spring
---

### spring的常见业务组件采取单例如何保证线程安全？    
　　spring作为ioc框架，一般spring项目管理的bean如controller、api、service、dao大多数是单例（默认单例，配置多例模式使用scope=prototype），既然是单例，那么如何控制单例被多个线程同时访问线程安全呢？    
  <!-- more -->
  　　首先要理解每个http请求到后台都是一个单独的线程，线程之间共享同一个进程的内存、io、cpu等资源，但线程栈是线程独有，线程之间不共享栈资源。    
　　其次，bean分为有状态bean和无状态bean，有状态bean即类定义了成员变量，可能被多个线程同时访问，则会出现线程安全问题；无状态bean每个线程访问不会产生线程安全问题，因为各个线程栈及方法栈资源都是独立的，不共享。即是，无状态bean可以在多线程环境下共享，有状态bean不能。    
　　所以一般来说，spring管理的api、service、dao都是单例存在，节省内存和cpu、提高单机资源利用率。

### spring的dao、service层使用的数据库connection连接等一些有状态bean如何保证线程安全？
　　spring应用中dao、service一般以单例形式存在，dao、service中使用的数据库connection以及RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等都是有状态bean，而dao、service又是单例，如何保证线程安全呢？    
　　答案是使用threadLocal进行处理，ThreadLocal是线程本地变量，每个线程拥有变量的一个独立副本，所以各个线程之间互不影响，保证了线程安全。

### springMVC的controller并发访问呢？    
　　springMVC中的controller默认是单例的，所以如上所述，属性变量会到值线程安全问题，解决方法包括使用threadLocal或、不使用属性变量、配置为多例均可（加锁控制效率不行）。

### spring的几个概念
singleton：单例、缺省,就是指一个spring的容器中，只有一个实例存在，和单例模式不同。
prototype：原型，多例，指一个spring容器中，每个请求创建一个单独的实例存在。   
scope属性的其他值：request：请求范围内，session：session范围内。

### 如何配置单例与多例
spring的注解或者xml配置属性scope=singleton表示单例，scope=prototype表示多例。
