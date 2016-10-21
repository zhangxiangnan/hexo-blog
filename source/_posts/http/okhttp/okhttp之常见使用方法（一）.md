layout: title
title: okhttp之常见使用方法（一）
date: 2016-09-25 21:18:48
tags:
- okhttp
- http
- 译
categories: http
---
常见的GET同步、异步调用示例，POST提交字符串、提交流示例。
<!-- more -->

### GET方式同步调用
    package cn.xiangnan.okhttpdemo;

    /**
     * Created by zhangxiangnan on 16/9/25.
     */

    import okhttp3.Headers;
    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;

    import java.io.IOException;

    /**
     * 使用GET方式同步掉调用,输出响应的header及返回内容。
     * string方法适合于<=1M的响应,再大的话最好用stream流方式。
     */
    public class SynchronousGet {
        private final OkHttpClient client = new OkHttpClient();

        public void run() throws Exception {
            Request request = new Request.Builder()
                    .url("http://www.baidu.com")// 可以是需要下载的文件地址
                    .build();
            Response response = null;
            try {
                response = client.newCall(request).execute();
                if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

                Headers responseHeaders = response.headers();
                for (int i = 0; i < responseHeaders.size(); i++) {
                    System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
                }

                System.out.println(response.body().string());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (response != null) {
                    response.close();
                }
            }
        }

        public static void main(String... args) throws Exception {
            new SynchronousGet().run();
        }
    }

### GET方式异步调用
    package cn.xiangnan.okhttpdemo;

    /**
     * Created by zhangxiangnan on 16/9/25.
     */

    import okhttp3.*;

    import java.io.IOException;

    /**
     * 异步GET调用,在一个工作线程中进行文件或者普通页面调用,当响应可读时在另一个线程中执行回调函数;
     * 在响应头部信息准备好后,就执行回调函数;但是读响应体可能仍然阻塞。
     * Okhttp目前不提供对响应体异步分开读取。
     */
    public final class AsynchronousGet {
        private final OkHttpClient client = new OkHttpClient();

        public void run() throws Exception {
            Request request = new Request.Builder()
                    .url("http://www.baidu.com")
                    .build();

            // 异步请求是通过执行入队
            client.newCall(request).enqueue(new Callback() {
                // 失败
                public void onFailure(Call call, IOException e) {
                    e.printStackTrace();
                }

                //正常
                public void onResponse(Call call, Response response) throws IOException {
                    ResponseBody responseBody = null;
                    try {
                        responseBody = response.body();
                        if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

                        Headers responseHeaders = response.headers();
                        for (int i = 0, size = responseHeaders.size(); i < size; i++) {
                            System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
                        }

                        System.out.println(responseBody.string());
                    } catch (Exception e) {

                    } finally {
                        responseBody.close();
                        response.close();
                    }
                }
            });
        }

        public static void main(String... args) throws Exception {
            new AsynchronousGet().run();
        }
    }

### 访问头部信息
通常HTTP头部类似Map<String,String>工作：每一个字段有一个值或更多。但是一些头部允许多个值，如Guava的MultiMap。例如，HTTP响应体提供多个变化的headers是合法和常见的。OKHttp的API尝试优雅的处理2中情形。

当写入请求体时，使用header(name, value)来设置name和value的首次设置，如果已经存在values，values会被移除，新值会被添加。可以使用addHeader(name,value)来追加header，而不用移除已经存在的headers.

当读取响应的头部时，使用header(name)来获取最近出现的命名的value值。通常这也是仅有的一个值。如果值不存在，header(name)返回null。想要读取一个字段的所有值返回一个list，可以使用headers(name).

