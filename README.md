# 前言

这篇文章主要是我平时在学习HTTP过程中看到的一些知识点，现在把他们总结成一篇文章，建立一个自己的知识体系。另外，推荐一本非常棒的HTTP书——《图解HTTP》，这本书图文并茂，挺有趣的。<br>

HTTP是基于TCP/IP协议的应用层协议，用于客户端和服务器之间的通信，默认80端口。我们按照他的发展历程的时间顺序开始说。


# 1. HTTP/0.9 

1990年提出的，是最早期的版本，只有一个命令GET。

# 2. HTTP/1.0

1996年5月提出的。
- `缺点`：每个TCP连接只能发送一个请求。<br>
- `解决方法`：Connection：keep-alive <br>

# 3. HTTP/1.1

1997年1月提出，现在使用最广泛的。

## 3.1 特性

1. `长连接`：TCP连接默认不关闭，可以被多个请求复用。对于同一个域名，大多数浏览器允许同时建立6个持久连接。默认开启Connection：keep-alive。<br>

2. `管道机制`：在同一个TCP连接里，可以同时发送多个请求。但是服务器还是要按照请求的顺序进行响应，会造成“队头阻塞”。<br>

## 3.2 HTTP首部

HTTP首部分为请求报文和响应报文。它们的格式如下所示：<br>

- 请求报文：<br>
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/1.jpg)

- 响应报文：<br>
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/2.jpg)

其中首部字段又分为很多种，我们先看通用首部字段，这是请求报文和响应报文中都会使用的首部。

### 3.2.1 通用首部字段
1、`Cache-Control`：操作缓存的工作机制<br>

参数：

> * public：明确表明其他用户也可以利用缓存
> * private：缓存只给特定的用户
> * no-cache：客户端发送这个指令，表示客户端不接收缓存过的响应，必须到服务器取；服务器返回这个指令，指缓存服务器不能对资源进行缓存。其实是不缓存过期资源，要向服务器进行有效期确认后再处理资源。
> * no-store：指不进行缓存
> * max-age：缓存的有效时间（相对时间）

2、`Connection`：
> * Connection：keep-Alive (持久连接)
> * Connection：不再转发的首部字段名

3、`Date`：表明创建http报文的日期和时间

4、`Pragma`：兼容http1.0，与Cache-Control：no-cache含义一样。但只用在客户端发送的请求中，告诉所有的中间服务器不返回缓存。形式唯一：Pragma：no-cache

5、`Trailer`：会事先说明在报文主体后记录了哪些首部字段，该首部字段可以应用在http1.1版本分块传输编码中。

6、`Transfer-Encoding`：chunked （分块传输编码）,
规定传输报文主体时采用的编码方式，http1.1的传输编码方式只对分块传输编码有效

7、`Upgrade`：升级一个成其他的协议，需要额外指定Connection：Upgrade。服务器可用101状态码作为相应返回。

8、`Via`：追踪客户端和服务器之间的请求和响应报文的传输路径。可以避免请求回环发生，所以在经过代理时必须要附加这个字段。

### 3.2.2 请求首部字段

1、Accept：通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级
q表示优先级的权重值，默认为q = 1.0，范围是0~1（可精确到小数点后3位，1为最大值）
当服务器提供多种内容时，会先返回权重值最高的媒体类型

2、Accept-Charset：支持的字符集及字符集的相对优先顺序，跟Accept一样，用q来表示相对优先级。这个字段应用于内容协商机制的服务器驱动协商。

3、Accept-Encoding：支持的内容编码及内容编码的优先级顺序，q表示相对优先级。
内容编码：gzip、compress、deflate、identity（不执行压缩或者不会变化的默认编码格式）。
可以使用*作为通配符，指定任意的编码格式。

4、Accept-Language：能够处理的自然语言集，以及相对优先级。

## 3.3 状态码

`101 协议升级`，主要用于升级到websocket，也可以用于http2<br>

`200 OK`<br>

`204 No content`，服务器成功处理请求，但是返回的响应报文中不含实体的主体部分<br>

`206 Partial Content`，表示客户端像服务器进行了范围请求（Content-Range字段），服务器成功返回指定范围的实体内容<br>

`301 永久性重定向`，表示请求的资源已经被分配了新的url，旧地址以后都不能再访问了，服务器会返回location字段，包含的是新的地址。<br>

`302 临时性重定向`，表示请求的资源临时移动到一个新地址<br>

> **注意**：尽量使用301跳转，因为302会造成网址劫持，可能被搜索引擎判为可疑转向，甚至认为是作弊。<br>
> **原因**：从网站A（网站比较烂）上做了一个302跳转到网站B（搜索排名很靠前），这时候有时搜索引擎会使用网站B的内容，但却收录了网站A的地址，这样在不知不觉间，网站B在为网站A作贡献，网站A的排名就靠前了。

