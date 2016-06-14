layout: title
title: spring的7种事务传播行为（传播级别、传播行为）
date: 2015-06-01 21:23:53
tags:
- spring
- transaction
- propagation
categories: spring
---
spring管理的事务中，事务是否允许嵌套？如何配置无事务？如何配置必有事务？如何配置事务存在则报错？嵌套事务任一方回滚会发生什么？ ……

### 什么是事务传播行为？
　　事务保证原子性操作的手段，要么全部操作都提交，要么都回滚，保证数据一致性，避免产生脏数据。   
　　当我们调用spring管理的service方法A时，A运行在spring管理的事务，A可能调用service的B方法，这就是服务接口方法嵌套调用，spring通过事务传播行为控制当前事务如何传播到被嵌套调用的目标服务接口方法中。
　　就是说，事务传播属性（传播行为、传播级别）定义的是事务的控制范围（影响作用范围），举例简单说就是A方法的事务会传递到B方法吗，B方法用不用新建事务？还是用A的？还是嵌套事务？还是报错？还是将A方法的事务挂起？。

### 事务传播行为种类
spring的TransactionDefinition接口定义了7中类型的事务传播行为，他们规定了事务方法与事务方法在发生嵌套调用时如何传播事务。

    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;

传播行为     | 说明
-------- | ---
PROPAGATION_REQUIRED | 只支持单个事务，有事务就使用这个事务，无则创建一个新事务，事务定义的默认配置，事务同步范围的常用配置。
PROPAGATION_SUPPORTS    | 有事务则使用当前事务，无事务则以无事务方式执行。
PROPAGATION_MANDATORY     | 有事务则使用当前事务，当前无事务则抛出异常。
PROPAGATION_REQUIRES_NEW | 创建新事务，当前有事务则挂起当前事务。
PROPAGATION_NOT_SUPPORTED |不支持事务，如果存在事务，则将当前事务挂起。
PROPAGATION_NEVER | 只能以非事务运行，有事务就报错。
 PROPAGATION_NESTED | 如果当前环境不存在事务则与PROPAGATION_REQUIRED一样，否则若有事务，则新建子事务，子事务与父事务形成嵌套事务，事务有联系，不是独立的。该行为需基于jdbc3.0，且持久化框架需实现了保存点事务机制。

 ### PROPAGATION_REQUIRES_NEW（新事务）与PROPAGATION_NESTED（嵌套事务）的区别？
- PROPAGATION_REQUIRES_NEW(新事务)当前没有事务则创建新事务，当前有事务会将外层事务挂起，新事务执行完毕外层事务继续执行；PROPAGATION_NESTED（嵌套事务）不会挂起外层事务。
- PROPAGATION_REQUIRES_NEW创建的新事务和外层事务无关，相互独立地提交和回滚，拥有自己的数据库隔离级别和锁；PROPAGATION_NESTED的嵌套事务和外层事务有关联，外层事务提交或回滚，子事务也会提交回滚。

### 嵌套事务的提交回滚说明
　　外层事务相当于父事务，内层事务相当于子事务。    
　　嵌套事务是子事务套在父事务中执行，进入子事务之前，父事务建立一个回滚点或者叫保存点savepoint，然后执行子事务，子事务执行完毕，父事务继续执行。
- 子事务回滚，父事务发生什么？　父事务会滚到savepoint，继续执行其他事务与业务逻辑，父事务之前操作不影响，不会自动回滚。
- 父事务回滚，子事务呢？  子事务回滚，子事务是父事务的一部分，随着父事务提交而提交。
- 子事务、父事务提交事务的顺序？外层事务提交时，子事务一并提交，子事务不能先提交。
- 子事务异常回滚，并抛异常，父事务若捕获异常则可能提交，否则回滚。

### PROPAGATION_MANDATORY设置的方法能被其他非事务方法调用吗？
不能，会报错，比如controller直接调该方法，则报异常，必须由其他含有事务的业务方法调用，设置该行为的方法一般间接被调用。

### PROPAGATION_NEVER被其他有事务的方法调用的话？
报错，设置该行为的方法一般是直接调用。
### 注意点
- 事务传播行为是设置在某个具体方法上的
