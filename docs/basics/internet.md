# 网络基础

## 网络分层协议

![网络分层协议](https://p.ipic.vip/l32re6.png ':size=70%')

根据不同体系，存在多种网络层级的划分模型，以下就 OSI 七层协议进行讲解：

OSI（Open System Interconnection Reference Model / 开放系统互联参考模型）由国际标准化组织ISO于1984年提出，OSI七层参考模型的各个层次的划分遵循下列原则：

- 同一层中的各网络节点都有相同的层次结构，具有同样的功能。
- 同一节点内相邻层之间通过接口(可以是逻辑接口)进行通信。
- 七层结构中的每一层使用下一层提供的服务，并且向其上层提供服务。
- 不同节点的同等层按照协议实现对等层之间的通信

### 第一层：物理层(PhysicalLayer)

规定通信设备的机械、电气、功能和过程的特性，用以建立、维护和拆除物理链路连接。在这一层，数据的单位称为比特(bit)。属于物理层定义的典型规范代表包括：EIA/TIA RS-232、EIA/TIA RS-449、V.35、RJ-45等。

### 第二层：数据链路层(DataLinkLayer)

在物理层提供比特流服务的基础上，建立相邻结点之间的数据链路，通过差错控制提供数据帧(frame)在信道上无差错的传输，在不可靠的物理介质上提供可靠的传输。该层的作用包括：物理地址寻址、数据的成帧、流量控制、数据的检错、重发等。在这一层，数据的单位称为帧(frame)。数据链路层协议的代表包括：SDLC、HDLC、PPP、STP、帧中继等。

### 第三层：网络层(Network)

在计算机网络中进行通信的两个计算机之间可能会经过很多个数据链路，也可能还要经过很多通信子网。网络层的任务就是选择合适的网间路由和交换结点， 确保数据及时传送。网络层将数据链路层提供的帧组成数据包，包中封装有网络层包头，其中含有逻辑地址信息（源站点）和目的站点地址的网络地址。如果你在谈论一个 IP 地址，那么你是在处理第三层的问题，这是“数据包”问题，而不是第2层的“帧”。IP 是第三层问题的一部分，此外还有一些路由协议和地 址解析协议(ARP)。有关路由的一切事情都在这第三层处理。地址解析和路由是3层的重要目的。网络层还可以实现拥塞控制、网际互连等功能。在这一层，数据的单位称为数据包(packet)。网络层协议的代表包括：IP、IPX、RIP、OSPF等。

### 第四层：传输层(Transport)

第四层的数据单元也称作数据包(packets)。但是，当谈论 TCP 等具体的协议时又有特殊的叫法，TCP 的数据单元称为段 (segments)，而UDP协议的数据单元称为数据报(datagrams)。这个层负责获取全部信息，因此，它必须跟踪数据单元碎片、乱序到达的 数据包和其它在传输过程中可能发生的危险。第四层为上层提供端到端(最终用户到最终用户)的透明的、可靠的数据传输服务。所为透明的传输是指在通信过程中 传输层对上层屏蔽了通信传输系统的具体细节。传输层协议的代表包括：TCP、UDP、SPX 等。

### 第五层：会话层(Session)

这一层也可以称为会晤层或对话层，在会话层及以上的高层次中，数据传送的单位不再另外命名，而是统称为报文。会话层不参与具体的传输，它提供包括访问验证和会话管理在内的建立和维护应用之间通信的机制。如服务器验证用户登录便是由会话层完成的。

### 第六层: 表示层（Presentation）

这一层主要解决拥护信息的语法表示问题。它将欲交换的数据从适合于某一用户的抽象语法，转换为适合于 OSI 系统内部使用的传送语法。即提供格式化的表示和转换数据服务。数据的压缩和解压缩， 加密和解密等工作都由表示层负责。

### 第七层: 应用层（Application）

应用层为操作系统或网络应用程序提供访问网络服务的接口。应用层协议的代表包括：Telnet、FTP、HTTP、SNMP 等。

## HTTP

### HTTP 状态码

- 1XX：接收的请求正在处理 
  - 100 Continue：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应
- 2XX：请求正常处理完毕 
  - 200 OK
  - 204 No Content : 请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在不需要返回数据时使用
  - 206 Partial Content： 表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容
- 3XX：需要进行附加操作以完成请求 
  - 301 Moved Permanently：永久性重定向
  - 302 Found：临时性重定向
  - 303 See Other：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源
  - 304 Not Modifies：如果请求报文首部包含一些条件，例如: If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码
- 4XX：服务器无法处理请求 
  - 400 BadRequest：请求报文中存在语法错误
  - 401 Unauthorized：该状态码表示发送的请求需要有认证信息(BASIC 认证、DIGEST 认证)。如果之前已进行过一次请求，则表示用户认证失败
  - 403 Forbidden：请求被拒绝
  - 404 Not Found
- 5XX：服务器处理请求出错 
  - 500 Internal Server Error：服务器正在执行请求时发生错误
  - 503 Service Unavailable：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求

### Cookies

HTTP 协议是无状态的，主要是为了让 HTTP 协议尽可能简单，使得它能够处理大量事务。HTTP/1.1 引入 Cookie 来保存状态信息。

Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，浏览器向同一服务器再次发起请求时会携带 Cookie，告知两个请求来自同一浏览器。由于之后每次请求都会需要携带 Cookie 数据，因此会带来额外的性能开销（尤其是在移动环境下）。

Cookie 曾一度用于客户端数据的存储，因为当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie 渐渐被淘汰。新的浏览器 API 已经允许开发者直接将数据存储到本地，如使用 Web storage API （本地存储和会话存储）或 IndexedDB。

Cookies 用途

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题、个性公告等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

### Session

除了可以将用户信息通过 Cookie 存储在用户浏览器中，也可以利用 Session 存储在服务器端，存储在服务器端的信息更加安全。

Session 可以存储在服务器上的文件、数据库或者内存中。也可以将 Session 存储在 Redis 这种内存型数据库中，效率会更高。

使用 Session 维护用户登录状态的过程如下:

- 用户进行登录时，用户提交包含用户名和密码的表单，放入 HTTP 请求报文中；
- 服务器验证该用户名和密码；
- 如果正确则把用户信息存储到 Redis 中，它在 Redis 中的 Key 称为 Session ID；
- 服务器返回的响应报文的 Set-Cookie 首部字段包含了这个 Session ID，客户端收到响应报文之后将该 Cookie 值存入浏览器中；
- 客户端之后对同一个服务器进行请求时会包含该 Cookie 值，服务器收到之后提取出 Session ID，从 Redis 中取出用户信息，继续之前的业务操作。

应该注意 Session ID 的安全性问题，不能让它被恶意攻击者轻易获取，那么就不能产生一个容易被猜到的 Session ID 值。此外，还需要经常重新生成 Session ID。在对安全性要求极高的场景下，例如转账等操作，除了使用 Session 管理用户状态之外，还需要对用户进行重新验证，比如重新输入密码，或者使用短信验证码等方式。

### Cookies 和 Session 的选择

- Cookie 只能存储 ASCII 码字符串，而 Session 则可以存取任何类型的数据，因此在考虑数据复杂性时首选 Session；
- Cookie 存储在浏览器中，容易被恶意查看。如果非要将一些隐私数据存在 Cookie 中，可以将 Cookie 值进行加密，然后在服务器进行解密；
- 对于大型网站，如果用户所有的信息都存储在 Session 中，那么开销是非常大的，因此不建议将所有的用户信息都存储到 Session 中。

## DNS （Domain Name System）

DNS 提供的功能：

1. 主机名到 IP 地址的转换
2. 主机别名（一个主机可以有一个规范主机名和多个主机别名）
3. 邮件服务器别名
4. 负载均衡（一个IP地址集合可以对应于同一个规范主机名）

DNS 缓存：

一旦名字服务器获得 DNS 映射, 它将缓存该映射到局部内存。服务器在一定时间后将丢弃缓存的信息。本地 DNS服务器可以缓存 TLD 服务器的IP地址，因此根 DNS 服务器不会被经常访问。

## UDP

UDP 是运输层协议，其特点如下：

- 无连接：在 UDP 接收者发送者之间没有握手，因此延时较低；每个 UDP 数据段的处理独立于其他数据段
- 无拥塞控制：应用层能够更好的控制要发送的数据和发送时间，网络中的拥塞控制也不会影响主机的发送速率。某些实时应用要求以稳定的速度发送，能容忍一些数据的丢失，但是不能允许有较大的时延（比如实时视频，直播等）
- 尽最大努力的交付，不保证可靠交付：所有维护传输可靠性的工作需要用户在应用层来完成。没有 TCP 的确认机制、重传机制。如果因为网络原因没有传送到对端，UDP 也不会给应用层返回错误信息
- 面向报文：发送方 UDP 对应用程序交下来的报文，在添加首部后就向下交付 IP 层。UDP 对应用层交下来的报文，既不合并，也不拆分，而是保留这些报文的边界。

## TCP

TCP 是运输层协议，其特点如下：

- 全双工数据：同一个连接上的双向数据流
- 面向连接：在数据交换前握手(交换控制信息) 初始化发送方和接收方的状态
- 流量控制：发送方不会淹没接收方

从输入 URL 到页面加载过程

1. 地址栏输入 URL
2. DNS 域名解析 IP
3. 建立 TCP 连接（3次握手）
4. 发送 HTTP 请求
5. 服务器处理请求
6. 返回 HTTP 响应结果
7. 关闭 TCP 连接（4次挥手）

## netstat 查看服务及监听端口

查看本机监听端口:

```
[root@VM-0-14-centos ~]# netstat -tlun
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN                          
udp        0      0 172.21.0.14:123         0.0.0.0:*                          
udp        0      0 127.0.0.1:123           0.0.0.0:*                          
udp6       0      0 fe80::5054:ff:fe2b::123 :::*                               
udp6       0      0 ::1:123                 :::*
```

查看本机路由表:

```
[root@VM-0-14-centos ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.21.0.1      0.0.0.0         UG        0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
172.21.0.0      0.0.0.0         255.255.240.0   U         0 0          0 eth0
```

列出所有的 TCP 端口:

```
[root@VM-0-14-centos ~]# netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 VM-0-14-cent:cslistener 0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:us-srv          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:https           0.0.0.0:*               LISTEN     
tcp        0      0 VM-0-14-centos:https    39.144.22.xx:50742       FIN_WAIT2  
tcp        0      0 VM-0-14-centos:https    59.71.243.xx:6994      ESTABLISHED
tcp        0      0 VM-0-14-centos:https    125.85.238.xx:7096      FIN_WAIT2  
tcp6       0      0 [::]:mysql              [::]:*                  LISTEN     
tcp6       0      0 [::]:distinct           [::]:*                  LISTEN     
tcp6       0      0 [::]:ddi-tcp-1          [::]:*                  LISTEN     
tcp6       1      0 172.21.0.14:ddi-tcp-1   140.82.115.xx:24923    CLOSE_WAIT 
tcp6       1      0 172.21.0.14:ddi-tcp-1   140.82.115.xx:64953    CLOSE_WAIT 
tcp6       1      0 172.21.0.14:ddi-tcp-1   140.82.115.xx:55955    CLOSE_WAIT
```

找出程序运行的端口:

```
[root@VM-0-14-centos ~]# netstat -ap | grep ssh 
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      1120/sshd           
tcp        0     64 VM-0-14-centos:ssh      117.82.229.xx:41408    ESTABLISHED 28968/sshd: xx@pt 
tcp        0      0 VM-0-14-centos:ssh      xxx.122.15.xx:32966     TIME_WAIT   -                   
tcp        0      0 VM-0-14-centos:ssh      203.xxx.85.146:43730    ESTABLISHED 687/sshd: xx [pri 
tcp        0      0 VM-0-14-centos:ssh      134.xx.xx.36:54474     TIME_WAIT   -                   
tcp        0      0 VM-0-14-centos:ssh      64.xx.111.127:45444    TIME_WAIT   -                   
unix  3      [ ]         STREAM     CONNECTED     16580    1120/sshd            
unix  2      [ ]         DGRAM                    113619201 28968/sshd: root@pt
```

## 参考资料

[Java全栈知识体系-网络基础(2) - 7层协议，4层，5层](https://pdai.tech/md/develop/protocol/dev-protocol-osi7.html)

## 进阶学习

- [《计算机网络：自顶向下方法》](https://book.douban.com/subject/36081529/)
- [小林coding-图解网络](https://xiaolincoding.com/network/)
