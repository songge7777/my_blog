# http
[[toc]]

## 应用层
- http
- DNS 协议: 将域名解析成ip地址(会缓存, 基于UDP)
- DHCP 协议: 自动获取网络配置(机器链接后 我们无需自己手动配置 动态配置IP, 基于UDP)
## 传输层
- TCP / UDP
## TCP
- TCP: 传输控制协议 在不可靠的 IP 层上建立可靠的传输层。 TCP提供全双工服务，即数据可在同一时间双向传播。
    -   一个字节8个bit
    -   数据格式
    -   (源端和目标的端口 16bit + 16bit)
    -   (32bit序列号)
    -   (32bit确认号)
    -   (首部长度和窗口大小等 32bit)
    -   (校验和 16bit 紧急指针 16bit)
    -   以及可选项和数据(默认情况没有),以上综合(20个字节),一个tcp至少有21个字节(只要一个)
## TCP优化策略
### 滑动窗口
- tcp 是全双工, 客户端和服务端 都有自己的缓存区域, 会根据网络调整发送数据的多少, 数据是乱序发送的,当接受到某个包,可能之前的包没有收到,此时需要等待前面序号的包到了才可以(队头堵塞)
- 如果某个数据丢包了  那需要重新发送(超时重传 RTO) 
- 当接收方的缓存区满了 每隔一段时间 发送方会 会发送一个探索包 来询问能否调整窗口大小，同时上层协议消耗掉了接收方的数据,接收方也会主动通知发送方调整窗口 继续发送数据
- 一帧 大 概1500 个字节: ip 20字节 tcp头 20字节  tcp剩下的 1460字节
- 问题: 每次去咨询窗口大小 都会发送一次请求 比较浪费 性能差

### 粘包
- nagle 算法 在同一时间内，最多只能有一个未被确认的小段 (TCP 内部控制) node 默认用它
  - 发送和确认的很快 nagle无法控制到
- cork算法(粘包) 当达到最大值时统一进行发送(此时就是帧的大小 - ip头  - tcp头 = 1460个字节) 理论值

### 拥塞处理
- 拥塞处理 快重传 快恢复(Reno算法)
  - 快重传: 可能在发送的过程中出现丢包现象,此时不要立即回退到慢阶段开始,而是对已接受到的报文进行重复确认,如果达到3次,则立即进行重复传, 快恢复算法(减少超时重传机制出现),降低cwnd的频率
  - 先从指数增长,达到一定的值,在线性增长
  - 恢复成原来的一半


###  建立连接/数据传输
- 连接 3次握手
- 断开 四次挥手
### UDP 
- UDP: 是一个无连接、不保证可靠性的传输层协议。你让我发什么就发什么！
- 使用场景： DHCP 协议、 DNS 协议、 QUIC 协议等 (处理速度快，可以丢包的情况)
- 数据结构:
    -   (源端和目标的端口 16bit + 16bit)
    -   数据
## 网络层
- IP 协议: 寻址通过路由器查找,将消息发给对方路由器
- ARP 协议: 地址转化协议(将ip地址转换成mac地址)

