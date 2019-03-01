layout: title
title: okhttp简介及使用范例
date: 2016-09-24 16:34:32
tags:
- okhttp
- http
- 译
categories: http
---

### okhttp简介
Http是现代应用程序的通讯方式，我们用来交换数据和媒体。提高http的效率使我们的项目加载更快、节省带宽。  
okhttp客户端对请求响应做了哪些优化呢？以及简单get、post如何实现呢？

<!-- more -->
OkHttp是一个默认效率很高的http客户端：  
  - Http/2支持所有的对同一个host主机的请求共享同一个socket。
  - 连接池减少链接数目（如果HTTP/2不可用）
  - 默认GZIP压缩减少传输数据大小
  - 缓存响应减少网络请求对于重复的请求。

OkHttp当网络出问题时做如下尝试：
  它默默地恢复对于常见的连接问题。如果你的服务有多个ip地址，，OkHttp将选择一个其他可用的地址如果第一次连接请求失败的话。这对IPV6 + IPV4以及多数据中心的服务是很有必要的。OkHttp使用现代的TLS特征（SNI、ALPN）发起新连接，如果握手失败则使用TLS1.0进行兜底。

使用OkHttp很简单，它的请求响应API使用流式创建器及不变性。它同时支持同步调用及带有回调机制的异步调用。

OKHttp支持android>=2.3，java版本最低1.7

### Get请求示例
```   
    package cn.xiangnan.okhttpdemo;

    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;

    import java.io.IOException;

    /**get请求数据
    * Created by zhangxiangnan on 16/9/24.
    */
    public class GetDemo {
    OkHttpClient client = new OkHttpClient();

    String doGet(String url) throws IOException {
        Request request = new Request.Builder()
                .url(url)
                .build();

        Response response = client.newCall(request).execute();
        return response.body().string();
    }

    public static void main(String[] args) throws IOException {
        GetDemo demo = new GetDemo();
        String response = demo.doGet("https://raw.github.com/square/okhttp/master/README.md");
        System.out.println(response);
    }
    }
```
### POST请求示例（提交json数据）
```    
    package cn.xiangnan.okhttpdemo;

    import okhttp3.*;

    import java.io.IOException;

    /**post请求发送json数据
     * Created by zhangxiangnan on 16/9/24.
     */
    public class PostDemo {
        public static final MediaType JSON
                = MediaType.parse("application/json; charset=utf-8");

        OkHttpClient client = new OkHttpClient();

        String post(String url, String json) throws IOException {
            RequestBody body = RequestBody.create(JSON, json);
            Request request = new Request.Builder()
                    .url(url)
                    .post(body)
                    .build();
                Response response = client.newCall(request).execute();
            return response.body().string();
        }

        String bowlingJson(String player1, String player2) {
            return "{'winCondition':'HIGH_SCORE',"
                    + "'name':'Bowling',"
                    + "'round':4,"
                    + "'lastSaved':1367702411696,"
                    + "'dateStarted':1367702378785,"
                    + "'players':["
                    + "{'name':'" + player1 + "','history':[10,8,6,7,8],'color':-13388315,'total':39},"
                    + "{'name':'" + player2 + "','history':[6,10,5,10,10],'color':-48060,'total':41}"
                    + "]}";
        }

        public static void main(String[] args) throws IOException {
            PostDemo example = new PostDemo();
            String json = example.bowlingJson("Jesse", "Jake");
            String response = example.post("http://www.roundsapp.com/post", json);
            System.out.println(response);
        }
    }
```