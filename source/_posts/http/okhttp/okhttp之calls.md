layout: title
title: okhttp之calls
date: 2016-09-24 20:08:12
tags:
- okhttp
- http
- 译
categories: http
---

### okhttp的Calls介绍
Http客户端的工作是接受请求，并返回请求的响应，理论简单，实践复杂。Okhttp如何处理的呢？
<!-- more -->

### 请求
每一个http请求包含一个url，一个方法（get或者post），以及一系列headers。请求也可能包含body请求体：一种指定内容类型的数据流。

### 响应
响应应答请求，返回一个code（如200表示成功，404表示找不到），headers以及可选择的body响应体。

### 重写请求
当你提供http请求给Okhttp时，你以很抽象地描述这个请求：“帮我使用这些headers来抓取这个url的内容”。为了准确性和效率考虑，Okhttp重写请求后才发送该请求。

Okhttp可能增加原始请求中却少的headers，包含内容长度Content-Length,Transfer-Encoding传输编码，User-Agent用户代理，Host主机，连接Connection，内容类型Content-Type。它会添加支持编码Accept-Encoding头为了透明的响应压缩，除非该header已经存在。如果你有cookies，Okhttp会使用你的cookies添加到cookie头。

一些请求会有一个被缓存的响应信息。当这个被缓存的响应不是最新的，Okhttp可以做一个有条件的Get来下载一个更新的响应，如果比缓存的更 新。这需要诸如If-Modified-Since及If-None-Match头部添加到请求中。

### 重写响应
如果使用透明的压缩，Okhttp将会放弃对应的响应头部Content-Encoding和Content-Length，因为这2个头部不适合于解压缩的响应体中。

如果一个有条件的GET成功，来自网络服务端的响应和缓存中的内容将按照规范来合并。

### 跟踪后续请求
当你请求的url被移动，web服务器将返回如302响应码来表明文档的新url地址，Okhttp将调用重定向的地址来获取最终响应。

如果响应应答说需要认证，OkHttp会询问Authenticator（如果配置了一个）来满足认证。如果authenticator认证器提供一个凭证，Okhttp将拿着凭证再一次请求。

Retrying Requests
### 重试请求
有时连接失败：或者是一个被池化的连接过时和断开连接，或者是web服务端自己不可达。OkHttp将使用一个不同的路有（如果有一个可用路有）来重试请求。

### Calls
有了重写、重定向、跟踪后续请求、重试请求，你的简单的请求可能为更多请求和响应让路。Okhttp使用Call来模型化请求任务，然而许多中间的请求和响应是必须的。通常这种情况不是很多，但知道你的代码会在遇到URL重定向是自动继续跟踪请求或者兜底来请求一个备用ip地址还是很好的。

Calls以以下2种方式的一种来执行:  
  - 同步：你的请求线程阻塞直到响应可读。
  -  异步：你在任何线程上将请求加入到队列中，当响应可读时，在另一个线程中获取回调。Calls可以在任何线程中被取消。这将将还未完成的call执行失败，那些正在写请求体或者读响应体的代码将触发IOException.

### 转发分派
  对于同步调用calls，你使用自己创建的线程，并且有责任管理你发出的并发请求，太多并发请求连接浪费资源，对响应时间几乎没有影响。
对于异步调用calls，调度器实现控制最大并发请求的策略，你可以设置每台web服务器的最大并发数（默认5），以及全局并发数（默认64）。