## HTTP/1.1
-  他基于 TCP/IP 协议；
### http 和 https
- https核心(保存密文,防止串改)
- http -> tcp -> ip -> mac
- https -> ssl/tls -> tcp -> ip -> mac
-  SSL 安全套接层（Secure Sockets Layer） 发展到 v3 时改名为 TLS 传输层安全(Transport Layer Security)，主要的目的是提供数据的完整性和保密性。
### tls 算法
- 摘要算法(hash算法 128位 2进制 生成一个签名) 主要保证数据完整 性
- 一段数据通过摘要算法生产一个签名,数据和签名一起发给服务器,服务通过对数据再次进行摘要算法,对比传递过来的数据 是否一直,如果一直则数据没有被修改
- 问题,中间人拦截数据 修改数据 生成一个新的算法给服务器,这样就有问题
### 对称加密(AES和ChaCha20)
- 客户端和服务端 都有一把共同的钥匙, 两边都用同一把钥匙进行加密解密
- 问题,中间人拦截那把钥匙
### 非对称钥匙
- 客户端有(公钥a和私钥a),服务端有(公钥b和私钥b)
- 服务端用公钥b加密给客户端,客户端用私钥a解密,同时客户端用公钥a加密给服务端,客户端用公钥a解密,
- 问题,中间人拦截没用,但是非常消耗性能
### 混合加密
- 对称+非对称
- 通过非对称加密处理密钥传输(服务器公钥给客户端,客户端用公钥和一个随机数生成一个东西给服务器,服务器通过私钥解析随机数)
- 数据传输就用对称(即 随机数作为对称加密的钥匙)
- 问题: 你不知道公钥是谁发给你的,没法校验公钥安全， 中间人拦截公钥(中间人攻击)
### 数字证书和ca (这些都是在握手阶段 后面长链接就不触发这些)
- 数字证书里面 有ca公钥和ca私钥
- 包含服务端公钥，证书所有者，证书颁发给谁，证书有效期，签名算法，摘要用, 用非对称加密,用私钥签名
- 当客户端访问服务端的时候，服务器会生成自己的公钥和私钥 =》将公钥传递给ca进行认证(ca证书有自己的私钥和公钥，他会把内容进行一个签名(直接把证书用私钥签名内容太多,把证书进行摘要,将摘要的结果用私钥加密,发送给客户端))
- 客户端会在操作系统中放入根证书，所以收到证书后可以进行验证签名(用内置的ca公钥进行解密，在对解密出内容进行摘要)，同时将传递的铭文进行摘要,这俩进行匹配如果一直则说明公钥是合法的 
- 可能问题, 中间人拿到服务器的公钥 也没关系,他没法解析客户端的发送的数据,只有服务器的私钥才能对数据解密
### 核心三步
- 加密，数据一致性，身份认证
- 百度使用的 RSA 算法生成密钥(2048长度,主要依靠长度来保证安全)
- 淘宝使用的 DH密钥交换(密钥协商:客户端和服务端各自手握一个密钥,通过混合,在传递给对方,拿到混合物在进行添加混合,此时两天的混合物是一样的,客户端混合物有服务端的密钥,服务端同样也是,但是密钥没有公开的传输过)
    -   密钥协商(DH 算法: client 和 server各自有生成一个参数混合后,相互传送,这就是serverParams和clientParams,然后将各自的保存参数和接受对方参数进行混合,保证他们两最终的数据是一样)
    -   客户端: 随机数C 随机数S + serverParams clientParams => 密钥会话
    -   服务端: 随机数C 随机数S + serverParams clientParams => 密钥会话

## http之前的故事
### HTTP/1.0
-  增加请求头和响应头,实现多类型数据传输。都是串行

## tcp的一些问题
- TCP顺序问题 后面的包先到达需要等待前面的包返回之后才可以继续传输(队头堵塞问题)
- 慢启动的过程 非常消耗性能(tcp 自带流量控制,他要找到一个合适的传输多少数据)
- time-wait 客户端链接服务器最后不会立即断开 在高并发 短链接的情况下啊 会出现端口全被占用(四次分手)

## http/1.1
- 1、基于tcp 传输层 半双工通讯(请求应答模式) http 默认是无状态(默认tcp 不能在没有应答完成后复用tcp 通道继续发送消息)
- 2、tcp的规范 就是固定的组成结构
  - 请求行  响应行  请求行 响应行 主要的目的就说是描述我要做什么事  服务端告诉客户端ok
  - 请求头  响应头  描述我们传输的数据内容 自定义我们的header (http中自己所做的规范)
  - 请求体  响应体  两者的数据
- 3、纯文本协议,安全问题 明文 => https
- 4、核心在于内容协商
- 5、http 实现长链接 会默认在请求的时候 增加 connection:keep-alive, connection:close 复用tcp通道传递数据(必须在一次请求应答后才能复用) (队头阻塞 http1.1中的)
- 6、管线化方式传递数据
  - 我们针对每个域名 分配6 个 tcp 通道(域名分片 域名不宜过多,过多会导致dns解析大量域名)
  - 问题在与 虽然请求是并发的但是 应答依然是按顺序 (管道的特点就是先发送的先回来) (队头阻塞 http1.1中的)

## 请求头和 响应头
- 客户端    服务端
- Accept   Content-Type  数据类型
- Accept-encoding  Content-Encoding  发的数据是用什么格式压缩(gzip,deflate,br)
- Accept-language   语言
- Range    Content-Range  范围请求数据


- 多个请求
 ## 
