---
layout: article
title: HTTP 学习
date: 2018-03-03 17:04:59
tags:
categories: 
copyright: true
---

# **Reference**

* [Hypertext Transfer Protocol -- HTTP/1.1](https://www.w3.org/Protocols/HTTP/1.1/rfc2616.pdf "https://www.w3.org/Protocols/HTTP/1.1/rfc2616.pdf")
* [超文本传输协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE "https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE")
* [HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2 "https://zh.wikipedia.org/wiki/HTTP/2")
* [Angular 4.x HttpModule 揭秘](https://segmentfault.com/a/1190000009028150 "https://segmentfault.com/a/1190000009028150")
* [聊一聊 cookie](https://segmentfault.com/a/1190000004556040 "https://segmentfault.com/a/1190000004556040")
* [HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP "https://developer.mozilla.org/zh-CN/docs/Web/HTTP")
* [HTTP 请求头与请求体](https://segmentfault.com/a/1190000006689767#articleHeader6 "https://segmentfault.com/a/1190000006689767#articleHeader6")

---

# **超文本传输协议**

超文本传输协议（英文：HyperText Transfer Protocol，缩写：`HTTP`）是一种用于分布式、协作式和超媒体信息系统的应用层协议。`HTTP`是万维网的数据通信的基础。
通常，由`HTTP`客户端发起一个请求，创建一个到服务器指定端口（默认是`80`端口）的`TCP`连接。`HTTP`服务器则在那个端口监听客户端的请求。一旦收到请求，服务器会向客户端返回一个状态，比如`HTTP/1.1 200 OK`，以及返回的内容，如请求的文件、错误消息、或者其它信息。

---

# **报文**

## **请求/响应报文共有内容**

### **常用符号**

#### **空格**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A01.png)

`0x20`

#### **回车符**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A02.png)

`0x0d`

#### **换行符**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A03.png)

`0x0a`

#### **引号**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A04.png)

`0x22`

#### **制表符**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A05.png)

`0x09`

### **协议版本**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A06.png)

### **实体**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A07.png)

#### **实体头**

##### **协议中的实体头**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A08.png)

###### **Allow**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A09.png)

枚举资源所支持的`HTTP`方法的集合。

###### **Content-Encoding**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A010.png)

告知客户端应该怎样解码才能获取在`Content-Type`中标示的媒体类型内容。

###### **Content-Language**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A011.png)

通知客户端应该使用的语言。

###### **Content-Length**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A012.png)

指明发送给接收方的消息主体的大小，即用十进制数字表示的字节数。

###### **Content-Location**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A013.png)

指向的是可供访问的资源的直接地址。

###### **Content-MD5**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A014.png)

###### **Content-Range**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A0121.png)

一个数据片段在整个文件中的位置。

###### **Content-Type**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A016.png)

告诉客户端实际返回的内容的内容类型。

###### **Expires**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A017.png)

指定一个时间，在这个时间之后，`HTTP`响应被认为是过时的。

###### **Last-Modified**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A018.png)

包含源头服务器认定的资源做出修改的时间。

###### **extension-header**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A019.png)

就是消息头。

#### **实体体**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A021.png)

十六进制的数据。

### **消息**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A022.png)

#### **消息头**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A023.png)

##### **Cache-Control**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A0122.png)

通过指定指令来实现缓存机制。

##### **Data**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A025.png)

包含了消息生成的日期和时间。

#### **消息体**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A026.png)

实体体或实体体编码后的。

## **请求报文格式**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A027.png)

### **图形化看请求报文格式**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A028.png)

### **从实际报文看请求报文格式**

报文：

    GET http://google.com/ HTTP/1.1
    Host: google.com
    Proxy-Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
    Accept-Encoding: gzip, deflate
    Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
    Cookie: CONSENT=YES+CN.zh-CN+20180212-01-0; SID=zwW58_4F0D8AzCbilnhAA59AS5u-hSyYn4DVyXcfCX4UYiQymleOqJ2B29o7ztTJ6C0TBw.; HSID=Aytk1MX24fv9cmSGl; APISID=NQCJtL-0Hs86V74b/AwpE5C4kviQOQC7ey; 1P_JAR=2018-3-1-0; NID=124=qmExYKzxr_cAisI1O_YQzj7EZVMl6RIorDC0vFZXYsmurbzi-YsRiYdnsFpNy9wy2e6PlpE67ipRAuvgD9iMMqMuwLzBJgjmbiC2atdGhrNAk52sVzj8B-fCaCKm3TwJV2Z4sWBQXGT1RefjynkwiUji2M4eXHaXXKETVprordDqVpZqxkSZfB8BS0ooJqT4PcnjRk0Hjw-A1xqO8S1yM8XJtmL1a5hyPPLg8PZ9suXNjVcno75ZBaoypOp7igjE3XIQiRAkbejXFbcqfLL97NQP; SIDCC=AAiTGe8ZMSv1kX5ZjP4x1uctVkWNqv7U15nY8YQn0vK2a1yOTVXJM6APXxPJ8ZtXzORc4qw8uw
    

报文对应的十六进制：

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A029.png)

### **请求方法**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A030.png)

#### **GET**

获取数据。

#### **HEAD**

请求资源的`header`信息，并且这些`header`与`GET`方法请求时返回的一致。 该请求方法的一个使用场景是在下载一个大文件前先获取其大小再决定是否要下载，以此可以节约带宽资源。

#### **POST**

发送数据给服务器。请求主体的类型由`Content-Type header`指定。一个`POST`请求通常是通过`HTML`表单发送，并返回服务器的修改结果。

### **Request-URI**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A031.png)

### **请求头**

#### **协议中的请求头**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A032.png)

##### **Accept**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A033.png)

说明客户端可以处理的内容类型（[MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types")），服务器可以从诸多备选项中选择一项进行应用，并使用`Content-Type`响应头通知客户端它的选择。

##### **Accept-Charset**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A034.png)

说明客户端可以处理的字符集类型，服务器可以从诸多备选项中选择一项进行应用，并使用`Content-Type`响应头通知客户端它的选择。

##### **Accept-Encoding**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A035.png)

说明客户端可以处理的内容编码方式（通常是某种压缩算法），服务器可以从诸多备选项中选择一项进行应用，并使用`Content-Encoding`响应头通知客户端它的选择。

##### **Accept-Language**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A036.png)

说明客户端可以处理的自然语言和优先处理的区域方言，服务器可以从诸多备选项中选择一项进行应用，并使用`Content-Language`响应头通知客户端它的选择。

##### **Authorization**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A037.png)

含有服务器用于验证用户代理身份的凭证。

##### **Expect**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A038.png)

表示服务器只有在满足此期望条件的情况下才能妥善地处理请求。

##### **From**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A039.png)

包含一个电子邮箱地址。

##### **Host**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A040.png)

指明了服务器的域名（对于虚拟主机来说），同时可选择设置服务器监听的`TCP`端口号。
如果没有给定端口号，会自动使用被请求服务的默认端口（比如请求一个`HTTP`的`URL`会自动使用`80`端口）。

##### **If-Match**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A041.png)

表示这是一个条件请求。
在请求方法为`GET`和`HEAD`的情况下，服务器仅在请求的资源满足此`header`列出的`ETag`之一时才会返回资源。
而对于`PUT`或其他非安全方法来说，只有在满足条件的情况下才可以将资源上传。

##### **If-Modified-Since**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A042.png)

服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回，状态码为`200`。
如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的`304`响应，而在`Last-Modified`中会带有上次修改时间。
不同于`If-Unmodified-Since`，`If-Modified-Since`只可以用在`GET`或`HEAD`请求中。
当与`If-None-Match`一同出现时，它会被忽略掉，除非服务器不支持`If-None-Match`。

##### **If-None-Match**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A043.png)

对于`GET`和`HEAD`请求方法来说，当且仅当服务器上没有任何资源的`ETag`属性值与这个`header`中列出的相匹配的时候，服务器端才会返回所请求的资源，响应码为`200`。
对于其他方法来说，当且仅当最终确认没有已存在的资源的`ETag`属性值与这个`header`中所列出的相匹配的时候，才会对请求进行相应的处理。

##### **If-Range**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A044.png)

当字段值中的条件得到满足时，`Range`字段才会起作用，同时服务器回复`206`部分内容状态码，以及`Range`字段请求的相应部分；如果字段值中的条件没有得到满足，服务器将会返回`200`状态码，并返回完整的请求资源。

##### **If-Unmodified-Since**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A045.png)

只有当资源在指定的时间之后没有进行过修改的情况下，服务器才会返回请求的资源，或是接受`POST`或其他非安全方法的请求；如果所请求的资源在指定的时间之后发生了修改，那么会返回`412`错误。

##### **Max-Forwards**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A0123.png)

通过`TRACE`方法或`OPTIONS`方法，发送包含`Max-Forwards`的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。
服务器在往下一个服务器转发请求之前，会将`Max-Forwards`的值减`1`后重新赋值。当服务器接收到`Max-Forwards`值为`0`的请求时，则不再进行转发，而是直接返回响应。

##### **Proxy-Authorization**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A046.png)

包含了用户代理提供给代理服务器的用于身份验证的凭证。

##### **Range**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A0147.png)

告知服务器返回文件的哪一部分。在一个`Range`中，可以一次性请求多个部分，服务器会以`multipart`文件的形式将其返回。如果服务器返回的是范围响应，需要使用`206`状态码。假如所请求的范围不合法，那么服务器会返回`416`状态码，表示客户端错误。服务器允许忽略`Range`，从而返回整个文件，状态码用`200`。

##### **Referer**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A0124.png)

包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。

##### **TE**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A048.png)

指定用户代理希望使用的传输编码类型。

##### **User-Agent**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A049.png)

包含了一个特征字符串，用来让网络协议的对端来识别发起请求的用户代理软件的应用类型、操作系统、软件开发商以及版本号。

#### **其他请求头**

##### **Cookie**

    Cookie: <cookie-list>
    Cookie: name=value
    Cookie: name=value; name2=value2; name3=value3

含有先前由服务器通过`Set-Cookie`投放并存储到客户端的`HTTP` `cookies`。

##### **Upgrade-Insecure-Requests**

    Upgrade-Insecure-Requests: 1

表示客户端优先选择加密及带有身份验证的响应，并且它可以成功处理`upgrade-insecure-requests` `CSP`指令。

##### **Origin**
	
	Origin: ""
	Origin: <scheme> "://" <host> [ ":" <port> ]

指示了请求来自于哪个站点。该字段仅指示服务器名称，并不包含任何路径信息。

### **请求体**

就是消息体。
请求体有 3 种类型：

#### **任意类型**

服务器不会解析请求体，请求体的处理需要自己解析。

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A050.png)

如`POST` `JSON`时就是这类，建议使用`Content-Type:application/json`。

#### **application/x-www-form-urlencoded**

多个键值对之间用`&`连接，键与值之前用`=`连接，且只能用`ASCII`字符，非`ASCII`字符需使用`UrlEncode`编码。

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A051.png)

#### **multipart/form-data**

请求体被分成为多个部分，文件上传时会被使用，每个字段/文件都被`boundary`（`Content-Type`中指定）分成单独的段，每段以`--`加 boundary 开头，然后是该段的描述头，描述头之后空一行接内容，请求结束的标志为 boundary 后面加`--`。

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A052.png)

区分是否被当成文件的关键是`Content-Disposition`是否包含`filename`，因为文件有不同的类型，所以还要使用`Content-Type`指示文件的类型，如果不知道是什么类型取值可以为`application/octet-stream`表示该文件是个二进制文件，如果不是文件则`Content-Type`可以省略。

## **响应报文格式**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A053.png)

### **图形化看响应报文格式**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A054.png)

### **从实际报文看响应报文格式**

报文：

    HTTP/1.1 302 Found
    Cache-Control: private
    Content-Type: text/html; charset=UTF-8
    Referrer-Policy: no-referrer
    Location: http://www.google.com.hk/?gfe_rd=cr&dcr=0&ei=w1yXWu_sAabj8weo7rPIDw
    Content-Length: 272
    Date: Thu, 01 Mar 2018 01:52:03 GMT
    Connection: close
    Proxy-Connection: keep-alive
    
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>302 Moved</TITLE></HEAD><BODY>
    <H1>302 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com.hk/?gfe_rd=cr&amp;dcr=0&amp;ei=w1yXWu_sAabj8weo7rPIDw">here</A>.
    </BODY></HTML>
    

报文对应的十六进制：

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A055.png)

