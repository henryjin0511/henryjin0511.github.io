---

title: HTTP协议

date: 2016-10-26 21:05:25

categories: 
- http

tags:
- http
- url

---

# HTTP协议

## URL

### URL简介 

URL（Uniform Resoure Locator：统一资源定位符），区别于URI（ Universal Resource Identifier 统一资源标识符）和URN（Universal Resource Name 统一资源名称），URL是URI其中的一个子集
例如：

    * url http://xxx.com/path/xxx.html
    * urn urn:issn:0000-0000  
    
以下内容将主要针对URL展开：

实际使用中一般为三部分，协议+主机+路径（例如：`http://www.xxx.com/xxx/xxx.html`）

### URL语法

    protocol://[username:password@]host:port/path?query#fragment

其他参数

* protocol：协议类型（HTTP/HTTPS/FTP/FILE）
* username：用户名（FTP常用）
* password：密码（FTP常用）
* host：资源服务器的域名或IP地址
* port：服务器监听的端口号，HTTP默认为80，HTTPS默认为443
* path：服务器资源路径，'/'做分隔
* query：查询格式一般为'?k1=v1&k2=v2'
* fragment：信息片段，常用语网页锚点

<!-- more -->

### 扩展

#### JAVASCRIPT URL函数

```javascript
//window代表当前窗口，top为顶级窗口
//返回完整URL字符串
console.log(window.location.href);
//返回协议
console.log(window.location.protocol);
//返回主机
console.log(window.location.host);
//返回端口号（HTTP默认为80，HTTPS默认443，但是此处返回为空字符串）
console.log(window.location.port);
//返回路径
console.log(window.location.pathname);
//返回查询
console.log(window.location.search);
//返回锚点
console.log(window.location.hash);
```

#### PHP URL

```php
//eg:http://localhost/test/index.php?page=5
//返回取域名或主机地址 
echo $_SERVER['HTTP_HOST']."<br>";// localhost
//返回路径
echo $_SERVER['PHP_SELF']."<br>";// test/index.php
//返回查询
echo $_SERVER['QUERY_STRING']."<br>";// test/index.php
//获取完整的url
echo 'http://'.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'];
echo 'http://'.$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'].'?'.$_SERVER['QUERY_STRING'];
```

## 请求

### 请求头信息

    //消息行
    //格式：方法(POST) 资源路径(/meta/) 协议版本(HTTP/1.1)
    POST /meta/ HTTP/1.1
    //此处开始为报文头信息息
    Host: web.webstrom.com
    Connection: keep-alive
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    DNT: 1
    Accept-Encoding: gzip, deflate, sdch
    Accept-Language: zh-CN,zh;q=0.8
    Cookie: name=jm
    //请求体信息
    username=jm&password=123456
    

方法
* GET     请求获取Request-URI所标识的资源
* POST    在Request-URI所标识的资源后附加新的数据
* HEAD    请求获取由Request-URI所标识的资源的响应消息报头
* PUT     请求服务器存储一个资源，并用Request-URI作为其标识
* DELETE  请求服务器删除Request-URI所标识的资源
* TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
* CONNECT 保留将来使用
* OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求.

常用为GET和POST
    
### 响应头信息

    //状态行
    HTTP/1.1 200 OK
    //消息报头
    Date: Wed, 26 Oct 2016 07:46:40 GMT
    Server: Apache/2.2.21 (Win32) mod_ssl/2.2.21 OpenSSL/1.0.0e PHP/5.3.8 mod_perl/2.0.4 Perl/v5.10.1
    X-Powered-By: PHP/5.3.8
    Content-Length: 5289
    Keep-Alive: timeout=5, max=100
    Connection: Keep-Alive
    Content-Type: text/html
    //正文(由服务器返回)

状态码

* 100~199 信息，服务器收到请求，需要请求者继续执行操作 
* 200~299 成功，操作被成功接收并处理 
* 300~399 重定向，需要进一步的操作以完成请求 
* 400~499 客户端错误，请求包含语法错误或无法完成请求 
* 500~599 服务器错误，服务器在处理请求的过程中发生了错误

常用状态码如下：

* 200-请求成功
* 404-请求的资源不存在
* 500-内部服务器错误

## 消息报头详解

* **`Accept`** : 请求报文可通过一个“Accept”报文头属性告诉服务端 客户端接受什么类型的响应。
* **`Cookie`** : 客户端的Cookie就是通过这个报文头属性传给服务端
* **`Referer`** : 表示这个请求是从哪个URL过来的
* **`Cache-Control`** : 对缓存进行控制

更多说明可以参考

1. [HTTP头字段列表（需要科学上网）](https://zh.wikipedia.org/wiki/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5%E5%88%97%E8%A1%A8)

2. [脚本之家总结](http://tools.jb51.net/table/http_header)


## 扩展

HTTP协议是建立在TCP/IP协议基础之上。HTTP通信机制是在一次完整的HTTP通信过程中，Web浏览器与Web服务器之间将完成下列7个步骤：

1. 建立TCP连接

    在HTTP工作开始之前，Web浏览器首先要通过网络与Web服务器建立连接，该连接是通过TCP来完成的，该协议与IP协议共同构建Internet，即著名的TCP/IP协议族，因此Internet又被称作是TCP/IP网络。HTTP是比TCP更高层次的应用层协议，根据规则，只有低层协议建立之后才能，才能进行更层协议的连接，因此，首先要建立TCP连接，一般TCP连接的端口号是80

2. Web浏览器向Web服务器发送请求命令

    一旦建立了TCP连接，Web浏览器就会向Web服务器发送请求命令,例如：GET/sample/hello.jsp HTTP/1.1

3. Web浏览器发送请求头信息

	浏览器发送其请求命令之后，还要以头信息的形式向Web服务器发送一些别的信息，之后浏览器发送了一空白行来通知服务器，它已经结束了该头信息的发送。

4. Web服务器应答

    客户机向服务器发出请求后，服务器会客户机回送应答，HTTP/1.1 200 OK，应答的第一部分是协议的版本号和应答状态码

5. Web服务器发送应答头信息

    正如客户端会随同请求发送关于自身的信息一样，服务器也会随同应答向用户发送关于它自己的数据及被请求的文档。

6. Web服务器向浏览器发送数据

    Web服务器向浏览器发送头信息后，它会发送一个空白行来表示头信息的发送到此为结束，接着，它就以Content-Type应答头信息所描述的格式发送用户所请求的实际数据

7. Web服务器关闭TCP连接

    一般情况下，一旦Web服务器向浏览器发送了请求数据，它就要关闭TCP连接，然后如果浏览器或者服务器在其头信息加入了这行代码Connection:keep-aliveTCP连接在发送后将仍然保持打开状态，于是，浏览器可以继续通过相同的连接发送请求。保持连接节省了为每个请求建立新连接所需的时间，还节约了网络带宽。