`303 See Other`，与302功能相同，但是它明确规定客户端应采用GET方法获取资源<br>

`304 未修改`，协商缓存中返回的状态码<br>

`307 临时重定向`，与302功能相同，但规定不能从POST变成GET
> 当301、302、303响应状态码返回时，几乎所有浏览器都会把post改成get，并删除请求报文内的主体，之后请求会自动再次发送。然而301、302标准是禁止将post方法改变成get方法的，但实际使用时大家都会这么做。所以需要307。

`400 Bad Request`，表示请求报文中存在语法错误。当错误发生时，需要修改请求的内容再次发送请求<br>

`401  unauthorized`，表示发送的请求需要有通过HTTP认证（BASIC认证、DIGEST认证）的认证信息。如果之前已经进行过一次请求，表示用户认证失败。<br>

`403  禁止`，表示拒绝对请求资源的访问<br>

`404 Not Found`，表明服务器上无法找到请求的资源<br>

`500 Internet Server Error`，该状态码表示服务器在执行请求时发生了错误<br>

`500 Service Unavailable`，表示服务器暂时处于超负荷或者处于停机维护状态，现在无法处理请求<br>

# 4. SPDY协议

2009年谷歌提出。
- `SPDY结构`：<br>
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/3.jpg)
- `新增特性`：<br>
（1）多路复用：通过一个TCP连接，可以无限制处理多个HTTP请求。<br>
（2）赋予请求优先级：给请求逐个分配优先级顺序。可以解决在发送多个请求时，因带宽低而导致响应变慢的问题。<br>
（3）压缩HTTP首部：压缩方式：DELEFT<br>
（4）推送功能<br>
（5）服务器提示功能：服务器可以主动提示客户端请求所需的资源。<br>
- `缺点`：<br>
SPDY强制使用https。而且SPDY基本上只是将单个域名下的通信多路复用，所以当一个web网站上使用多个域名下的资源时，改善效果就会受到限制。<br>

# 5. WebSocket
html5新提出来的，是web浏览器与web服务器之间的全双工通信标准。主要是为了解决ajax和comet里的xmlhttprequest附带的缺陷所引起的问题。<br>

## 5.1 特性
（1）推送功能：服务器可直接发送数据，不需要等待客户端的请求；<br>
（2）基于TCP传输协议，并复用HTTP的握手通道；<br>
（3）支持双向通信，用于实时传输消息；<br>
（4）更好的二进制支持；<br>
（5）更灵活，更高效。<br>

## 5.2 建立连接过程

**1、客户端：发起协议升级请求**

```javascript
GET / HTTP/1.1      `采用HTTP报文格式，只支持get请求`
Host: localhost:8080    
Origin: http://127.0.0.1:3000 
Connection: Upgrade   `表示要升级协议`
Upgrade: websocket    `表示升级到websocket协议`
Sec-WebSocket-Version: 13    `表示websocket 的版本`
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==   `是一个 Base64 encode 的值，是浏览器随机生成的`
Sec-WebSocket-Protocol：chat, superchat  `用来指定一个特定的子协议，一旦这个字段有设置，那么服务器需要在建立连接的响应头中包含同样的字段，内容就是选择的子协议之一。`
```
**2、服务端：响应协议升级**

```javascript
HTTP/1.1 101 Switching Protocols    `101表示协议切换==`
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU= `经过服务器确认，并且加密过后的 Sec-WebSocket-Key`
Sec-WebSocket-Protocol：chat  `表示最终使用的协议`
```
Sec-WebSocket-Key 的加密过程为：<br>
> 1. 将Sec-WebSocket-Key跟258EAFA5-E914-47DA-95CA-C5AB0DC85B11拼接。
> 2. 通过SHA1计算出摘要，并转成base64字符串。

**3、双方握手成功后，就是全双工的通信了，接下来就是用websocket协议来进行通信了。**

## 5.3 Ajax 轮询、长轮询、WebSocket原理解析

1、`ajax轮询`<br>

让浏览器每隔一定的时间就发送一次请求，询问服务器是否有新信息。<br>

2、`长轮询（Long Poll）`<br>

采用的阻塞模式。客户端发起连接后，如果没消息，服务器不会马上告诉你没消息，而是将这个请求挂起（pending），直到有消息才返回。返回完成或者客户端主动断开后，客户端再次建立连接，周而复始。Comet就是采用的长轮询。<br>

3、`websocket`<br>

 WebSocket 是类似 Socket 的TCP长连接通讯模式。一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。而且浏览器和服务器就可以随时主动发送消息给对方，是全双工通信。<br>
 
 优点：在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。<br>
 
