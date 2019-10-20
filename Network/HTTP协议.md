# HTTP协议

## 1 URI统一资源标识符

### 1.1 什么是URI

- **URL**：Uniform Resource Locator，表示资源的位置， 期望提供查找资源的方法
- **URN**：Uniform Resource Name，期望为资源提供持久 的、位置无关的标识方式，并允许简单地将多个命名空间映射到单个URN命名 空间
  - 例如磁力链接 magnet:?xt=urn:sha1:YNCKHTQC5C 
- **URI**：Uniform Resource Identifier，用以区分资源，是 URL 和 URN 的超集，用以取代 URL 和 URN 概念



严格地说，URI 不完全等同于网址，它包含有 URL 和 URN两个部分，在 HTTP 世界里用的网址实际上是 URL——统一资源定位符（Uniform Resource Locator）。但因为URL 实在是太普及了，所以常常把这两者简单地视为相等。

### 1.2 URI的组成

![](http://base422.oss-cn-beijing.aliyuncs.com/neturi2.png)



- URI 通常由 scheme、host:port、path 和 query 四个部分组成，有的可以省略；
- scheme 叫“方案名”或者“协议名”，表示资源应该使用哪种协议来访问；
- “host:port”表示资源所在的主机名和端口号;
- path 标记资源所在的位置；
- query 表示对资源附加的额外要求；
- 在 URI 里对“**@&/”等特殊字符和汉字必须要做编码，否则服务器收到 HTTP 报文后会无法正确处理**



## 2 常见方法

- **GET**：主要的获取信息方法，大量的性能优化都针对该方法，**幂等方法**
- **HEAD**：类似 GET 方法，但服务器不发送 BODY，用以获取 HEAD 元数据，**幂等方法**
- **POST**：常用于提交 HTML FORM 表单、新增资源等
- **PUT**：更新资源，带条件时是**幂等方法** 
- **DELETE**：删除资源，**幂等方法** 
- **CONNECT**：建立 tunnel 隧道 
- **OPTIONS**：显示服务器对访问资源支持的方法，幂等方法 
- **TRACE**：回显服务器收到的请求，用于定位问题。有安全风险（**已不使用**）



## 3 HTTP Code

### 3.1 分类

RFC 标准把状态码分成了五类，用数字的第一位表示分类:

- **1××**：**请求已接收到，需要进一步处理才能完成，HTTP1.0 不支持**；
- **2××**：**成功**，报文已经收到并被正确处理；
- **3××**：**重定向**，资源位置发生变动，需要客户端重新发送请求;
- **4××**：**客户端错误**，请求报文有误，服务器无法处理;
- **5××**：**服务器错误**，服务器在处理请求时内部发生了错误

### 3.2 1xx

- **100 Continue**：**上传大文件前使用**
- **101 Switch Protocols**：**协议升级使用** ,由客户端发起请求中携带 Upgrade: 头部触发，如升级 websocket 或者 http/2.0

### 3.3 2xx

- **200 OK**:是最常见的成功状态码，表示一切正常，服务器如客户端所期望的那样返回了处理结果，如果是非 HEAD请求，通常在响应头后都会有 body 数据
- **204 No Content**:是另一个很常见的成功状态码，它的含义**与“200 OK”基本相同，但响应头后没有 body 数据**
- **206 Partial Content**：使用 **range 协议时返回部分响应内容时的响应码**，状态码 206 通常还会伴随着头字段“**Content-Range**”，表示响应报文里 body 数据的具体范围，供客户端确认，例如“Content-Range: bytes 0-99/2000”，意思是此次获取的是总计 2000 个字节的前 100 个字节。

### 3.4 3xx

- **301 Moved Permanently**：资源**永久性的重定**向到另一个 URI 中
- **302 Found**：资源**临时的重定向**到另一个 URI 
- **304 Not Modified**：当客户端拥有可能过期的缓存时，会携带缓存的标识 etag、时间等信息询问服务器缓存是否仍可复用，而304是告诉客户端可以**复用缓存**
-  **307 Temporary Redirect**：**类似302**，但明确重定向后请求方法必须**与原请求方法相同**，不得改变
- **308 Permanent Redirect**：**类似301**，但明确重定向后请求方法必须**与原请求方法相同**，不得改变

### 3.5 4xx

- **400 Bad Request**：服务器认为客户端出现了错误，但**不能明确判断为以下哪种错误**时使用此错误码。例如HTTP请求格式错误
- **401 Unauthorized**：**用户认证信息缺失或者不正确**，导致服务器无法处理请求
- **403 Forbidden**：服务器理解请求的含义，但**没有权限执行**此请求
- **404 Not Found**：服务器没有找到对应的资源

### 3.6 5xx
- **500 Internal Server Error**：服务器**内部错误**，且不属于以下错误类型
- **501 Not Implemented**：**服务器不支持实现请求所需要的功能** 
- **502 Bad Gateway**：代理服务器**无法获取到合法响应**
- **503 Service Unavailable**：服务器资源**尚未准备好处理当前请求**
- **504 Gateway Timeout**：代理服务器无法及时的**从上游获得响应**



## 4 HTTP连接流程



![](http://base422.oss-cn-beijing.aliyuncs.com/httpduring.png)

## 5 长连接与短连接

### 5.1 短连接

HTTP 协议最初（0.9/1.0）是个非常简单的协议，通信过程也采用了简单的“请求 - 应答”方式。

**它底层的数据传输基于 TCP/IP，每次发送请求前需要先与服务器建立连接，收到响应报文后会立即关闭连接。**

因为客户端与服务器的整个连接过程很短暂，不会与服务器保持长时间的连接状态，所以就被称为“短连接”（shortlived connections）。早期的 HTTP 协议也被称为是“无连接”的协议。

![](http://base422.oss-cn-beijing.aliyuncs.com/nethttpshort.png)

### 5.2 长连接

针对短连接暴露出的缺点，HTTP 协议就提出了“长连接”的通信方式，也叫“持久连接”（persistent connections）、“连接保活”（keep alive）、“连接复用”（connection reuse）。
其实解决办法也很简单，用的就是“成本均摊”的思路，**既然 TCP 的连接和关闭非常耗时间，那么就把这个时间成本由原来的一个“请求 - 应答”均摊到多个“请求 - 应答”上**。
这样虽然不能改善 TCP 的连接效率，但基于“分母效应”，每个“请求 - 应答”的无效时间就会降低不少，整体传输效率也就提高了。
这里我画了一个短连接与长连接的对比示意图

![](http://base422.oss-cn-beijing.aliyuncs.com/nethttplong.png)

### 5.3 连接相关的头字段

- **Keep-Alive**：长连接
  - 客户端请求长连接 
    	- Connection: Keep-Alive
  - 服务器表示支持长连接
  	- Connection: Keep-Alive
  -  客户端复用连接
  -  HTTP/1.1 默认支持长连接 
- **Close**：短连接
- 对代理服务器的要求
  - 不转发 Connection 列出头部，该头部仅与当前连接相关



### 5.4 Connection作用范围

**Connection 仅针对当前连接有效!**

![](http://base422.oss-cn-beijing.aliyuncs.com/netkeepalive.png)



### 5.5 桥头堵塞问题

因为 HTTP 规定报文必须是“一发一收”，这就形成了一个先进先出的“串行”队列。队列里的请求没有轻重缓急的优先级，只有入队的先后顺序，排在最前面的请求被最优先处理。
**如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本**

![](http://base422.oss-cn-beijing.aliyuncs.com/httpheadblocking.png)

#### 5.5.1 性能优化

因为“请求 - 应答”模型不能变，所以“队头阻塞”问题在HTTP/1.1 里无法解决，只能缓解，有什么办法呢？

**通过HTTP 里的“并发连接”（concurrentconnections）来缓解此问题，也就是同时对一个域名发起多个长连接，用数量来解决质量的问题。**

一个客户端**最多对一个域名发起6~8个并发连接**

若果是多个域名,一共可以发起并发的数量为:**域名数乘以(6~8)**，通过多个域名提高并发的技术我们称为**域名分片**

**总结:“队头阻塞”问题会导致性能下降，可以用“并发连接”和“域名分片”技术缓解**





## 6 HTTP Range规范

### 6.1 Http Range

- 允许服务器基于客户端的请求只发送响应包体的一部分给到客户端，而客户端 自动将多个片断的包体组合成完整的体积更大的包体
  - 支持断点续传

  - 支持多线程下载

  - 支持视频播放器实时拖动
- 服务器通过 Accept-Range 头部表示是否支持 Range 请求
  - Accept-Ranges = acceptable-ranges
  - 例如:
    - Accept-Ranges: bytes：支持 
    - Accept-Ranges: none：不支持

### 6.2 Range 请求范围的单位 

基于字节，设包体总长度为 10000 

- 第 1 个 500 字节：bytes=0-499

- 第 2 个 500 字节
  - bytes=500-999 
  - bytes=500-600,601-999 •
  - bytes=500-700,601-999

- 最后 1 个 500 字节
  -  bytes=-500 
  - bytes=9500

- 仅要第 1 个和最后 1 个字节：bytes=0-0,-1 

**通过Range头部传递请求范围，如：Range: bytes=0-499**

### 6.3 Range 条件请求 

- 如果客户端已经得到了 Range 响应的一部分，并想在这部分响应未过期 的情况下，获取其他部分的响应
  - 常与 If-Unmodified-Since 或者 If-Match 头部共同使用
- If-Range = entity-tag / HTTP-date
  - 可以使用 Etag 或者 Last-Modified

### 6.4 服务器响应

- **206 Partial Content** :**正常返回**,Content-Range 头部：显示当前片断包体在完整包体中的位置
- **416 Range Not Satisfiable** :**请求范围不满足实际资源的大小**，其中 Content-Range 中的 complete- length 显示完整响应的长度，例如:Content-Range: bytes */1234
- **200 OK** :服务器**不支持 Range 请求时**，则以 200 返回完整的响应包体

### 6.5 多重范围与 multipart 

- 请求：
  - Range: bytes=0-50, 100-150
- 响应:
  - Content-Type：multipart/byteranges; boundary=…