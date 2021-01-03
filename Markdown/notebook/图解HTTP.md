# 图解HTTP
## 第二章 简单的HTTP协议
### 2.1 HTTP协议用于客户端和服务端之间的通信
### 2.2 通过请求和响应的交换达成通信
### 2.3 HTTP是不保存状态的的协议
### 2.4 请求URI定位资源
### 2.5 告知服务器意图的HTTP的方法
* GET: 获取资源
* POST: 传输实体主体
* PUT: 传输文件
* HEAD: 获得报文首部
* DELETE: 删除文件
* OPTION: 询问支持的方法
* TRACE: 跟踪路径
* CONNECT: 要求使用隧道协议连接代理：CONNECT方法要求在于代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。主要使用**SSL**跟**TLS**协议把通信内容加密后经网络隧道传输。
> SSL: Secure Sockets Layer, 安全套接层  
> TLS: Transport Layer Security, 传输层安全

* LINK: 建立与资源之间的联系（1.1已废弃）
* UNLINK: 断开连接关系（1.1已废弃）
### 2.6 使用方法下发命令
### 2.7 持久连接节省通信量
* **持久连接（HTTP PERISISTENT CONNECTIONS）**
> 只要任意一端没有明确提出断开连接，则保持TCP连接。`Connection: keep-alive`

* **管线化(pipelining)**
> 持久连接使得多数请求以管线化方式发送成为可能。管线化技术，其实就是将请求从同步发送转化为异步发送。

### 2.8 使用Cookie的状态管理
> 1.Http是**无状态**的协议，它不对之前发生过的请求和响应的状态进行管理， 也就是说，无法根据之前的状态进行本次的请求处理。  
> 2.**无状态的优点**：
>
* 1、减少服务器保存状态的资源损耗；
* 2、简化协议，易于应用；

> 3.Set-Cookie = xxx
## 第三章 HTTP报文内的HTTP信息
### 3.1 HTTP报文
### 3.2 请求报文及响应报文的结构
> 由**报文首部**、**空行（CR+LF）**、**报文主体**组成，其中报文主体不是必要选项  
> 报文首部包括：**请求行**/**状态行**，**请求首部字段**/**响应首部字段**，**通用首部字段**，**实体首部字段**，**其他**  
> **其他**：可能包含HTTP的RFC里未定义的首部，比如Cookie
### 3.3 编码提升传输速率
* 报文主体和实体主体的差异  
> **报文（message）**：是http通信中的基本单位，由8位组字节流（octet sequence）组成，通过http通信传输。  
> **实体（entity）**：作为请求或响应的有效载荷数据（补充项）被传输，其内容是由实体首部和实体主体组成。  
> HTTP报文的主体用于传输请求或者响应的实体主体。  
> 通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异。

* 压缩传输的内容编码
> 常用的内容编码有以下几种：  
>
* gzip(GNU zip)
* compress(UNIX系统标准压缩)
* deflate(zlib)
* identity(不进行编码)

* 分割发送的分块传输编码  
>
* 在http通信过程中，请求的编码实体资源尚未全部传输完成之前，浏览器无法显示请求页面。在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。这种把实体主体分块的功能称为**分块传输编码（Chunked Transfer Coding）**。
* **分块传输编码**会将实体主体分成多个部分（块）。每一块使用16进制来标记块的大小，而实体的最后一块会使用“0（CR+LF）”来标记。
* HTTP/1.1中存在一种称为**传输编码（Transfer Coding）**的机制，它可以在通信时按某种编码方式传输，但只定义作用于**分块传输编码**中。

### 3.4 发送多种数据的多部分对象集合
> multipart/form-data:在web表单文件上传时使用   
```xml
    Content-Type: multipart/form-data; **boundary**=AaB03x  
    --AaB03x  
    Content-Disposition: form-data; name="field1"  
    Joe Blow  
    --AaB03x  
    Content-Disposition: form-data; name="pics"; filename="file1.txt"  
    Content-Type=text/plain
    ...(file1.txt的数据)  
    --AaB03x--
```
> multipart/byteranges：状态码206（Partial Content,部分内容），响应报文包含了多个范围时使用  
```xml
    HTTP/1.1 206 Partial Content  
    Date: Fri, 13 Jul 2012 02:45:26 GMT  
    Last-Modify: Fri, 31 Aug 2007 02:02:20 GMT  
    **Content-Type**: mutilpart/byteranges; **boundary**="THIS_STRING_SEPARATES"  
    --THIS_STRING_SEPARATES  
    **Content-Type**: application/pdf  
    **Content-Range**: bytes 500-999/8000  
    ...(范围指定的数据)  
    --THIS_STRING_SEPARATES  
    **Content-Type**: application/pdf  
    **Content-Range**:bytes 7000-7999/8000  
    ...(范围指定的数据)  
    --THIS_STRING_SEPARATES--
```
### 3.5 获取部分内容的范围请求
* 5001-10000字节：Range: bytes=5001-10000
* 5001字节之后的全部:Range: byte=5001-
* 从一开始到3000字节和5000-7000字节的多重范围: Range: bytes=-3000, 5000-7000  
> 针对范围请求，响应会返回状态码为206Partial Content的响应报文。另外，针对多重范围请求，响应会在首部字段Content-Type标明multipart/byteranges后返回响应报文。如果服务器端无法响应范围请求，则会返回状态吗200 OK和完整的实体内容

### 3.6 内容协商返回最合适的内容
    内容协商机制是指客户端和服务端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。
    内容协商会以响应资源的*语言 ，**字符集 ，**编码方式\*等作为判断的基准。
    1、Accept
    2、Accept-Charset
    3、Accept-Encoding
    4、Accept-Language
    5、Content-Language
