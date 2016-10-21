layout: title
title: okhttp之连接Connections
date: 2016-09-25 12:51:16
tags:
- okhttp
- http
- 译
- Connections
categories: http
---
### 连接
在使用Okhttp发送请求时，尽管你仅仅提供URL，OkHttp使用三种方式来设计连接：URL、Address、和路有route。
<!-- more -->

#### URLs链接

URLs（如 https://github.com/square/okhttp ）是HTTP和Internet互联网的基石。为了成为一个为web上任何东西的通用的、去中心化分散的的命名方案，他们也指出了如何访问web资源。

URLs是抽象的：
  - URLS指出调用可能是明文的（http）或者加密的（https），但没有指出哪一种加密算法应该使用，也未指定该如何验证访问者的身份或者何种的凭证应该被信任，
  - 也未指出是否应该使用一个特定的代理服务器或者如何和代理服务器认证。

URLS也是具体的：每一个URL标示一个特定的路径（如/square/okhttp)和查询(如?=q=xx&a=xx)。每一个web服务器管理许多URLS。

#### Addresses地址
地址指定一个web服务器（如github.com），以及连接到服务器所需的静态配置：端口、Https设置、优先的网络协议（如HTTP/2或者SPDY）。

共享相同address地址的URLs可能共享相同的底层的TCP socket连接。共享一个连接对性能有重大提升：低延迟、高吞吐（因为TCP启动较慢，三次握手）以及节省用电.Okhttp使用一个自动重用HTTP/1.x连接的连接池、支持多路复用技术的HTTP/2、SPDY连接。
在Okhttp中，address地址的一些字段来自URL（协议，主机，端口），剩余来自OkHttpClient。

#### 路由Routes
路由提供动态必需的动态信息来实际连接到一个web服务器。通过特定的ip地址尝试（通过DNS查询发现），使用准确的代理服务器（如果使用了一个ProxySelector），以及TLS协商的版本（针对HTTPS连接）。
一个地址可能有许多路由，例如，在多数据中心的一个web服务器可能在DNS应答中返回多个ip地址。

#### 连接
当你使用Okhttp请求一个URL，如下是Okhttp请求的内部过程：   

  1、它使用URL和配置好的OkHttpClient来创建一个address地址，这个address地址指出了我们如何连接到web服务器。  
  2、它尝试在连接池中检索来获取那个地址的连接  
  3、若在连接池中没有找到对应连接，它选择一个路由来尝试。这通常意味着发出一个DNS请求来得到服务器的IP地址。如果必须的话，它还会选择一个TLS版本和代理服务器  
  4、如果是一个新路由，它通过构建一个直接socket连接、一个TLS隧道（针对一个HTTP代理上的HTTPS）、或者一个直接的TLS连接来连接。必要时它会进行TLS握手。
  5、最后发送HTTP请求然后读取响应。

如果连接异常，Okhttp将再选择一个路有，再次重试。这允许Okhttp在当一个服务端的多个地址有一部分不可达时，进行恢复。这当一个连接池中的连接过时失效时，或者已经尝试的TLS版本不支持的话，也是有用处的。

一旦响应被接受，连接就会被返回到连接池，以便新的请求可以重用该连接。在一个连接不活跃一段时间后，连接将被从连接池回收。