想要访问所有的头部，使用Headers类，支持按索引访问。


    package cn.xiangnan.okhttpdemo;

    /**
     * Created by zhangxiangnan on 16/9/25.
     */

    import java.io.IOException;

    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;

    /**
     * 访问头部,header(name)访问最近的头部,headers(name)返回多个值
     */
    public final class AccessHeaders {
        private final OkHttpClient client = new OkHttpClient();

        public void run() throws Exception {
            Request request = new Request.Builder()
                    .url("https://api.github.com/repos/square/okhttp/issues")
                    .header("User-Agent", "OkHttp Headers.java")
                    .addHeader("Accept", "application/json; q=0.5")
                    .addHeader("Accept", "application/vnd.github.v3+json")
                    .build();
            Response response = null;
            try {
                response = client.newCall(request).execute();

                if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

                System.out.println("Server: " + response.header("Server"));
                System.out.println("Date: " + response.header("Date"));
                System.out.println("Vary: " + response.headers("Vary"));
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                response.close();
            }

        }

        public static void main(String... args) throws Exception {
            new AccessHeaders().run();
        }
    }

### POST提交String字符串
使用一个HTTP POST来发送一个请求体到一个服务，因为该API的整个请求体全部在内存里，所以最好<=1m。

    package cn.xiangnan.okhttpdemo;

    /**
     * Created by zhangxiangnan on 16/9/25.
     */

    import okhttp3.*;

    import java.io.IOException;

    /**
     * 使用一个HTTP POST来发送一个请求体到一个服务，因为该API的整个请求体全部在内存里，所以最好<=1m。
     */
    public final class PostString {
        public static final MediaType MEDIA_TYPE_MARKDOWN
                = MediaType.parse("text/x-markdown; charset=utf-8");

        private final OkHttpClient client = new OkHttpClient();

        public void run() throws Exception {
            String postBody = ""
                    + "Releases\n"
                    + "--------\n"
                    + "\n"
                    + " * _1.0_ May 6, 2013\n"
                    + " * _1.1_ June 15, 2013\n"
                    + " * _1.2_ August 11, 2013\n";

            Request request = new Request.Builder()
                    .url("https://api.github.com/markdown/raw")
                    .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
                    .build();

            Response response = null;
            try {
                response = client.newCall(request).execute();
                if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
                System.out.println(response.body().string());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                response.body();
            }
        }

        public static void main(String... args) throws Exception {
            new PostString().run();
        }
    }

### POST提交流
    package cn.xiangnan.okhttpdemo;

    /**
     * Created by zhangxiangnan on 16/9/25.
     *  POST方式提交流，请求提写入的同时产生；该例直接将流写入到okio
     * 的buffered sink,可以优先使用OutputStream，并从BufferedSink.outputStream()得到。
     *
     */

    import java.io.IOException;

    import okhttp3.MediaType;
    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.RequestBody;
    import okhttp3.Response;
    import okio.BufferedSink;

    public final class PostStreaming {
        public static final MediaType MEDIA_TYPE_MARKDOWN
                = MediaType.parse("text/x-markdown; charset=utf-8");

        private final OkHttpClient client = new OkHttpClient();

        public void run() throws Exception {
            RequestBody requestBody = new RequestBody() {
                @Override
                public MediaType contentType() {
                    return MEDIA_TYPE_MARKDOWN;
                }

                @Override
                public void writeTo(BufferedSink sink) throws IOException {
                    sink.writeUtf8("Numbers\n");
                    sink.writeUtf8("-------\n");
                    for (int i = 2; i <= 997; i++) {
                        sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
                    }
                }

                private String factor(int n) {
                    for (int i = 2; i < n; i++) {
                        int x = n / i;
                        if (x * i == n) return factor(x) + " × " + i;
                    }
                    return Integer.toString(n);
                }
            };

            Request request = new Request.Builder()
                    .url("https://api.github.com/markdown/raw")
                    .post(requestBody)
                    .build();
            Response response = null;
            try {
                response = client.newCall(request).execute();
                if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
                System.out.println(response.body().string());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                response.close();
            }
        }

        public static void main(String... args) throws Exception {
            new PostStreaming().run();
        }
    }