内容协商技术有一下三种类型：
* **服务器协商驱动**（Server-driven Negotiation）：由服务端进行内容协商。以请求的首部字段为参考，在服务器端自动处理。但对用户来说，以服务器发送的信息来作为判定的依据，并不一定能筛选出最优内容。
* **客户端驱动协商**(Agent-driven Negotiation)：由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中手动选择。还可以利用JS脚本在Web页面上自动进行上述选择。比如按OS的类型或者浏览器类型，自行切换成PC版页面或者手机版页面。
* **透明协商**（Transparent Negotiation）:是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法。

## 第四章 返回结果的HTTP状态码
### 4.1 状态码告知从服务器端返回的请求结果
状态码|类别|原因短语
---|---|---
1XX|Informational（信息性状态码）|接收的请求正在处理
2XX|Success（成功状态码）|请求正常处理完毕
3XX|Redirection（重定向状态码）|需要进行附加操作以完成请求
4XX|CLient Error（客户端错误状态码）|服务器无法处理请求
5XX|Server Error（服务器错误状态码）|服务器处理请求出错

状态码|状态信息|概述
---|---|---
200|OK|客户端发来的请求在服务器端被正常处理了。
204|Not Content|服务器接收的请求已成功处理,<br>但返回的响应报文中不含实体的主体部分。
206|Partial Content|表示客户端进行了范围请求,<br>而服务器成功执行了这部分的Get请求,<br>响应报文中包含由Content-Range指定范围的实体内容。
301|Moved Permanently|永久性重定向，刷新书签。
302|Found|临时性重定向。<br>不允许更换方法（POST、GET等），但实际上并不遵守。
303|See Other|请求对应资源存在另外一个URI,<br>应当使用**GET**方法定向获取请求的资源。
304|Not Modified|客户端发送请求附带条件:<br>If-Match<br>If-Modified-Since<br>If-None-Match<br>If-Range<br>If-Unmodified-since<br>服务器允许请求访问资源，但未满足条件（**该错误码跟重定向没有关系**）。
307|Temporary Redirect|临时重定向。与302有相同含义，但必须严格遵照浏览器标准，<br>并且对于处理响应时的行为，每种浏览器有可能出现不同的情况。
400|Bad Request|请求报文中存在语法错误。
401|Unauthorized|表示发送的请求需要通过HTTP认证（BASIC认证、DIGEST认证）。
403|Forbidden|对资源的访问请求被服务器拒绝了。
404|Not Found|服务器无法找到请求的资源。
500|Internal Server Error|服务器端执行请求时发生了错误。
503|Service Unavailable|服务器暂时处于超负载或者正在进行停机维护，<br>现在无法处理请求。
## 第五章 与HTTP协作的Web服务器
### 5.1 用单台虚拟主机实现多个域名
### 5.2 通信数据转发程序：代理、网关、隧道
#### 5.2.1 代理
    使用代理服务器的理由有：利用缓存技术减少网络带宽的流量，组织内部针对特定网站的访问控制，以获取访问日志为主要目的，等等。  
    代理有多种使用方法，按两种基准分类：1、缓存代理；2、透明代理。
#### 5.2.2 网关
    网关的工作机制和代理十分相似，但网关能使通信线路上的服务器提供非HTTP协议服务。
    利用网关能提高通信的安全性，因此可以在客户端和网关之间的通信线路上加密以确保连接的安全。比如，网关可以连接数据库，使用SQL语句查询数据。
    另外在Web购物网站上进行信用卡结算时，网关可以和信用卡结算系统联动。   
#### 5.2.3 隧道
    隧道可按要求建立起一条与其他服务器的通信线路，届时使用SSL等加密手段进行通信。隧道的目的是确保客户端能与服务器进行安全的通信。
    隧道本身不会去解析HTTP请求，也就是说，请求保持原样中转给之后的服务器，隧道会在通信双方断开连接时结束。
### 5.3 保存资源的缓存

## 第六章 HTTP首部
### 6.1 HTTP报文首部
* 请求首部
* 响应首部
* 通用首部
* 实体首部
### 6.2 HTTP首部字段
#### 6.2.1 HTTP首部字段传递重要信息
#### 6.2.2 HTTP首部字段结构
> key : value  
> Content-Type : text/html  
> Keep-Alive : timeout=15, max = 100
#### 6.2.3 4种HTTP首部字段类型
#### 6.2.4 HTTP/1.1 首部字段一览
* 通用首部字段    

首部字段名|说明
---|---
Cache-Control|控制缓存的行为
Connection|逐跳首部、连接的管理
Date|创建报文的日期时间
Pragma|报文指令
Trailer|报文末段的首部一览
Transfer-Encoding|指定报文主体的传输编码方式
Upgrade|升级为其他协议
Via|代理服务器的相关信息
Warning|错误通知
* 请求首部字段

首部字段名|说明
---|---
Accept|用户代理可处理的媒体类型
Accept-Charset|优先的字符集
Accept-Encoding|优先的内容编码
Accept-Language|优先的语言（自然语言）
Authorization|Web认证信息
Expect|期待服务器的特定行为
From|用户的电子邮箱地址
Host|请求资源所在的服务器
If-Match|比较实体标记（ETag）
If-Modified-Since|比较资源的更新时间
If-None-Match|比较实体标记（与If-Match相反）
If-Range|资源未更新时发送实体 Byte 的范围请求
If-Unmodified-Since|比较资源的更新时间（ 与If-Modified-Since相反）
Max-Forwards|最大传输逐跳数
Proxy-Authorization|代理服务器要求客户端的认证信息
Range|实体的字节范围请求
Referer|对请求中 URI 的原始获取方
TE|传输编码的优先级
User-Agent|HTTP 客户端程序的信息
* 响应首部字段

首部字段名|说明
---|---
Accept-Ranges|是否接受字节范围请求
Age|推算资源创建经过时间
ETag|资源的匹配信息
Location|令客户端重定向至指定URI
Proxy-Authenticate|代理服务器对客户端的认证信息
Retry-After|对再次发起请求的时机要求
Server|HTTP服务器的安装信息
Vary|代理服务器缓存的管理信息
WWW-Authenticate|服务器对客户端的认证信息
* 实体首部字段

