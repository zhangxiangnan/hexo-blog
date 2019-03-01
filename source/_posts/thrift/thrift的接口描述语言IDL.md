layout: title
title: thrift的接口描述语言IDL
date: 2016-08-01 10:29:05
tags:
- thrift
- thrift type

categories: thrift
---
本文介绍thrift的接口描述语言IDL。
<!-- more -->

#### 简介
thrfit的接口描述语言IDL（interface description language）允许使用thrfit类型。thrift编译器处理thrift的IDL文件来根据不同的目标语言，生成对应的代码，以便支持IDL文件中定义的结构体和服务。

#### IDL结构描述
##### Document
每个thrift的document包含0个或者多个headers，后面跟着0个或多个其他定义：    
```
    [1]  Document        ::=  Header* Definition*
```
##### Header
一个header是一个thrift的include、一个c++的include、或者一个namespace的定义：   
```
    [2]  Header          ::=  Include | CppInclude | Namespace
```
###### Thrift Include
该种include使来自另一个文件的所有符号标记对该文件可见（使用前缀），并且添加对应的include语句到该thrift文档所生成的代码中。   
```   
    [3]  Include         ::=  'include' Literal
```
###### C++ Include
该include添加一个自定义的c++ include到thrift文档对应c++代码生成器生成的代码里。   
```   
    [4]  CppInclude      ::=  'cpp_include' Literal
```
###### Namespace
一个namespace定义了哪一个namespace、包、模块等。thrift文件定义的类型将在目标语言中声明定义。namespacescope定义了该namespace所适用于哪种语言。范围“ \*”表示该namespace适用于所有语言。   
```
[5]  Namespace       ::=  ( 'namespace' ( NamespaceScope Identifier ) |
                                    ( 'smalltalk.category' STIdentifier ) |
                                    ( 'smalltalk.prefix' Identifier ) ) |
                      ( 'php_namespace' Literal ) |
                      ( 'xsd_namespace' Literal )

[6]  NamespaceScope  ::=  '\*' | 'cpp' | 'java' | 'py' | 'perl' | 'rb' | 'cocoa' | 'csharp'
```
N.B.:smalltalk有2种不同的命名指令；php_namespace与xsd_namespace不建议用。

##### 定义Definition   
```
[7]  Definition      ::=  Const | Typedef | Enum | Senum | Struct | Union | Exception | Service
```
##### Const
```
[8]  Const           ::=  'const' FieldType Identifier '=' ConstValue ListSeparator?
```
##### Typedef
类型定义为一个类型创建别名：    
```
[9]  Typedef         ::=  'typedef' DefinitionType Identifier
```
##### Enum
一个枚举创建一个枚举类型，带有命名的值。如果没提供常量值，则第一个元素值要么是0，或者比先前的子序列的数字都大。任何常量值必须非负。
