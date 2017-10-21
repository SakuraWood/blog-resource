# HTTP协议

## HTTP Request

* 协议格式
GET/sample.jsp HTTP/1.1
Accept:image/gif.image/jpeg,*/*
Accept-Language:zh-cn
Connection:Keep-Alive
Host:localhost
User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)
Accept-Encoding:gzip,deflate

username=jinqiao&password=1234

① 第一行是“方法URL协议版本”
GET/sample.jsp HTTP/1.1
GET代表请求方法，/sample.jsp表示URI，“HTTP/1.1代表协议和协议版本号”

请求方法可有多种，HTTP/1.1支持7中请求方式：
1. GET  请求获取由 Request-URI 所标识的资源。
2. POST  在 Request-URI 所标识的资源后附加新的数据。
3. HEAD  请求获取由 Request-URI 所标识的资源的响应消息报头。
4. OPTIONS  请求查询服务器的性能，或查询与资源相关的选项和需求。
5. PUT  请求服务器存储一个资源，并用 Request-URI 作为其标识。
6. DELETE  请求服务器删除由 Request-URI 所标识的资源。
7. TRACE  请求服务器回送收到的请求信息，主要用语测试或诊断。
其中最常用的是GET和POST。

② 请求头
Accept:image/gif.image/jpeg,*/*
Accept-Language:zh-cn
Connection:Keep-Alive
Host:localhost
User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)
Accept-Encoding:gzip,deflate

③ 请求正文
请求头和请求正文之间是一个空行，这个行非常重要，它表示请求头已经结束，接下来的是请求正文。请求正文中可以包含客户提 交的查询字符串信息：

username=jinqiao&password=1234

## HTTP响应:
### Response Headers
#### Cache-Control：
must-revalidate, no-cache, private。这个值告诉客户端，服务端不希望客户端缓存资源，在下次请求资源时，必须要从新请求服务器，不能从缓存副本中获取资源。  
Cache-Control是响应头中很重要的信息，当客户端请求头中包含Cache-Control:max-age=0请求，明确表示不会缓存服务器资源时,Cache-Control作为作为回应信息，通常会返回no-cache，意思就是说，“不缓存就不缓存呗”；当客户端在请求头中没有包含Cache-Control时，服务端往往会定,不同的资源不同的缓存策略，比如说oschina在缓存图片资源的策略就是Cache-Control：max-age=86400,这个意思是，从当前时间开始，在86400秒的时间内，客户端可以直接从缓存副本中读取资源，而不需要向服务器请求。

#### Connection：
keep-alive，

这个字段作为回应客户端的Connection：keep-alive，告诉客户端服务器的tcp连接也是一个长连接，客户端可以继续使用这个tcp连接发送http请求。关于长连接的更多知识，后面我再详细讲。

#### Content-Encoding:

gzip

告诉客户端，服务端发送的资源是采用gzip编码的，客户端看到这个信息后，应该采用gzip对资源进行解码。

#### Content-Type:

text/html;charset=UTF-8

告诉客户端，资源文件的类型，还有字符编码，客户端通过utf-8对资源进行解码，然后对资源进行html解析。通常我们会看到有些网站是乱码的，往往就是服务器端没有返回正确的编码。

#### Date：

Sun, 21 Sep 2014 06:18:21 GMT

这个是服务端发送资源时的服务器时间，刚开始我不知道GMT是格林尼治所在地的标准时间，以为是服务器的时间错了，还去服务器上查看过时间。http协议中发送的时间都是GMT的，这主要是解决在互联网上，不同时区在相互请求资源的时候，时间混乱问题。

#### Expires:

Sun, 1 Jan 2000 01:00:00 GMT

这个响应头也是跟缓存有关的，告诉客户端在这个时间前，可以直接访问缓存副本，很显然这个值会存在问题，因为客户端和服务器的时间不一定会都是相同的，如果时间不同就会导致问题。所以这个响应头是没有Cache-Control：max-age=***这个响应头准确的，因为max-age=date中的date是个相对时间，不仅更好理解，也更准确。

#### Pragma:no-cache

这个含义与Cache-Control等同。

#### Server：

Tengine/1.4.6

这个是服务器和相对应的版本，只是告诉客户端服务器信息，没有更多的意思。

#### Transfer-Encoding：

chunked

这个响应头告诉客户端，服务器发送的资源的方式是分块发送的。一般分块发送的资源都是服务器动态生成的，在发送时还不知道发送资源的大小，所以采用分块发送，每一块都是独立的，独立的块都能标示自己的长度，最后一块是0长度的，当客户端读到这个0长度的块时，就可以确定资源已经传输完了。

#### Vary:

Accept-Encoding

告诉缓存服务器，缓存压缩文件和非压缩文件两个版本，现在这个字段用处并不大，因为现在的浏览器都是支持压缩的。