首部字段名|说明
---|---
Allow|资源可支持的HTTP方法
Content-Encoding|实体主体适用的编码方式
Content-Language|实体主体的自然语言
Content-Length|实体主体的大小（ 单位： 字节）
Content-Location|替代对应资源的URI
Content-MD5|实体主体的报文摘要
Content-Range|实体主体的位置范围
Content-Type|实体主体的媒体类型
Expires|实体主体过期的日期时间
Last-Modified|资源的最后修改日期时间
#### 6.2.5 非HTTP/1.1首部字段
> 在HTTP协议通信交互中使用到的首部字段，不限于RFC2616中定义的47种首部字段。还有Cookie、SetCookie和Content-Disposition等在其他RFC中定义的首部字段，他们的使用频率也很高。
#### 6.2.6 End-to-end首部和Hop-by-hop首部
> HTTP首部字段将定义成缓存代理和非缓存代理的行为，分成2种类型  
>* 端到端首部（End-to-end Header）:分在此类别中的首部会转发给请求/响应对应的最终接收目标，且必须保存在由缓存生成的响应中，另外规定它必须被转发。
>* 逐跳首部（Hop-by-hop Header）：分在此类别中的首部只对单次转发有效，会因为通过缓存或代理而不再转发。在HTTP/1.1和之后的版本中，如果要使用hop-by-hop首部，
需提供Connection首部字段

> 下面列举了HTTP/1.1中逐跳首部字段，除了这八个首部字段外，其他所有字段都属于端到端首部
>* Connection
>* Keep-Alive
>* Proxy-Authenticate
>* Proxy-Authorization
>* Trailer
>* TE
>* Transfer-Encoding
>* Upgrade
### 6.3 HTTP/1.1 通用首部字段
#### 6.3.1 Cache-Control
> 缓存请求指令

指令|参数|说明  
---|---|---  
no-cache|无|强制向源服务器再次验证
no-store|无|不缓存请求或响应的任何内容
max-age=（秒）|必须|响应的最大age值
max-stale=（秒）|可省略|接收已过期的响应
min-fresh=（秒）|必须|期望在指定时间内的响应仍有效
no-transform|无|代理不可更改媒体类型
only-if-cached|无|从缓存获取资源
cache-extension|-|新指令标记（token）
> 缓存响应指令

指令|参数|说明  
---|---|---  
public|无|可向任意方提供响应的缓存
private|可省略|仅向特定用户返回响应
no-cache|可省略|缓存前必须先确认其有效性
no-store|无|不缓存请求或响应的任何内容
no-transform|无|代理不可更改媒体类型
must-revalidate|无|可缓存但必须再向源服务器进行确认
proxy-revalidate|无|要求中间缓存服务器对缓存的响应有效性再进行确认
max-age=（秒）|必需|响应的最大Age值
s-maxage=（秒）|必需|公共缓存服务器响应的最大Age值
cache-extension|-|新指令标记（token）

> 表示是否能缓存的指令  
>* public:明确标明其他用户也可以利用缓存  
>* private:与public行为相反，只对特定用户提供缓存服务  
>* no-cache：客户端发送的请求携带该字段表示需要从服务器中获取数据；服务器返回的报文携带该字段，表示缓存服务器不得缓存数据  

> 控制可执行缓存的对象的指令  
>* no-store:不进行缓存

> 指定缓存期限和认证的指令  
>* s-maxage:s-maxage 指令的功能和 max-age 指令的相同， 它们的不同点是 smaxage 指令只适用于供多位用户使用的公共缓存服务器 。 也就是说，对于向同一用户重复返回响应的服务器来说， 这个指令没有任何作用。另外， 当使用 s-maxage 指令后， 则直接忽略对 Expires 首部字段及max-age 指令的处理。
>* max-age:当客户端发送的请求中包含 max-age 指令时， 如果判定缓存资源的缓存时间数值比定时间的数值更小，那么客户端就接收缓存的资源。
另外， 当指定max-age值为0，那么缓存服务器通常需要将请求转发给源服务器。
>* min-fresh:要求缓存服务器返回至少还未过指定时间的缓存资源。
>* max-stale:指示缓存资源，即使过期也照常接收。
>* only-if-cached:使用 only-if-cached 指令表示客户端仅在缓存服务器本地缓存目标资
源的情况下才会要求其返回。换言之，该指令要求缓存服务器不重新加载响应，也不会再次确认资源有效性。若发生请求缓存服务器的本地缓存无响应， 则返回状态码 504 Gateway Timeout。
>* must-revalidate:使用 must-revalidate 指令， 代理会向源服务器再次验证即将返回的响
应缓存目前是否仍然有效。
>* proxy-revalidate:要求所有的缓存服务器在接收到客户端带有该指令的请求返回响应之前，必须再次验证缓存的有效性。
>* no-transform:使用no-transform指令规定无论是在请求还是响应中，缓存都不能改变实体主体的媒体类型。这样做可防止缓存或代理压缩图片等类似操作。