### **状态码 & 状态码描述**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A056.png)

### **响应头**

#### **协议中的响应头**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A057.png)

##### **Accept-Ranges**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A058.png)

标识自身支持请求的范围。

##### **Age**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A059.png)

消息对象在缓存代理中存贮的时长，以秒为单位。

##### **ETag**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A061.png)

资源的特定版本的标识符。这可以让缓存更高效，并节省带宽，因为如果内容没有改变，Web 服务器不需要发送完整的响应。

##### **Location**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A062.png)

需要将页面重新定向至的地址。

##### **Proxy-Authenticate**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A063.png)

指定了为获取代理服务器上的资源访问权限而使用的身份验证方式。

##### **Retry-After**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A064.png)

表示用户代理需要等待多长时间之后才能继续发送请求。

##### **Server**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A065.png)

包含了处理请求的源头服务器所用到的软件相关信息。

##### **Vary**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A066.png)

决定了对于未来的一个请求头，应该用一个缓存的回复还是向源服务器请求一个新的回复。

##### **WWW-Authenticate**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A067.png)

定义了使用何种验证方式获取资源的连接。

#### **其他响应头**

##### **Content-Disposition**

1、作为消息主体中的消息头

    Content-Disposition: inline
    Content-Disposition: attachment
    Content-Disposition: attachment; filename="filename.jpg"