# 6. HTTP2
2015年发布，它是基于SPDY的，以下是它的一些新特性：<br>

1、 `二进制分帧`：<br>

http/1.x 是一个超文本协议，而 http2 是一个二进制协议，被称之为二进制分帧。
二进制格式在协议的解析和优化扩展上带来更多的优势和可能。<br>

协议格式为帧，帧由 Frame Header（头信息帧）和 Frame Payload（数据帧）组成，如下所示：
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/4.jpg)
* Length 字段用来表示 Frame Payload 数据大小。
* Type 字段用来表示该帧中的 Frame Payload 保存的是 header 数据还是 body 数据。除了用于标识 header/body，还有一些额外的 Frame Type。
* Stream Identifier 用来标识该 frame 属于哪个 stream。
* Frame Payload 用来保存 header 或者 body 的数据。

2、`头部压缩 HPACK`：<br>

请求和响应首部压缩，客户端和服务端共同维护一张头信息表，所有字段存入这个表，生成一个索引号，通过发送索引号提高速度。HPACK压缩会经过两步:
> * 传输的value,会经过一遍Huffman coding来节省资源；
> * 为了server和client同步, 两边都需要保留一份Header list, 并且,每次发送请求时,都会检查更新。<br>

3、`服务端推送`：<br>

服务端主动向客户端推送数据。如果客户端请求一个html文件，服务端把html文件返回给客户端之后，还会相应的把html文件中的js、css、图片推送给客户端。<br>

4、`多路复用`：<br>

只需要建立一个TCP连接，浏览器和服务器可以同时发送多个请求或者回应，而且不需要按照顺序一一对应，避免了“队头阻塞”。<br>

5、`数据流`：<br>

当客户端同时向服务端发起多个请求，那么这些请求会被分解成一一个的帧，每个帧都会在一个 TCP 链路中无序的传输，同一个请求的帧的 Stream Identifier 都是一样的。当帧到达服务端之后，就可以根据 Stream Identifier 来重新组合得到完整的请求。<br>

并且规定：客户端发出的数据流ID为奇数，服务器发出的ID为偶数。Stream Identifier （数据流ID）就是用来标识该帧属于哪个请求的。

# 7. HTTPS

> HTTPS = HTTP+加密+认证+完整性保护
> 
它的加密过程是：

1. server生成一个公钥和私钥，把公钥发送给第三方认证机构（CA）；
2. CA把公钥进行MD5加密，生成数字签名；再把数字签名用CA的私钥进行加密，生成数字证书。CA会把这个数字证书返回给server；
3. server拿到数字证书之后，就把它传送给浏览器；
4. 浏览器会对数字证书进行验证，首先，浏览器本身会内置CA的公钥，会用这个公钥对数字证书解密，验证是否是受信任的CA生成的数字证书；
5. 验证成功后，浏览器会随机生成对称秘钥，用server的公钥加密这个对称秘钥，再把加密的对称秘钥传送给server；
6. server收到对称秘钥，会用自己的私钥进行解密，之后，它们之间的通信就用这个对称秘钥进行加密，来维持通信。<br>

下图是加密过程的图解，可以对照着图片理一遍。<br>
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/5.jpg)

# 8. HTTP缓存机制

## 8.1 缓存分类

HTTP的缓存分为强缓存和协商缓存（对比缓存）。<br>

1. `强制缓存`<br>

在缓存数据未失效的情况下，可以直接使用缓存数据；在没有缓存数据的时候，浏览器向服务器请求数据时，服务器会将数据和缓存规则一并返回，缓存规则信息包含在响应header中。<br>

- Expires：缓存过期时间（HTTP1.0）<br>

    缺点：生成的是绝对时间，但是客户端时间可以随意修改，会导致误差。
- Cache-Control ：HTTP1.1，优先级高于Expires<br>
 
    可设置参数：<br>
> private:           客户端可以缓存<br>
> public:            客户端和代理服务器都可缓存<br>
> max-age=xxx:       缓存的内容将在 xxx 秒后失效<br>
> no-cache:          需要使用协商缓存来验证缓存数据（后面介绍）<br>
> no-store:          所有内容都不会缓存，强制缓存，对比缓存都不会触发<br>

Expires和Cache-Control决定了浏览器是否要发送请求到服务器，ETag和Last-Modified决定了服务器是要返回304+空内容还是新的资源文件。<br>

2. `协商缓存`

浏览器第一次请求数据时，服务器会将缓存标识与数据一起返回给客户端，客户端将二者备份至缓存数据库中。再次请求数据时，客户端将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回304状态码，通知客户端比较成功，可以使用缓存数据。<br>

- Last-Modified / If-Modified-Since<br>
    
> `Last-Modified`：服务器在响应请求时，告诉浏览器资源的最后修改时间。<br>`If-Modified-Since`：再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改时间。<br>