> Cache-Control扩展
>* cache-extension token:通过cache-extension 标记（ token），可以扩展 Cache-Control 首部字段内的指令。
#### 6.3.2 Connection
> Connection首部字段具备如下两个作用：
>* 控制不再转发给代理的首部字段
>* 管理持久连接
#### 6.3.3 Date
> 首部字段Date表明创建HTTP报文的日期和时间。
#### 6.3.4 Pragma
> Pragma 是 HTTP/1.1之前版本的历史遗留字段， 仅作为与 HTTP/1.0的向后兼容而定义
#### 6.3.5 Trailer
> 首部字段 Trailer会事先说明在报文主体后记录了哪些首部字段。 该首部字段可应用在 HTTP/1.1 版本分块传输编码时。  
```xml
HTTP/1.1 200 OK  
Date: Tue, 03 Jul 2012 04:40:56 GMT  
Content-Type: text/html  
...  
Transfer-Encoding: chunked  
Trailer: Expires  
...(报文主体)...  
0  
Expires: Tue, 28 Sep 2004 23:59:59 GMT
```
> 以上用例中，指定首部字段 Trailer的值为 Expires，在报文主体之后（分块长度0之后）出现了首部字段Expires。
#### 6.3.6 Transfer-Encoding
> 首部字段 Transfer-Encoding规定了传输报文主体时采用的编码方式。
#### 6.3.7 Upgrade
> 首部字段 Upgrade用于检测HTTP协议及其他协议是否可使用更高的版本进行通信，其参数值可以用来指定一个完全不同的通信协议。
#### 6.3.8 Via
> 使用首部字段 Via 是为了追踪客户端与服务器之间的请求和响应报文
的传输路径。
#### 6.3.9 Warning
> HTTP/1.1 的Warning首部是从HTTP/1.0的响应首部（ Retry-After）演变过来的。该首部通常会告知用户一些与缓存相关的问题的警告。

警告码|警告内容|说明
---|---|---
110|Response is stale（ 响应已过期）|代理返回已过期的资源
111|Revalidation failed（ 再验证失败） |代理再验证资源有效性时失败（服务器无法到达等原因）
112|Disconnection operation（断开连接操作）|代理与互联网连接被故意切断
113|Heuristic expiration（ 试探性过期） |响应的使用期超过24小时（ 有效缓存的设定时间大于24小时的情况下）
199|Miscellaneous warning（ 杂项警告） |任意的警告内容
214|Transformation applied（ 使用了转换） |代理对内容编码或媒体类型等执行了某些处理时
299|Miscellaneous persistent warning（ 持久杂项警告）|任意的警告内容
### 6.4 请求首部字段
#### 6.4.1 Accept
> Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级。可使用 type/subtype这种形式，一次指定多种媒体类型。
>* 文本文件
> text/html, text/plain, text/css ...
> application/xhtml+xml, application/xml ...
>* 图片文件
> image/jpeg, image/gif, image/png ...
>* 视频文件
> video/mpeg, video/quicktime ...
>* 应用程序使用的二进制文件
> application/octet-stream, application/zip ...
#### 6.4.2 Accept-Charset
> Accept-Charset首部字段可用来通知服务器用户代理支持的字符集及字符集的相对优先顺序。另外， 可一次性指定多种字符集。 与首部字段Accept 相同的是可用权重q值来表示相对优先级。**该首部字段应用于内容协商机制的服务器驱动协商**。
#### 6.4.3 Accept-Encoding
> Accept-Encoding首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先级顺序。可一次性指定多种内容编码。
> 下面试举出几个内容编码的例子。
>* gzip  
 由文件压缩程序gzip（GNUzip）生成的编码格式
（ RFC1952），采用 Lempel-Ziv 算法（ LZ77） 及 32 位循环冗余校验（ Cyclic Redundancy Check， 通称 CRC） 。
>* compress  
由 UNIX 文件压缩程序 compress 生成的编码格式， 采用 LempelZiv-Welch 算法（ LZW） 。
>* deflate  
组合使用 zlib 格式（ RFC1950） 及由 deflate 压缩算法
（ RFC1951） 生成的编码格式。
>* identity  
不执行压缩或不会变化的默认编码格式
#### 6.4.4 Accept-Language
#### 6.4.5 Authorization
> 首部字段 Authorization 是用来告知服务器， 用户代理的认证信息（证书值）。
#### 6.4.6 Expect
> 客户端使用首部字段 Expect 来告知服务器， 期望出现的某种特定行为。因服务器无法理解客户端的期望作出回应而发生错误时， 会返回状态码 417 Expectation Failed。

> 客户端可以利用该首部字段，写明所期望的扩展。 虽然 HTTP/1.1 规范只定义了 100-continue（ 状态码 100 Continue 之意）。等待状态码 100 响应的客户端在发生请求时，需要指定Expect:100-continue。
#### 6.4.7 From
#### 6.4.8 Host
#### 6.4.9 If-Match
> 匹配ETag
#### 6.4.10 If-Modified-Since
#### 6.4.11 If-None-Match
#### 6.4.12 If-Range
> 它告知服务器若指定的 IfRange 字段值（ ETag 值或者时间） 和请求资源的 ETag 值或时间相一
致时， 则作为范围请求处理。反之，则返回全体资源。
#### 6.4.13 If-Unmodified-Since
#### 6.4.14 Max-Forwards
#### 6.4.15 Proxy-Authorization
> 接收到从代理服务器发来的认证质询时， 客户端会发送包含首部字段Proxy-Authorization的请求，以告知服务器认证所需要的信息。这个行为是与客户端和服务器之间的HTTP访问认证相类似的，不同之处在于，认证行为发生在客户端与代理之间。客户端与服务器之间的认证，使用首部字段Authorization可起到相同作用。
#### 6.4.16 Range
#### 6.4.17 Referer
> 首部字段 Referer会告知服务器请求的原始资源的 URI。
#### 6.4.18 TE
> 首部字段 TE会告知服务器客户端能够处理响应的传输编码方式及相对优先级。它和首部字段Accept-Encoding 的功能很相像，但是用于传输编码。

> 首部字段TE除指定传输编码之外，还可以指定伴随trailer字段的分块传输编码的方式。应用后者时，只需把trailers赋值给该字段值。
#### 6.4.189 User-Agent
> 首部字段 User-Agent会将创建请求的浏览器和用户代理名称等信息传达给服务器。

