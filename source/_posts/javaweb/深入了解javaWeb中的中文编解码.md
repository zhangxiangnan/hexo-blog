layout: title
title: 深入了解javaWeb中的中文编解码
date: 2015-05-12 09:40:52
tags:
- encode
- decode
- javaWeb
categories:
- javaWeb
- encode decode
---

为什么需要编解码？有几种编解码？javaweb哪些场景需要编解码？常见乱码问题如何分析解决？

<!-- more -->

### 为啥需要编解码？
- 计算机中存储信息的最小单位是字节，一个字节8位，能表示范围是0~255，假如说0表示汉字“我”，1表示。。，这样最多只能表示256个汉字，而汉字有很多。
- 其他语言也太多，不可能255个字符只用来表示汉字。

那就需要一个新的若干个byte组合的数据结构来表示众多的语言文字-char。    
从char到byte需要编码，从byte到char需要解码。

### 常见编解码格式
编解码格式定义了从字节到字符，从字符到字节的转化的规则，通过这个规则可以把汉字转成字节传输，可以从字节转成汉字。    
常见的编解码格式如下表：


编码格式   | 表示个数 | 所需字节数 |  说明
--------  | ---
ASCII   | 128 | 单字节的低七位表示 | 0~31为控制字符如回车换行等；32~126为打印字符，可键盘输入能够显示出来
ISO-8859-1 | 表示256个字符 |  单字节 | 扩展ASCII码，ISO8859-1到ISO8859-15，ISO8859-涵盖大多数西欧语言字符，应用最广泛。
GB2312 | 7445  | 双字节 | A1~A9是符号区，682个；B0~F7是汉字区，共6763个汉字。
GBK | 23940 | 双字节 | 扩展自GB2312，支持更多汉字，范围从8140~FEFE（去掉XX7F），能表示21003汉字，兼容GB2312。
GBK18030 | 兼容GB2312 | 应用不广泛 | 应用不广泛
UTF-16 | 处理Unicode编码 | 双字节 | 用2字节表示Unicode的转化格式，任何字符都通过2个字节表示，定长表示，效率快，java以UTF-16内存的字符存储格式。适合在本地磁盘和内存之间使用，可以达到字符和字节快速切换。
UTF-8 |处理unicode编码 | 变长 | 每个编码区域不同字码长度，不同类型字符可以由1~6个字节组成，节省空间，效率不如utf-16，介于gbk和uft-16之间，适合网络传输，对ASCII码单字节存储，单字符损坏不影响后面字符。其通过首字节的前2位确定需要几个字节表示。

说明：unicode是统一码，ISO创建的全新的超语言字典，所有语言都可以通过这个字典相互翻译。

### java中哪些场景要编码
#### IO编解码
分磁盘IO与网络IO，网络IO见javaWeb中编解码。    
读文件（字节到字符）：java使用InputStreamReader关联字节到字符的桥梁，进行字节解码，解码由StreamDecoder使用指定charset，无则默认z中文使用GBK。

写文件（字符到字节）：java使用OutputStreamWriter做字符到字节的转换桥梁，进行字符的编码，编码由StringEncoder处理，使用指定charset，无默认GBK。

#### 内存中编解码
##### String提供的方法
如下：

    String s = "hah哈你好";
    byte[] b = s.getBytes("UTF-8");
    String n =new String(b, "UTF-8");

##### Charset的encode和decode方法
 其提供char[]到byte[]编码及byte[]到char[]解码，如下：   


    Charset charset = Charset.forName("UTF-8");
    ByteBuffer byteBuffer = charset.encode(string);
    CharBuffer charBuffer = charset.decode(byteBuffer);

##### ByteBuffer类的软转换
ByteBuffer提供char与byte的软转换，转换不需要编解码，只是把16bit的char拆分为2个8bit的byte表示，实际值没有修改，仅仅数据类型做转换：   

      ByteBuffer bytebuffer2 = ByteBuffer.allocate(1024);   
      ByteBuffer byteBuffer = bytebuffer2.putChar(c);

### javaWeb中涉及的编解码
网络传输都是以字节为单位，java对象要经过网络传输必须经过序列化，实现Serializable序列化接口。

用户发起http请求，需要编码的地方有URL、Cookie、Parameter服务端收到htt请求，对URI、Cookie、POST表单参数解码，服务端还可能从数据库、本地或网络其他地方的文本文件读取数据，都可能存在解码问题。servlet处理请求，将响应数据编码通过socket返回到浏览器，浏览器解码为文本信息。

#### URL的编解码
URL中可能有中文，需要编码，根据URL的编码规范RFC3986，浏览器将非ASCII字符按照某编码格式编码然后前面加上%，可以设置编码，不同浏览器不一样。

tomcat对URL的URL部门进行解码的字符集在<Connector URIEncoding='UTF-8'/>设置，默认ISO-8859-1，一般URL有中文该项需设置。

URL的QueryString（get请求的url后的请求参数）是通过HTTP的Header的content-type定义传到服务器，要么默认ISO-8859-1，使用自定义的ContentTYpe，需要设置 <Connector URIEncoding="UTF-8" useBodyEncodingFOrURI="true"/>，该配置不是针对全部URI采用BodyEncoding编码，而是只针对QueryString部分。

总结：尽量避免在URL中使用非ASCII字符，服务器最好也设置Connector的2个属性。

#### HTTP Header的编解码
Header中的Cookie、redirectPath也可能存在编码问题。
tomcat对Header中项解码在调用request.getHeader()时进行。如果请求的Header项没有解码则调用MessageBytes的toString方法，默认使用ISO-8859-1，也不能设置其他格式，如果Header中有非ASCII字符，解码会乱码。

#### POST表单的编解码
POST表单提交的参数的解码在第一次调用request.getParameter时发生，POST表单参数通过http的body传递到服务端。
整个流程是点提交时，浏览器根据contenttype的charset对表单参数编码，提交到服务端，服务端同样用contenttype中的字符集进行解码，所以post表单的参数一般不会乱码。
通过request.setCharacterEncoding(charset)可以设置。

注意：要在第一次调用request.getParameter方法之前设置request.setCharacterEncoding(charset)，否则POST表单提交的数据可能出现乱码。

####  HTTP BODY的编解码
服务端返回的结果经过编码返回浏览器，浏览器解码，然后显示。
编码可以通过response.setCharacterEncoding设置，会覆盖request.setCharacterEncoding的值，通过Header的content-type返回给客户端。
浏览器首先根据Content-type解码，无则根据HTML的<meta HTTP-equiv="content-type" content="text/html;charset=gbk"/>来解码，无则使用浏览器默认编码解码。

#### 数据库连接的编解码
若使用jdbc，存取数据时要和数据的内置编码保持一致，可在jdbc的url中设置编码指定。

      url="jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=GBK"
#### js编解码
##### js文件编解码
    <html>
    <head>
    <script src="xxx/a.js" charset="gbk"/>
引入的js文件若有中文，和本html页面的编码若不一致则会乱码，可以手动指定编码格式。

##### js的url编解码
js中发起ajax请求的url默认编码受浏览器不同影响，可使用escape()、encodeURI()、encodeURIComponent()几个函数搞定。
java中的URLEncoder、URLDecoder和js的encodeURIComponent对应。

#### xml文件的编解码
xml文件开始设定encoding

#### velocity模板设置编码
services.VelocityService.input.encoding=UTF-8

#### jsp设置编码
jsp页面开始设置charset