缺点：Last-Modified 标注的最后修改时间只能精确到秒，如果有些资源在一秒之内被多次修改的话，他就不能准确标注文件的新鲜度了。如果某些资源会被定期生成，当内容没有变化，但 Last-Modified 却改变了，导致文件没使用缓存有可能存在服务器没有准确获取资源修改时间，或者与代理服务器时间不一致的情形。
- Etag / If-None-Match（优先级高于Last-Modified / If-Modified-Since）<br>

> `Etag`：给资源计算得出的一个唯一标志符。<br>`If-None-Match`：再次请求服务器时，通过此字段通知服务器客户端缓存数据的唯一标识。<br>

## 8.2 缓存判断顺序

1. 先判断Cache-Control，在Cache-Control的max-age之内，直接返回200 from cache；
2. 没有Cache-Control再判断Expires，再Expires之内，直接返回200 from cache；
3. Cache-Control=no-cache或者不符合Expires，浏览器向服务器发送请求；
4. 服务器同时判断ETag和Last-Modified，都一致，返回304，有任何一个不一致，返回200。<br>

具体过程如下图：<br>
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/6.jpg)

# 9. 跨域

跨域产生的原因，是因为受到同源策略的限制。同源策略指的是协议、域名、端口不相同。这里我将介绍三种跨域的方式：JSONP、CORS(跨域资源共享)、document.domain + iframe。<br>

## 9.1 JSONP

**1. 原理**<br>

动态插入script标签（因为script标签不受同源策略的限制），通过插入script标签引入一个js文件，这个js文件加载成功之后会执行我们在url中指定的回调函数，并且会把我们需要的json数据作为参数传入。<br>

**2. 实现**<br>

（1）原生实现：

 ```javascript
var script = document.createElement('script');
script.type = 'text/javascript';

// 传参并指定回调执行函数为onBack
script.src = 'http://www.domain2.com:8080/login?user=admin&callback=onBack';
document.head.appendChild(script);

// 回调函数
function onBack(res) {
    alert(JSON.stringify(res));
}
    
//服务端返回如下（返回时即执行全局函数）：
onBack({"status": true, "user": "admin"})
```

（2）jquery ajax：

```javascript
$.ajax({
    url: 'http://www.domain2.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "onBack",    // 自定义回调函数名
    data: {}
});
```

## 9.2 CORS

**1. 原理**<br>

服务器在响应头中设置相应的选项，浏览器如果支持这种方法的话就会将这种跨站资源请求视为合法，进而获取资源。<br>

**2. 实现**<br>

CORS分为简单请求和复杂请求，简单请求指的是：<br>

（1）请求方法是以下三种方法之一：HEAD、GET、POST；<br>

（2）HTTP的头信息不超出以下几种字段：
Accept、Accept-Language、Content-Language、Last-Event-ID、
Content-Type（只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain）。<br>

其他情况就是非简单请求了。

- **简单请求**<br>

（1）`请求头`：

```javascript
Origin: http://www.domain.com
```
（2）`响应头`：
```javascript
Access-Control-Allow-Origin: http://www.domain.com
Access-Control-Allow-Credentials: true   `是否允许传送cookie`
Access-Control-Expose-Headers: FooBar `CORS请求时，只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须指定。`
```
（3）另外，`ajax请求中`,如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名，还要设置以下内容：
```javascript
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```
- **非简单请求**<br>

（1）`预检请求`:
```javascript
OPTIONS /cors HTTP/1.1  `OPTIONS请求是用来询问的`
Origin: http://www.domian.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
```
（2）`响应头`：
```javascript
Access-Control-Allow-Origin: http://www.domain.com
Access-Control-Allow-Methods: GET, POST, PUT  `服务器支持的所有跨域请求的方法`
Access-Control-Allow-Headers: X-Custom-Header  `服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。`
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000  `指定本次预检请求的有效期，单位为秒`
```
（3）`之后的步骤就同简单请求了`。<br>

这是CORS的整个流程图：
![Alt text](https://github.com/ddduanlian/http_note/raw/master/assets/7.jpg)

**与JSONP的比较**：<br>

    JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

## 9.3 document.domain + iframe
此方案仅限主域相同，子域不同的跨域应用场景。<br>

**1.原理**<br>

两个页面都通过js强制设置document.domain为基础主域，就实现了同域。

**2.实现**<br>
（1）父窗口：(www.domain.com/a.html)

```
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>

<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>
```



（2）子窗口：(child.domain.com/b.html)
```
<script>
    document.domain = 'domain.com';
    // 获取父窗口中变量
    alert('get js data from parent ---> ' + window.parent.user);
</script>
```
---

先写这么多，后续我会继续补充的，像cookie，session那些我以后会加进来的。