在`HTTP`场景中，第一个参数或者是`inline`（默认值，表示回复中的消息体会以页面的一部分或者整个页面的形式展示），或者是`attachment`（意味着消息体应该被下载到本地；大多数浏览器会呈现一个“保存为”的对话框，将`filename`的值预填为下载后的文件名，假如它存在的话）。

2、作为 multipart body 中的消息头

    Content-Disposition: form-data
    Content-Disposition: form-data; name="fieldName"
    Content-Disposition: form-data; name="fieldName"; filename="filename.jpg"

在`HTTP`场景中。第一个参数总是固定不变的`form-data`；附加的参数不区分大小写，并且拥有参数值，参数名与参数值用`=`连接，参数值用`"`括起来。参数之间用`;`分隔。

##### **Set-Cookie**

    Set-Cookie: <cookie-name>=<cookie-value> 
    Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
    Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>
    Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
    Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
    Set-Cookie: <cookie-name>=<cookie-value>; Secure
    Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly
    
    Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
    Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
    
    // Multiple directives are also possible, for example:
    Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly

服务器端向客户端发送`cookie`。

##### **Transfer-Encoding**

![](http://otkw6sse5.bkt.clouddn.com/HTTP-%E5%AD%A6%E4%B9%A068.png)

逐跳传输消息首部，即仅应用于两个节点之间的消息传递，而不是所请求的资源本身。一个多节点连接中的每一段都可以应用不同的`Transfer-Encoding`值。

### **响应体**

就是消息体。

---