### 6.5 响应实体首部
#### 6.5.1 Accept-Ranges
> 首部字段 Accept-Ranges是用来告知客户端服务器是否能处理范围请求，以指定获取服务器端某个部分的资源。可指定的字段值有两种，可处理范围请求时指定其为 bytes， 反之则指定其为none。
#### 6.5.2 Age
> 首部字段 Age能告知客户端，源服务器在多久前创建了响应。字段值的单位为秒。
#### 6.5.3 ETag
#### 6.5.4 Location
> 使用首部字段Location可以将响应接收方引导至某个与请求 URI 位置不同的资源。
#### 6.5.5 Proxy-Authenticate
#### 6.5.6 Retry-After
#### 6.5.7 Server
> 首部字段Server告知客户端当前服务器上安装的 HTTP服务器应用程序的信息。不单单会标出服务器上的软件应用名称，还有可能包括版本号和安装时启用的可选项。
#### 6.5.8 Vary
> 从代理服务器接收到源服务器返回包含Vary指定项的响应之后，若再要进行缓存，仅对请求中含有相同 Vary 指定首部字段的请求返回缓存。即使对相同资源发起请求，但由于Vary指定的首部字段不相同，因此必须要从源服务器重新获取资源。
#### 6.5.9 WWW-Authenticate
> 首部字段WWW-Authenticate用于HTTP访问认证。 它会告知客户端适用于访问请求URI所指定资源的认证方案（Basic或是Digest）和带参数提示的质询（challenge）。状态码401 Unauthorized响应中，肯定带有首部字段WWW-Authenticate。
### 6.6 实体首部字段
#### 6.6.1 Allow
> 首部字段Allow用于通知客户端能够支持Request-URI 指定资源的所有HTTP方法。当服务器接收到不支持的HTTP方法时，会以状态码405 Method Not Allowed 作为响应返回。与此同时，还会把所有能支持的HTTP方法写入首部字段Allow后返回。
#### 6.6.2 Content-Encoding
#### 6.6.3 Content-Language
#### 6.6.4 Content-Length
> 首部字段 Content-Length表明了实体主体部分的大小（ 单位是字节）。对实体主体进行内容编码传输时， 不能再使用Content-Length首部字段。
#### 6.6.5 Content-Location
#### 6.6.6 Content-MD5
#### 6.6.7 Content-Range
#### 6.6.8 Content-Type
> Content-Type: text/html; charset=UTF-8  
> 首部字段 Content-Type说明了实体主体内对象的媒体类型。 和首部字段 Accept一样，字段值用 type/subtype 形式赋值。参数 charset 使用 iso-8859-1 或 euc-jp 等字符集进行赋值。
#### 6.6.9 Expires
> 首部字段Expires会将资源失效的日期告知客户端。 缓存服务器在接收到含有首部字段 Expires 的响应后， 会以缓存来应答请求， 在Expires 字段值指定的时间之前，响应的副本会一直被保存。 当超过指定的时间后，缓存服务器在请求发送过来时， 会转向源服务器请求资源。
#### 6.6.10 Last-Modified
### 6.7 为 Cookie 服务的首部字段
#### 6.7.1 Set-Cookie
属性|说明
---|---
NAME=VALUE|赋予 Cookie 的名称和其值（ 必需项）
expires=DATE|Cookie 的有效期（ 若不明确指定则默认为浏览器关闭前为止）
path=PATH|将服务器上的文件目录作为Cookie的适用对象（ 若不指定则默认为文档所在的文件目录）
domain=域名|作为 Cookie 适用对象的域名 （ 若不指定则默认为创建 Cookie的服务器的域名）
Secure|仅在 HTTPS 安全通信时才会发送 Cookie
HttpOnly|加以限制， 使 Cookie 不能被 JavaScript 脚本访问
#### 6.7.2 Cookie
> Cookie: status=enable
### 6.8 其他首部字段
#### 6.8.1 X-Frame-Options
#### 6.8.2 X-XSS-Protection
#### 6.8.3 DNT
#### 6.8.4 P3P
## 第7章 确保 Web 安全的HTTPS
### 7.1 HTTP 的缺点
> HTTP 主要有这些不足， 例举如下。
>* 通信使用明文（ 不加密），内容可能会被窃听
>* 不验证通信方的身份，因此有可能遭遇伪装
>* 无法证明报文的完整性，所以有可能已遭篡改
#### 7.1.1 通信使用明文可能会被窃听
>* TCP/IP 是可能被窃听的网络:讲述了为何会被窃听
>* 加密处理防止被窃听:2种方式（通道加密，实体加密）
#### 7.1.2 不验证通信方的身份就可能遭遇伪装
>* 任何人都可发起请求
>* 查明对手的证书
#### 7.1.3 无法证明报文完整性， 可能已遭篡改
>* 接收到的内容可能有误
>* 如何防止篡改
### 7.2 HTTP+加密+认证+完整性保护=HTTPS
#### 7.2.1 HTTP加上加密处理和认证以及完整性保护后即是HTTPS
>* 加密处理：通道加密
>* 认证：预置证书
>* 完整性保护
#### 7.2.2 HTTPS是身披SSL外壳的HTTP
#### 7.2.3 相互交换秘钥的公开秘钥加密技术
>* 公开密钥：安全，但效率低
>* 共享秘钥：非安全，但效率高
#### 7.2.4 证明公开密钥正确性的证书
>* 可证明组织真实性的EV SSL证书
>* 用以确认客户端的的客户端证书：1、证书额外费用；2、安全系数相对较低（安装证书的的设备就具备证书使用权限）
>* 认证机构信誉第一
>* 由自认证机构颁发的证书称为自签名证书
#### 7.2.5 HTTPS的安全通信机制
![image](https://raw.githubusercontent.com/LinHongbo/pic/master/%E5%9B%BE%E8%A7%A3HTTP/https.png)
>1、客户端通过发送 Client Hello 报文开始 SSL通信。 报文中包含客户端支持的 SSL的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法及密钥长度等）。  
2、服务器可进行SSL通信时，会以Server Hello报文作为应答。和客户端一样，在报文中包含SSL版本以及加密组件。
服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的。  
3、之后服务器发送Certificate 报文。报文中包含公开密钥证书。  
4、最后服务器发送Server Hello Done报文通知客户端，最初阶段的SSL握手协商部分结束。  
5、SSL第一次握手结束之后，客户端以Client Key Exchange报文作为回应。报文中包含通信加密中使用的一种被称为
Pre-master
secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密。  
6、接着客户端继续发送 Change Cipher Spec 报文。 该报文会提示服务器， 在此报文之后的通信会采用 Pre-master secret 密钥加密。  
7、客户端发送 Finished 报文。该报文包含连接至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确解密该报文作为判定标准。  
8、服务器同样发送 Change Cipher Spec 报文。  
9、服务器同样发送Finished报文。  
10、服务器和客户端的Finished报文交换完毕之后，SSL连接就算建立完成。当然，通信会受到SSL的保护。从此处开始进行应用层协议的通信，即发送HTTP 请求。  
11、应用层协议通信，即发送HTTP响应。  
12、最后由客户端断开连接。断开连接时，发送close_notify报文。上图做了一些省略，这步之后再发送TCP FIN 报文来关闭与 TCP的通信。

在以上流程中，应用层发送数据时会附加一种叫做 MAC（Message
Authentication Code）的报文摘要。MAC能够查知报文是否遭到篡改，从而保护报文的完整性。
![image](https://raw.githubusercontent.com/LinHongbo/pic/master/%E5%9B%BE%E8%A7%A3HTTP/digitalCertificate.png)

**为什么不一直使用HTTPS:一方面需要增加通信过程中的开销；另外一方面服务端、客户端均需要进行加解密操作也需要消耗更多的CPU、内存等资源**

## 第八章 确认访问用户身份的认证
### 8.1 何为认证
> 核对的信息通常是指以下这些。
>* 密码： 只有本人才会知道的字符串信息。
>* 动态令牌： 仅限本人持有的设备内显示的一次性密码。
>* 数字证书： 仅限本人（ 终端） 持有的信息。
>* 生物认证： 指纹和虹膜等本人的生理信息。
>* IC 卡等： 仅限本人持有的信息。

> HTTP 使用的认证方式, HTTP/1.1 使用的认证方式如下所示。
>* BASIC认证（基本认证）
>* DIGEST认证（摘要认证）
>* SSL客户端认证
>* FormBase认证（基于表单认证）

### 8.2 BASIC认证
### 8.3 DIGEST认证
### 8.4 SSL客户端认证
#### 8.4.1 SSL客户端认证的认证步骤
> 为达到SSL客户端认证的目的，需要事先将客户端证书分发给客户端，且客户端必须安装此证书。  
> 1、接收到需要认证资源的请求， 服务器会发送 Certificate
Request 报文， 要求客户端提供客户端证书。  
> 2、用户选择将发送的客户端证书后， 客户端会把客户端证书信息以 Client Certificate 报文方式发送给服务器。  
> 3、服务器验证客户端证书验证通过后方可领取证书内客户端的公开密钥， 然后开始 HTTPS 加密通信。
#### 8.4.2 SSL客户端认证采用双因素认证
> 1、使用证书确定合法设备；  
> 2、使用账号密码确定本人行为；
#### 8.4.3 SSL客户端认证必要的费用
> 1、证书费用；
> 2、运营费用；
### 8.5 基于表单认证
#### 8.5.1 认证多半为基于表单认证
#### 8.5.2 Session管理以及Cookie应用
![image](https://raw.githubusercontent.com/LinHongbo/pic/master/%E5%9B%BE%E8%A7%A3HTTP/session%20and%20cookie.png)
> 如果Session ID被第三方盗走，对方就可以伪装成你的身份进行恶意操作了。因此必须防止Session ID被盗，或被猜出。为了做到这点，Session ID应使用难以推测的字符串，且服务器端也需要进行有效期的管理，保证其安全性。另外，为减轻跨站脚本攻击（XSS） 造成的损失，建议事先在Cookie内加上httponly属性。

> 另外，不仅基于表单认证的登录信息及认证过程都无标准化的方法，服务器端应如何保存用户提交的密码等登录信息等也没有标准化。
> 通常， 一种安全的保存方法是，先利用给密码加盐（salt）的方式增加额外信息，再使用散列（hash）函数计算出散列值后保存。 但是我们也经常看到直接保存明文密码的做法， 而这样的做法具有导致密码泄露的风险。
> salt其实就是由服务器随机生成的一个字符串，但是要保证长度足够长，并且是真正随机生成的。然后把它和密码字符串相连接（前后都可以）生成散列值。当两个用户使用了同一个密码时，由于随机生成的salt值不同，对应的散列值也将是不同的。这样一来，很大程度上减少了密码特征，攻击者也就很难利用自己手中的密码特征库进行破解。

## 第九章 基于HTTP的功能追加协议
### 9.1 基于HTTP的协议
### 9.2 消除HTTP瓶颈的SPDY
#### 9.2.1 HTTP的瓶颈
> 以下HTTP标准会成为瓶颈
>* 一条连接上只可发送一个请求
>* 请求只能从客户端开始，客户端不可以接收除响应以外的指令
>* 请求/响应首部未经压缩就发送；首部信息越多延迟越大
>* 发送冗长的首部，每次互相发送相同的首部造成的浪费较多
>* 可任意选择数据压缩格式，非强制压缩发送

> Ajax的解决方案  
&emsp;&emsp;Ajax(Asynchronous JavaScript and XML，异步JavaScript与XML技术)是一种有效利用JavaScript和DOM(Document Object Model，文档对象模型)的操作，以达到局部Web页面替换加载的异步通信手段。和以前的同步通信相比， 由于它只更新一部分页面，响应中传输的数据量会因此而减少，这一优点显而易见。  
&emsp;&emsp;Ajax的核心技术是名为XMLHttpRequest的API，通过JavaScript脚本语言的调用就能和服务器进行HTTP通信。借由这种手段，就能从已加载完毕的Web页面上发起请求，只更新局部页面。而利用Ajax实时地从服务器获取内容，有可能会导致大量请求产生。另外，Ajax仍未解决HTTP协议本身存在的问题。

> Comet的解决方法  
&emsp;&emsp;一旦服务器端有内容更新了，Comet不会让请求等待，而是直接给客户端返回响应。这是一种通过延迟应答，模拟你实现服务器端向客户端推送(Server Push)的功能。  
&emsp;&emsp;通常，服务器端接收到请求，在处理完毕后会立即返回响应，但为了实现推送功能，Comet会先将响应置于挂起状态，当服务器端有内容更新时，  再返回该响应。因此，服务器端一旦有更新，就可以立刻反馈给客户端。  
&emsp;&emsp;内容上虽然可以做到实时更新，但为了保留响应，一次连接的持续时间也变长了。期间，为了维持连接会消耗更多的资源。另外Comet也仍未解决HTTP协议本身存在的问题。
#### 9.2.2 SPDY的设计与功能
> SPDY没有完全改写HTTP协议，而是在TCP/IP的应用层与运输层之间通过新加会话层的形式运作。同时，考虑到安全性问题，**SPDY规定通信中使用SSL**。SPDY以会话层的形式加入，控制对数据的流动，但还是采用HTTP建立通信连接。因此，可照常使用HTTP的GET和POST等方法、Cookie以及HTTP报文等。
![image](https://raw.githubusercontent.com/LinHongbo/pic/master/%E5%9B%BE%E8%A7%A3HTTP/SPDY.png)

> 使用SPDY后，HTTP协议额外获得以下功能
>* 多路复用流  
&emsp;&emsp;通过单一的TCP连接，可以无限制处理多个HTTP请求，所有请求都是在一条TCP连接上上完成的，因此TCP的处理效率得到提高.
>* 赋予请求优先级  
&emsp;&emsp;SPDY不仅可以无限制地并发处理请求，还可以给请求逐个分配优先级顺序。这样主要是为了在发送多个请求时，解决因带宽低而导致响应变慢的问题。
>* 压缩HTTP首部  
&emsp;&emsp;压缩HTTP请求和响应的首部。这样一来，通信产生的数据包数量和发送的字节数就更少了。
>* 推送功能  
&emsp;&emsp;支持服务器主动向客户端推送数据的功能。这样，服务器可直接发送数据，而不必等待客户端的请求。
>* 服务器提示功能
&emsp;&emsp服务器可以主动提示客户端请求所需的资源。由于在客户端发现资源之前就可以获知资源的存在，因此在资源已缓存等情况下，可以避免发送不必要的请求。

#### 9.2.3 SPDY消除Web瓶颈了吗
> SPDY基本上只是将单个域名（IP地址）的通信多路复用，所以当一个Web网站上使用多个域名下的资源，改善效果就会受到限制。
### 9.3 使用浏览器进行全双工通信的WebSocket
&emsp;&emsp;利用Ajax和Comet技术进行通信可以提升Web的浏览速度。但问题在于通信若使用HTTP协议，就无法彻底解决瓶颈问题。WebSocket网络技术正是为解决这些问题而实现的一套新协议及API。当时筹划将WebSocket作为HTML5标准的一部分，而现在它却逐渐变成了独立的协议标准。WebSocket通信协议在2011年12月11日，被RFC 6455-The WebSocket Protocol 定为标准。
#### 9.3.1 WebSocket的设计与功能  
&emsp;&emsp;WebSocket，即Web浏览器与Web服务器之间全双工通信标准。其中，WebSocket协议由IETF定为标准，WebSocket API 由W3C定为标准。仍在开发中的WebSocket技术主要是为了解决Ajax和Comet里XMLHttpRequest附带的缺陷所引起的问题。
#### 9.3.2 WebSocket协议  
&emsp;&emsp;一旦Web服务器与客户端之间建立起WebSocket协议的通信连接，之后所有的通信都依靠这个专用协议进行。通信过程中可互相发送JSON、XML、HTML或图片等任意格式的数据。由于是建立在HTTP基础上的协议，因此连接的发起方仍是客户端，而一旦确立WebSocket通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文。
> WebSocket 协议的主要特点:
>* 推送功能：支持由服务器向客户端推送数据的推送功能。这样，服务器可直接发送数据，而不必等待客户端的请求。
>* 减少通信量：只要建立起WebSocket连接，就希望一直保持连接状态。和HTTP相比，不但每次连接时的总开销减少，而且由于WebSocket的首部信息很小，通信量也相应减少了。

> 为了实现WebSocket通信， 在HTTP连接建立之后，需要完成一次“握手”（Handshaking）的步骤。
>* 握手·请求
```xml
GET /chat HTTP/1.1  
Host: server.example.com  
**Upgrade: websocket**  
Connection: Upgrade  
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==  
Origin: http://example.com  
Sec-WebSocket-Protocol: chat, superchat  
Sec-WebSocket-Version: 13
```
>* 握手·响应  
```xml
HTTP/1.1 **101 Switching Protocols**  
Upgrade: websocket  
Connection: Upgrade  
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  
Sec-WebSocket-Protocol: chat  
```
### 9.4 期盼已久的HTTP/2.0
### 9.5 Web服务器管理文件的WebDAV
&emsp;&emsp;WebDAV（Web-based Distributed Authoring and Versioning，基于万维网的分布式创作和版本控制）是一个可对Web服务器上的内容直接进行文件复制、编辑等操作的分布式文件系统。 它作为扩展 HTTP/1.1的协议定义在RFC4918。  
&emsp;&emsp;除了创建、删除文件等基本功能，它还具备文件创建者管理、文件编辑过程中禁止其他用户内容覆盖的加锁功能，以及对文件内容修改的版本控制功能。  
&emsp;&emsp;使用HTTP/1.1的PUT方法和DELETE方法，就可以对Web服务器上的文件进行创建和删除操作。可是出于安全性及便捷性等考虑，一般不使用。
#### 9.5.1 扩展HTTP/1.1的WebDAV
## 第十章 构建Web内容的技术
### 10.1 HTML
#### 10.1.1 Web页面几乎 全由HTML构建
#### 10.1.2 HTML的版本
#### 10.1.3 设计应用CSS
### 10.2 动态HTML
#### 10.2.1 让Web页面动起来的动态HTML
&emsp;&emsp;所谓动态HTML（Dynamic HTML），是指使用客户端脚本语言将静态的HTML内容变成动态的技术的总称。 鼠标单击点开的新闻、Google Maps等可滚动的地图就用到了动态HTML。动态HTML技术是通过调用客户端脚本语言JavaScript， 实现对HTML的Web页面的动态改造。利用DOM（Document Object
Model，文档对象模型）可指定欲发生动态变化的HTML元素。
#### 10.2.2 更容易控制HTML的DOM
&emsp;&emsp;DOM是用以操作HTML文档和XML文档的API（Application
Programming Interface，应用编程接口）。使用DOM可以将HTML内的元素当作对象操作，如取出元素内的字符串、改变那个CSS的属性等，使页面的设计发生改变。  
&emsp;&emsp;通过调用JavaScript等脚本语言对DOM的操作，可以以更为简单的方式控制HTML的改变。
### 10.3 Web应用
#### 10.3.1 通过Web提供功能的Web应用
#### 10.3.2 与Web服务器及程序写作的CGI
&emsp;&emsp;CGI（Common Gateway Interface，通用网关接口）是指Web 服务器在接收到客户端发送过来的请求后转发给程序的一组机制。在CGI的作用下，程序会对请求内容做出相应的动作，比如创建HTML等动态内容。使用CGI的程序叫做CGI 程序，通常是用Perl、PHP、Ruby和C等编程语言编写而成。
#### 10.3.3 因Java而普及的Servlet
### 10.4 数据发布的格式及语言
#### 10.4.1 可扩展性标记语言
#### 10.4.2 发布更新信息的RSS/Atom
#### 10.4.3 JavaScript衍生的轻量级易用的JSON
## 第11章 Web的攻击技术
### 11.1 针对Web的攻击技术
#### 11.1.1 HTTP不具备必要的安全功能
#### 11.1.2 在客户端即可篡改请求
#### 11.1.3 针对Web应用的攻击模式
对Web应用的攻击模式有以下两种：1、主动攻击； 2、被动攻击
* 以服务器为目标的主动攻击
### 11.2 因输出值转义不完全引发的安全漏洞
实施Web应用的安全对策可大致分为以下两部分：1、客户端JavaScript验证；2、服务器端验证（输入值验证、输出值转义）
### 11.2.1 跨站脚本攻击
&emsp;&emsp;跨站脚本攻击（Cross-Site Scripting，XSS）是指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行的一种攻击。 动态创建的 HTML部分有可能隐藏着安全漏洞。就这样，攻击者编写脚本设下陷阱，用户在自己的浏览器上运行时，一不小心就会受到被动攻击。
> 跨站脚本攻击有可能造成以下影响:
>* 利用虚假输入表单骗取用户个人信息
>* 利用脚本窃取用户的 Cookie 值，被害者在不知情的情况下，帮助攻击者发送恶意请求
>* 显示伪造的文章或图片
#### 11.2.2 SQL注入攻击
#### 11.2.3 OS命令注入攻击
#### 11.2.4 HTTP首部注入攻击
#### 11.2.5 邮件首部注入攻击
#### 11.2.6 目录遍历攻击
#### 11.2.7 远程文件包含漏洞
关联的第三方url有问题
### 11.3 因设置或设计上的缺陷引发的安全漏洞
#### 11.3.1 强制浏览
&emsp;&emsp;强制浏览（Forced Browsing）安全漏洞是指，从安置在Web服务器的公开目录下的文件中，浏览那些原本非自愿公开的文件。
#### 11.3.2 不正确的错误消息处理
#### 11.3.3 开放重定向
### 11.4 因会话管理疏忽引发的安全漏洞
#### 11.4.1 会话劫持
&emsp;&emsp;会话劫持（Session Hijack）是指攻击者通过某种手段拿到了用户的会话ID，并非法使用此会话ID伪装成用户，达到攻击的目的。
> 下面列举了几种攻击者可获得会话ID的途径
>* 下面列举了几种攻击者可获得会话ID的途径。
>* 通过窃听或XSS攻击盗取会话ID
>* 通过会话固定攻击（Session Fixation）强行获取会话ID
#### 11.4.2 会话固定攻击
&emsp;&emsp;对以窃取目标会话ID为主动攻击手段的会话劫持而言，会话固定攻击（Session Fixation）攻击会强制用户使用攻击者指定的会话 ID，属于被动攻击。
#### 11.4.3 跨站点请求伪造
&emsp;&emsp;跨站点请求伪造（Cross-Site Request Forgeries，CSRF）攻击是指攻击者通过设置好的陷阱，强制对已完成认证的用户进行非预期的个人信息或设定信息等某些状态更新，属于被动攻击。
### 11.5 其他安全漏洞
#### 11.5.1 密码破解
#### 11.5.2 点击劫持
#### 11.5.3 DoS攻击
&emsp;&emsp;DoS 攻击（Denial of Service attack 是一种让运行中的服务呈停止状态的攻击。有时也叫做服务停止攻击或拒绝服务攻击。DoS攻击的对象不仅限于Web网站，还包括网络设备及服务器等。
#### 11.5.4 后门程序
