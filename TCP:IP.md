# TCP/IP协议栈

![image-20200507192428520](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200507192428520.png)

![image-20200507193533337](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200507193533337.png)

![image-20200507193546324](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200507193546324.png)

![image-20200511102455080](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200511102455080.png)

![image-20200511102633072](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200511102633072.png)

![image-20200511111345022](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200511111345022.png)

- RTT：往返时间

## 物理层

属于TCP/IP的网络接口层。

- 连接不同的物理设备
- 传输比特流

### 物理

双绞线：相互缠绕的铜线

同轴电缆：内导体--绝缘层--外导体屏蔽层--绝缘保护套层

光纤：纤芯（高折射率）--包层（低折射率）

红外线：

无线：

激光：

### 信道

往一个方向传送信息的媒体。

往往包含一个接收信道和一个发送信道。（全双工）

#### 分用与复用

![image-20200511113107996](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200511113107996.png)

频分

时分

波分

码分

## 数据链路层

也位于TCP/IP的网络接口层。

- 封装成帧
- 透明传输
- 差错检测

### 封装成帧

- 帧是数据链路层数据的基本单位
- 发送端在网络层的数据前后添加特定标记成帧，帧首部00000001SOH，帧尾部00000100EOT
- 接收端根据标记识别出帧

### 透明传输

透明，对上层透明，上层不清楚细节。 

帧内存在EOT，增加转义字符，如果存在转义字符，前面再转义

### 差错检测

物理层只管传输，无法控制出错。

- 奇偶校验码：尾部添加一个bit位，判断1的数量，比如规定1的数量位偶数，需要补bit位凑成偶数1，有漏
- 循环冗余校验码CRC（现在用的）：根据传输或保存的数据而产生**固定位数校验码**的方法（一位或多位），检测数据传输或者保存后可能的出错，生成的数字计算出来并且**附加到数据后面**。
  -  ![image-20200512234836359](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512234836359.png)
  - ![image-20200512234855839](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512234855839.png)
  - 例子：![image-20200512235023291](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512235023291.png)
  - ![image-20200512235227845](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512235227845.png)
  - ![image-20200512235254360](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512235254360.png)
  - ![image-20200512235341863](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512235341863.png)
  - 错误检测能力与位串的阶数r有关，r=1退化为奇偶校验码
  - ![image-20200512235649881](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200512235649881.png)
- 数据链路层只进行检测，不进行纠正，如果出错就丢弃掉

### 最大传输单元MTU

#### MTU

Maximum Transmission Unit。数据链路层的数据帧的最大值。数据帧过大过大都会影响传输的效率。以太网MTU一般为1500。（时延：发送+排队+传播+处理）

#### 路径MTU

路径上最小的。木桶效应。![image-20200513000041081](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513000041081.png)

### 以太网协议

MAC地址（物理地址、硬件地址）：设备身份证，每一个设备都拥有唯一的MAC地址，48位，16进制表示。

广泛使用的局域网技术，用于数据链路层的协议，可以完成**相邻设备**的数据帧传输。

帧结构：

![image-20200513000807891](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513000807891.png)

MAC地址表：

![image-20200513000821362](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513000821362.png)

## 网络层-IP协议

IP大致分为：IP寻址、路由、IP分包与组包。

- IP协议使得复杂实际网络变为一个虚拟互联的网络
- IP协议使得网络层可以屏蔽底层细节而专注网络层的数据转发
- IP协议解决了在虚拟网络中数据报文传输路径的问题

### IP基础

#### MTU

不同的数据链路有它们各自的最大传输单位（MTU），在以太网中是1500字节，在FDDI中是4352字节，ATM是9180字节。IP的上一层可能会要求传送更大的字节的数据，因此必须在线路上传送比包长还要小的MTU。

引入IP分片（IP Fragmentation），即将较大的IP包分成多个较小的IP包。

#### IP无连接

不需要建立连接：

- 一是简化
- 二是提速

为了提高可靠性，上一层的TCP采用了有连接。

#### IP地址基础

IP地址由32bit表示，以8位为一组，分成4组，每组以“.”隔开，再将每组数专为十进制数。

##### IP地址组成

IP地址由“网络标识（网络地址）”和“主机标识（主机地址）”组成。

保证相同数据链路段内有相同的网络地址，不同数据链路段由不同的网络地址。

保证主机地址在一个网段内不重复。

路由时正是更具目标IP地址的网络地址进行路由。

网络地址开始由地址分类进行区别，现在基本以子网掩码（网络前缀）区分。

##### IP地址分类

- A类地址：首位以0开头，从第1位到第8位是它的网络地址。即0.0.0.0~127.0.0.0是A类网络地址，后24位为主机地址；
- B类地址：前两位以10开头，从第1位到第16位是它的网络地址，即128.0.0.1~191.255.0.0，后16位为主机地址；

- C类地址：前三位以110开头，从第1位到第24位是它的网络地址，即192.168.0.0~239.255.255.0，后8位相当于主机地址；
- D类地址：前四位以1110开头，从第1位到第32位是它的网络地址，即224.0.0.0~239.255.255.255，没有主机地址，常用于多播。

注意：主机地址不能为全0和全1，用于地址不可知的情况和广播地址（主机地址全1）。

##### IP广播

本地广播：本网络内的广播，即网络地址为本网络，主机地址全1；

直接广播：不同网络之间的广播，网络地址为其他网络，主机地址全1。

##### IP多播

同时发送提高效率：用于将包发送给特定组内的所有主机，直接使用IP协议，也不存在可靠传输。

- 使用单播：采用复制1对1通信的方式，实现多播；
- 使用广播，则广播会发送给所有终端，再由IP之上的一层区判断是否有必要接收并处理数据，会给无关主机带来影响，造成网络上很多不必要的流量。而且由于广播无法穿透路由，若向给其他网段发送同样的包，就得采用其他的机制。
- 使用多播，可以穿透路由器，且只发送给组内的节点（IP层过滤）

**多播地址**

多播使用D类地址，首位为1110即为多播地址，剩余的28位可以成为多播的组编号。

224.0.0.0~239.255.255.255都是多播地址的可用范围。其中224.0.0.0~224.0.0.255的范围不需要路由控制，在同一个链路内也能实现多播。而在这个范围之外设置多播地址会给全网所有组内成员发送多播的包。

对于多播，所有的主机（路由器除外）必须属于224.0.0.1的组，所有路由器必须属于224.0.0.2的组。

![image-20200509011749964](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509011749964.png)

##### 子网掩码

分类造成浪费：因此用1表示网络地址的部分，用0表示主机地址的部分。

CIDR，无类域间路由，采用任意长度分割IP地址的网络标识；VLSM，可变长子网掩码，不同部门之间不同的掩码

##### 全局地址与私有地址

IP不足，无法为每台主机分配一个全局地址；因此只在必要的时候只为相应数量的设备分配唯一的全局IP地址；而这些设备使用私有地址。

- A类：10.0.0.0~10.255.255.255(/8)
- B类：172.16.0.0~172.31.255.255(/12)
- C类：192.168.0.0~192.168.255.255(/16)

私有地址的设备联网时，通过NAT通信。私有地址只要保证在同一个域内唯一即可。因此现在主流方式是私有地址+NAT来解决IP地址分配的问题。

### 路由控制

静态路由控制：手动配置

动态路由控制：路由器交互信息后自动刷新。

### IP协议转发流程

![image-20200513113013644](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513113013644.png)



### IP分片与重组

![image-20200509013056090](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013056090.png)

### IPv6

根本解决IPv4地址耗尽问题，同时弥补IPv4中的绝大多数缺陷。

![image-20200509013401471](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013401471.png)

### IPv4首部

![image-20200509013536266](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013536266.png)

#### 版本

![image-20200509013600367](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013600367.png)

#### 首部长度

4比特，表明IP首部的大小，单位为4字节。对于没有选项的头部设置为5（20字节）。

#### 区分服务

8比特，表明服务质量。

![image-20200509013738921](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013738921.png)

几乎不用。

#### DSCP段与ECN段

![image-20200509013836071](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509013836071.png)

#### 总长度

表示IP首部与数据部分结合起来的总字节数，长16bit，因此IP包的最大长度为65535字节。

#### 标识

16比特，用于分片重组。同一个分片的标识值相同，不同分片的标识值不同。通常，每发送一个IP包，它的值也递增。

#### 标志

3比特，表示包被分片的相关信息。![image-20200509014045841](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509014045841.png)

#### 片偏移

13比特，标识被分片的每一个分段相对于原始数据的位置。第一个分片对应值为0，可以最多表示8192个相对位置。单位为8字节，因此最大可以表示8*8192=65535个字节的位置。

#### 生存时间

8比特，指可以中转多少个路由器的意思，每经过一个路由器，TTL值为减少1，直到变成0则丢弃该包。

#### 协议

8比特，表示IP首部的下一个首部属于哪个协议。

![image-20200509014342543](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509014342543.png)

![image-20200509014351920](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509014351920.png)

#### 首部校验和

16比特，也叫IP首部校验和，只校验数据报的首部，部校验数据部分。主要用来确保IP数据报不被破坏。

计算过程：首先将该校验和的所有位置设置为0，然后以16比特为单位划分IP首部，并用1补数计算所有16位字的和，最后将所得到的这个和的1补数赋给首部校验和字段。

#### 源地址

32比特，发送端IP地址

#### 目的地址

32比特，接收端IP地址

#### 可选项

长度可变，通常在实验或诊断时使用。

#### 填充

在由可选项的情况下，首部长度需要调整为32比特的整数倍。

#### 数据

存入数据，将IP层之上的协议首部也作为数据进行处理。

## IP协议相关技术

### DNS

IP地址不便记忆。产生了管理主机名和IP地址之间对应关系的系统，即DNS系统，它维护一个表示组织内部主机名和IP地址之间对应关系的数据库。

#### 域名构成

分层结构为树形结构。

![image-20200509103144066](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509103144066.png)

#### 域名服务器

指管理域名的主机和相应的软件，它可以管理所在分层的域的相关信息。其所管理的分层叫做ZONE。

#### DNS查询

。。。

### ARP（Address Resolution Protocol）

将IP地址解析为MAC地址。

借助ARP请求（广播要了解MAC地址的IP地址）和ARP响应（MAC地址）来完成。

![image-20200509103801523](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509103801523.png)

![image-20200513113854412](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513113854412.png)

#### ARP缓存表

![image-20200513113542223](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513113542223.png)

### RARP(Reverse)

从MAC地址定位IP地址的一种协议。

![image-20200509104111490](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509104111490.png)

### ICMP

Internet Control Message Control

作为IP的辅助：确认IP包是否成功送达目的地址，通知在发送过程中IP包被废弃的具体原因，改善网络设置等。

![image-20200513115924136](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513115924136.png)

#### 主要的ICMP消息

差错报告：

![image-20200513115955749](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513115955749.png)

询问：

![image-20200513120135606](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513120135606.png)

##### ICMP目标不可达（类型3）

IP路由器无法将IP数据报发送给目标地址时，会给发送端主机返回一个目标不可达的ICMp消息，并显示具体原因。

![image-20200509104405059](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509104405059.png)

##### ICMP重定向消息（类型5）

如果路由器发现发送端主机使用了次优的路径发送数据，那么它会返回一个ICMP重定向消息给这个主机，消息中包含了最合适的路由信息和源数据。发生在路由器持有更好的路由信息的情况下。

##### ICMP超时消息（类型11）

IP包中字段TTL，减到0时包被丢弃，同时路由器将会发送一个ICMP超时消息给发送端主机，并通知该包已被丢弃。（为了避免循环，导致IP包无休止地在网络上被转发。同时TTL可以控制包的到达范围）

##### ICMP回送消息（类型0、8）

用于进行通信的主机或路由器之间，判断所发送的数据包是否已经成功到达对端的一种消息。可以向对端主机发送回送请求的消息（ICMP Echo Request Message，类型8），也可以接受对端主机发送回来的回送应答消息（ICMP Echo Reply Message，类型0）。ping命令就是用回送消息实现的。

#### 其他ICMP消息

##### ICMP原点抑制消息（类型4）

网络拥堵时，发送原点抑制，收到的节点了解了某一处发生拥堵，从而打开IP包的传输间隔。

##### ICMP路由探索消息（类型9、10）

主要用于发送与自己相连网络中的路由器。当一台主机发出ICMP路由器请求（类型10）时，路由器则返回ICMP路由器公告消息（类型9）给主机。

##### ICMP地址掩码消息（类型17、18）

主要用于主机或路由器想要了解子网掩码的情况，可以向那些目标主机或路由器发送ICMP地址掩码请求消息（17），然后通过接收ICMP地址掩码应答消息（18）获取子网掩码的信息。

#### ICMP应用

ping命令：0、8

traceroute：探测数据报走过的路径，通过ICMP终点不可达错误以及设置Ip报TTL

### DHCP

实现自动设置IP地址、统一管理IP地址分配，产生了DHCP（Dynamic Host Configuration Protocol）协议，让即插即用变为可能。

#### 工作机制

需要一台DHCP服务器，然后将DHCP所要分配的IP地址设置到服务器上，包括相应的子网掩码、路由控制信息以及DNS服务器的地址等设置到服务器上。

![image-20200509105533257](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509105533257.png)

### NAT

用于在本地网络中使用私有地址，在连接互联网时转而使用全局IP地址的技术。此外，还出现了可以转换TCP、UDP端口号的NAPT技术，实现用一个全局IP地址与多个主机的通信。

多台主机都要通信，需要NAPT。

![image-20200509105921341](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509105921341.png)

#### 潜在问题

- 无法从NAT的外部向内部服务器建立连接。
- 转换表的生成与转换操作都会产生一定的开销；
- 通信过程中一旦NAT遇到异常重启，所有的TCP连接都将被重置，即使两台NAT做容灾备份，TCP连接还是会断开

#### 解决

- 改用IPv6，每台设备都可以配置一个全局IP地址；
- 内网穿透，为了生成转换表，需要在内网主机上首先发送一个虚拟的网络包给NAT的外侧，而NAT并不知道这个虚拟的包究竟是什么，还是会照样读取包首部中的内容，并生成一个转换表，这样就能实现NAT外侧的主机与内侧的主机建立连接进行通信。

### IP隧道

![image-20200509111108384](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509111108384.png)

## 传输层-TCP与UDP

- TCP：面向连接的、可靠的流协议，流即不间断的数据结构。在TCP中虽然可以保证发送的顺序，但是还是犹如没有任何间隔的数据流发送给接收端。TCP提供可靠性传输，实行“顺序控制”或“重发控制”机制。此外还有流量控制、拥塞控制、提高网络利用率等众多功能。
- UDP：UDP是不具有可靠性的数据报协议，可以确保发送消息的大小，却不能保证消息一定到达，因此，应用有时会根据自己的需要进行重发处理。

**区别**

TCP用于在传输层有必要实现可靠传输的情况，由于它面向连接、具备顺序控制、重发控制等机制，所以它可以慰应用提供可靠传输。

UDP主要用于哪些对高速传输和实时性有较高要求的通信或广播通信。

### 端口号

用于识别同一台计算机中进行通信的不同应用程序，也被称为程序地址。0～65536（16位）标记不同进程。

![image-20200509112320577](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509112320577.png)

#### 通过IP地址、端口号、协议号进行通信识别

通常采用5个信息来识别一个通信：

- 源IP地址
- 目的IP地址
- 协议号
- 源端口号
- 目的端口号

只要其中某一项不同，则被认为是不同的通信。

#### 端口号的确定

- 标准既定的端口号（静态方法）：指每个应用程序都有其指定的端口号，也被称为知名端口号（Well-Known Port Number)，一般由0～1023的数字分配而成，应用程序应该避免使用致命端口号进行既定目的之外的通信，以免冲突。
- 时序分配法（动态分配法）：服务端有必要确定监听端口号，接受服务器的客户端没有必要确定端口号。客户端应用程序可以完全不用自己设置端口号，而全权交给操作系统进行分配，操作系统可以为每个应用程序分配互不冲突的端口号。动态分配的端口取值范围在49152到65535之间。

#### 端口号与协议

不同传输层协议可以使用相同的端口号，端口号的处理是根据每个传输协议的不同而进行的。

数据到达IP层之后，会先检查IP首部中的协议号，再传给相应的协议模块。

##### TCP端口

![image-20200509130855144](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509130855144.png)

![image-20200509130906895](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509130906895.png)

##### UDP端口

![image-20200509130943223](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509130943223.png)![image-20200509130951548](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509130951548.png)

### UDP

User Datagram Protocol的缩写。数据报，不合并不拆分，直接封包。

- 无连接协议
- 不保证可靠（想发就发，不保证不丢失）
- 面向报文传输
- 没有拥塞控制
- UDP首部开销很小

非常简单的协议，UDP不提供复杂的控制机制，利用IP提供面向无连接的通信服务。场景：

- 包总量较少的通信（DNS，SNMP等）
- 视频、音频等多媒体通信（即使通信）
- 限定于LAN等特定网络中的应用通信
- 广播通信（广播、多播）

#### UDP的首部

![image-20200509135342515](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509135342515.png)

![image-20200509135402785](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509135402785.png)

UDP也有可能不用校验和，此时校验和字段填入0，此时不进行校验和计算，协议处理开销降低，提高数据转发速度。

![image-20200509135712177](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509135712177.png)

### TCP

Transimission Control Protocol：传输控制协议

TCP：传输、发送、通信进行控制的协议。

- 面向连接的协议
- TCP的一个连接有两端（点对点通信）
- 提供可靠的传输服务
- 提供全双工的通信
- 面向字节流的协议

通过校验和、序列号、确认应答、重发控制、连接管理及窗口控制等机制实现可靠性传输。

#### 可靠传输基本原理

##### 停等协议（停止等待协议）

![image-20200514181841364](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514181841364.png)

停止生成新消息，等待确认消息。确认消息超时，重传消息。（发送消息丢失，确认消息丢失，确认消息延迟）

##### 超时定时器！！

需要设置超时定时器，每发送一个消息，都需要设置一个定时器。

最简单的可靠传输协议。对信道利用效率不高。

##### 连续ARQ协议

Automatic Repeat request：自动重传请求

![image-20200514182311050](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514182311050.png)

**滑动窗口**，收到确认就推动窗口，数据批量发送。

**累计确认**，同时发送1-6，若收到5的确认消息，则认为1-5的消息都发送成功了：

![image-20200514182430316](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514182430316.png)

减少确认的数量，提高效率。TCP基于此。

#### 通过序列号与确认应答提高可靠性

数据到达接收端，接收端主机会返回一个已收到消息的通知，即确认应答ACK。

确认应答处理、重发控制、重复控制等功能都通过序列号实现。通过序列号和确认应答号，TCP实现可靠传输。

#### 重发超时的确定

重发超时指在重发数据之前，等待确认应答到来的那个特定时间间隔，如果超过了这个时间没收到确认应答，发送端重传。

最理想，找到一个最小时间，保证确认应答一定能在这个时间内返回。但难确定。

最初重发超时设置为6秒左右。重发之后，等待应答时间将以2倍、4倍的指数函数延长。重发到一定次数之后，就会判断为网络或对端主机出现了异常，强制关闭连接，并通知应用通信异常强行终止。

#### 连接管理

3+4次握手。

##### 三次握手

![image-20200514223801823](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514223801823.png)

##### 为什么要三次？

1. 失效的请求报文再次到达![image-20200514224419487](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514224419487.png)
2. 2次无法确保双向连接，无法确保双方成功交换了序列号。四次没必要，浪费资源

##### 四次挥手

![image-20200514224800214](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514224800214.png)

##### 等待计时器！！（TIME-WAIT）

等待2MSL，Max Segment Lifetime（最长报文段寿命）

MSL建议设置为2分钟。

- 防止第四次ACK丢失，保证第四次挥手正确到达接收方。
- 2MSL时间内没有收到第四次ACK，接收方回重发第三次挥手报文
- 另外，确保当前连接的所有报文都已过期？





#### TCP以段为单位发送数据

在建立连接的同时，也可以确定发送数据报的单位，最大消息长度MSS，最理想的情况，最大消息长度正好是IP中不会被分片处理的最大数据长度。

TCP在传输大量数据时，是以MSS的大小将数据进行分割发送，进行重发时也是以MSS为单位。

MSS时在三次握手的时候，在两端主机之间被计算得出。两端主机在发出建立连接的请求时，会在TCP首部写入MSS选项，告诉对方自己接口能够适应的MSS的大小，然后会在两者之间选择一个较小的值投入使用。

#### 利用窗口控制提高速度

TCP以1个段为单位，每发一个段进行一次确认应答的处理。

![image-20200509133204441](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509133204441.png)

缺点：包的往返时间越长，通信性能就越低。

因此TCP引入窗口的概念，近视在往返时间较长的情况下，它也能控制网络性能的下降。确认应答不再是以每个分段，而是更大的单位，转发时间将会被大幅度的缩短。也就是说，发送端主机在发送了一个段以后不必要一直等待确认应答，而是继续发送。（累计确认）。

![image-20200509133352745](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509133352745.png)

窗口大小就是指无需等待确认应答而可以继续发送数据的最大值。即使用了大量的缓冲区，通过对多个段同时进行确认应答的功能。

#### 窗口控制与重发控制

快重传。

#### 流量控制

让发送方发送速率不要太快，利用滑动窗口来实现。

![image-20200514182840295](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514182840295.png)



接收端主机向发送端主机通知自己可以接受数据的大小，发送端的发送窗口设置小于这个大小。TCP首部有专门字段通知窗口大小，窗口指明允许对方发送的数据量。

同时为了通信得以继续（避免窗口更新通知丢失），在窗口为0 时会时不时得发送窗口探测的数据段，仅包含一个字节，以获得最新的窗口大小。

##### 坚持定时器！！解决死锁

- 当收到接口窗口位0的消息，启动坚持定时器
- 坚持定时器每隔一段时间发送一个窗口探测报文

#### 拥塞控制

拥塞的根源：

- 一条数据链路经过多个设备
- 数据链路中各个部分都有可能称为网络传输的瓶颈

与流控区别：

- 流控考虑点对点的通信量的控制
- 拥塞控制考虑整个网络，是全局性的考虑，难有最优解

判断方法：

- 报文超时则认为是拥塞（一种方法）

在通信刚开始就发送大量数据，也可能引发其他问题。

一般，计算机网络都处在一个共享的环境，因此也有可能会因为其他主机之间的通信使得网络拥堵，网络拥堵时，如果突然发送一个较大量的数据，极有可能会导致整个网络的瘫痪。

##### 慢启动算法

- 由小到达逐渐增加发送数据（拥塞窗口）
- 每收到一个报文确认，拥塞窗口就加一，指数增长

TCP为了防止该问题，在通信一开始通过**慢启动**算法，对发送数据量进行控制。

![image-20200509134157142](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509134157142.png)

- 定义拥塞窗口，初始1MSS；
- 每次收到一次ACK，拥塞窗口的值就加1；
- 发送窗口为拥塞窗口和接收窗口的较小值。

##### 拥塞避免算法

- 维护一个拥塞窗口的变量
- 只要网络不拥塞，就试探着调大拥塞窗口，每次加一

不过，拥塞窗口也会以1、2、4等指数增长，拥堵状况激增甚至导致网络拥塞的发生。为了防止这些，引入了慢启动阈值，超出阈值，启用**拥塞避免**算法。

**TCP通信开始时，并没有设置相应的慢启动阈值**，而是在**超时重发**时，将慢启动阈值设置为当时拥塞窗口的一半的大小。

而在三次重复确认后，有一些不同，因为此时网络拥堵要轻一些，慢启动阈值大小被设置为当时窗口大小的一半，然后将窗口的大小设置为该慢启动阈值+3个数据段的大小。

##### 快重传

三次重复确认，启动快重传

##### 快恢复

直接启动拥塞避免，因为不拥塞

#### 提高网络利用率的规范

如果每个小报文都直接发送的话，头部占去了大量的带宽，网络利用率很低，因此引入了Nagle算法、延迟确认应答机制。

##### Nagle算法

指即使发送端还有应该发送的数据，但是如果这部分数据很少的话，则进行延迟发送的一种处理机制，即仅仅在下列任意一种条件下才能发送数据，否则等待一段时间：

- 已发送的数据都已经收到确认应答时
- 可以发送最大段长度MSS的数据时

可能发生某种程度的延迟。

##### 延迟确认应答

接收数据的主机如果每次都立刻回复确认应答的话，可能返回一个较小的窗口，那是因为刚接收完数据，缓冲区已满。因此引入延迟应答。

- 在没有收到2xMSS为止不做应答确认
- 其他情况下，最大延迟0.5秒就发送确认应答（很多操作系统设置为0.2秒)

##### 捎带应答

必须启用延迟确认应答，才能实现捎带应答。

#### TCP的应用

。。可靠

#### TCP首部

![image-20200509135732196](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509135732196.png)

没有表示包长度和数据长度的字段，可又Ip层获知TCP包长，由TCP的包头长可以获知数据的长度。

序列号：数据的第一个字节的序号

确认号：期待收到数据的首字节号

数据偏移：4bit，单位4字节，表示数据偏移首部的距离，同时也是头部长度

TCP标记：

![image-20200509135956060](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509135956060.png)



![image-20200509140003602](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509140003602.png)![image-20200509140009571](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509140009571.png)![image-20200509140022703](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509140022703.png)![image-20200509140029869](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509140029869.png)

紧急指针：URG置位时起作用，指明紧急数据在报文中的位置

选项：协议拓展

### 套接字

![image-20200514225326894](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514225326894.png)

- 套接字（Socket）抽象概念，表示TCP连接的一端
- 通过套接字可以进行数据发送或接收
- TCP={Socket1:Socket2}={{IP:PORT}{IP:PORT}}

![image-20200514225538242](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514225538242.png)

#### 网络套接字与域套接字

![image-20200514230109024](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514230109024.png)



## 路由协议

![image-20200513122017632](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513122017632.png)

### RIP协议

#### 距离矢量（DV）算法

- 每个节点使用两个向量Di和Si
  - Di描述当前节点到别的节点的距离
  - Si描述当前节点到别的节点的下一节点
- 每个节点与相邻节点交换Di和Si
- 交换后更新自己的信息

![image-20200513122333058](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513122333058.png)

![image-20200513122531161](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513122531161.png)

#### RIP协议的过程

Routing Information Protocol

- 把跳数作为DV算法的距离
- RIP协议每30s交换一次路由信息
- RIP协议认为跳数>15不可达

过程：

- 路由器初始化路由信息（两个向量）
- 对相邻路由器X发过来的信息，对信息的对荣进行据该（下一跳地址设为X，所有距离+1）
  - 检索本地路由，将信息中新的路由插入到路由表里面
  - 检索本地路由，对于下一跳为X的，更新为修改后的信息
  - 检索本地路由，对比相同目的的距离，如果心信息的距离更小，则更新本地路由表
- 如果3分钟没有收到相邻的路由信息，则把相邻路由设置为不可达（16跳）

#### Dijkstra算法

广度优先搜索，解决有权图从一节点到其他节点的最短路径的问题。“以起始点为中心，向外层层扩展”

![image-20200513153934667](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200513153934667.png)

### OSPF协议

#### 基本过程

1. 向所有的路由器发送消息
   1. 获取网络中的所有消息->完整网络拓扑（链路状态数据库，全网一致）
   2. Dijkstra算法
2. 消息描述该路由器与相邻路由器的链路状态（貌似是用带宽计算的）
3. 只有链路状态发生变化时，才发送更新消息

#### 五种消息类型

- 问候消息（Hello）：维护与相邻路由器的可达性
- 链路状态数据库描述信息：向邻居发送自己的描述
- 链路状态请求消息：向邻居请求链路状态数据库
- 链路状态更新信息：
- 链路状态确认消息：对链路状态更新的确认

#### 协议过程

- 路由器接入网络
- 路由器向另据发出问候信息
- 与邻居交换链路状态数据库
- 广播和更新未知路由

#### 与RIP区别

- RIP从邻居看网络，OSPF有整个网络的拓扑
- RIP在路由器之间累加距离，OSPF使用Dijk算法计算最短路径
- RIP频繁、周期更新，收敛很慢，OSPF状态变化更新，收敛很快
- RIP路由间拷贝路由消息，OSPF路由间传递链路状态，自行计算路径。

### BGP协议（外部网络路由协议）

运行在AS之间的一种协议。

#### 为什么要外部网关路由

- 互联网的规模很大
- AS内部使用不同的路由协议
- AS之间考虑网络特性之外的一些因素（政治、安全）

#### 基本介绍

BGP协议能够找到一条到达目的比较好的路由，并不一定是最好的。

BGP发言人（speaker），路由器![image-20200514172930494](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514172930494.png)

- BGP不关心内部网络拓扑
- AS之间通过BGP发言人交流信息
- BGP Speaker可以人为配置一些策略

## 应用层-应用协议

应用层是面向用户的一层。

定义应用间通信的规则。

- 应用进程的报文类型
- 报文的语法、格式
- 应用进程发送数据的时机、规则

![image-20200514230312319](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514230312319.png)

![image-20200514230428993](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514230428993.png)

### DNS

Domain Name System，域名系统，域->网络/AS，名->IP

- 使用域名帮助记忆
- DNS服务将域名转换为IP地址
- 使用IP地址在网络中传输

#### 域名

- 点、字母、数字组成
- 点分割不同的域
- 域名可以分为顶级域(com)、二级域(taobao)、三级域(www)

##### 顶级域

![image-20200514231014222](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514231014222.png)

##### 二级域

![image-20200514231058743](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514231058743.png)

##### 结构

![image-20200514231132192](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514231132192.png)

##### 域名服务结构

![image-20200514231330765](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200514231330765.png)

##### 例子

访问本地->根->顶级->二级...

递归+迭代

### DHCP-即插即用联网

Dynamic Host Configuration Protocol，动态主机设计协议，是一个局域网协议，使用UDP

- 分配临时IP
- 租期，续租

#### DHCP过程

- DHCP监听67
- 主机使用UDP协议广播DHCP发现报文，源IP全1
- DHCP服务器发出DHCP提供报文
- 主机想服务器发出DHCP请求报文
- DHCP服务器回应并提供IP地址

### 远程登录

主要由TELNET和SSH两种协议。

#### TELNET

利用TCP一条连接，通过该连接向主机发送文字命令并在主机上执行。本地用户好像直接与远端主机内部的Shell相连着似的，直接在本地进行操作。

TELNET可以分为两类基本服务，一是仿真终端功能，二是协商选项机制。

![image-20200509141138512](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509141138512.png)

TELNET经常用于登录路由器或高性能交换机等网络设备进行相应的设置。通过TELNET登录主机或路由器时需要将自己的登录用户和密码注册到服务端。

![image-20200509141353504](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509141353504.png)

#### SSH

加密的远程登录系统。TELNET中登录时无需输入密码就可以发送，容易造成通信窃听和非法入侵的危险，使用SSH可以加密通信的内容，即使信息被窃听也无法破解所发送的密码、具体指令以及命令返回的结果是什么。

![image-20200509141630650](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509141630650.png)

### 文件传输

FTP是两个相连的计算机之间进行文件传输时使用的协议。

有一种FTP服务器是允许任何人进行访问的，这种服务器叫做匿名服务器。登录这些服务器时使用匿名（anonymous）或ftp都可以。

#### 工作机制

两条TCP连接：一条控制连接，一条数据连接。

用于控制的TCP连接主要在FTP的控制部分使用。例如登录用户名和密码的验证、发送文件的名称、发送方式的色值设置。利用这个连接，可以通过ASCII码字符串发送请求和接收应答。在这个连接上无法发送数据，数据需要一个专门的TCP进行连接。TCP21号端口，进行文件GET、PUT、LIST等操作时，每次都会建立一个用于数据传输的TCP连接，数据的传输和文件一览表的传输正式在这个新建的连接上进行。当数据传送完毕之后，传输数据的这条连接也会被断开，然后在控制用的连接上继续用命令或应答处理。

用于数据传输的TCP连接通常是按照与控制用的连接相反的方向建立的。因此，在通过NAT连接外部FTP服务器的时候，无法直接建立传输数据时使用的TCP连接。此时，必须使用PASV命令修改连接的方向才行。

控制用的连接，在用户要求断开之前会一直保持连接状态，不过，绝大多数FTP服务器都会对长时间没有任何新命令输入的用户的连接强制断开。

数据传输用的TCP连接通常使用端口20，不过可以用PORT命令修改为其他的值。最近，处于安全考虑，普遍在数据传输时用的端口号使用随机数进行分配。

![image-20200509142428853](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509142428853.png)

### 电子邮件

使用TCP协议。

![image-20200509142622441](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509142622441.png)

![image-20200509142637611](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509142637611.png)

![image-20200509142647904](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509142647904.png)

#### 邮件地址

名称@通信地址

发送地址由DNS管理。

#### MIME

之前只能处理文本格式的邮件，现在数据类型扩展到了MIME。可以发送静态图像、动画、声音、程序等各种形式的数据。相当于第6层表示层。

由首部和正文两部分组成。首部不能出现空行，

![image-20200509143023385](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143023385.png)

![image-20200509143030499](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143030499.png)

#### SMTP

发送电子邮件的协议，使用TCP25号端口。SMTP建立一个TCP连接后，在这个连接上进行控制和应答以及数据的发送。客户端已文本的形式发出请求，服务端返回一个三位数字的应答。

![image-20200509143225690](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143225690.png)![image-20200509143245602](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143245602.png)

![image-20200509143307520](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143307520.png)![image-20200509143314237](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143314237.png)

#### POP

接收电子邮件的协议，发送端发到POP服务器，客户端再根据POP协议从POP服务器接收对方发来的邮件。端口默认TCP110。

![image-20200509143402541](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143402541.png)

#### IMAP

类似于POP，接收电子邮件的协议，在POP中邮件由各户端进行管理，而IMAP中邮件则由服务器进行管理。

使用IMAP时，可以不必从服务器上下载所有的邮件也可以阅读。

### WWW

三个重要概念：

- 访问信息的手段和位置：URI
- 信息的表现形式：HTML
- 信息转发：HTTP

#### URI

标识资源

![image-20200509143714008](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509143714008.png)

这些例子属于一般主页地址，也被叫做URL，URL常被人们用来表示互联网中资源（文件）的具体位置。但是URI不局限于标示互联网资源，它可以作为所有资源的识别码。

#### HTML

记述Web页面的一种语言（数据格式）。可以指定浏览器中显示的文字、文件大小、颜色动画、音频等

#### HTTP协议

HyperTest Transfer Protocol，超文本传输协议，可靠的数据传输协议，基于UDP

地址：http(s):///<主机（ip或域名）>:<端口80/443>/<路径>

80端口，首先客户端向服务器端80端口建立一个TCP连接，然后在该连接上进行请求和应答以及数据报文的发送。

![image-20200509144039597](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144039597.png)

![image-20200509144141513](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144141513.png)

![image-20200515001628555](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515001628555.png)

![image-20200509144149266](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144149266.png)

![image-20200509144156438](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144156438.png)![image-20200509144203407](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144203407.png)

##### HTTP/1.0

每一个命令和应答都会触发一次TCP连接的建立和断开。

##### HTTP/1.1

允许在一个TCP连接上发送多个命令和应答，长连接。提高效率。

#### HTTPS

HTTP是明文传输的，HTTPS是安全的HTTP

##### 数字证书

可信任组织颁发给特定对象的认证。

![image-20200515002150636](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515002150636.png)

##### SSL

Secure Sockets Layer，安全套接层

![image-20200515002225975](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515002225975.png)

##### HTTPS工作过程

![image-20200515002349213](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515002349213.png)

![image-20200515002531815](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515002531815.png)

![image-20200515002621043](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200515002621043.png)



### Js、CGI、Cookie

Js，验证输入，操作HTML逻辑结构、动态显示Web页面的内容和页面风格。

![image-20200509144350477](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144350477.png)![image-20200509144358807](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144358807.png)

#### CGI

CGI时Web服务器调用外部程序时所使用的一种服务端应用的规范。

一般的Web，请求静态内容。

引入CGI以后，客户端请求会触发Web服务器端运行另一个程序，客户端所输入的数据也会传给这个外部程序，该程序运行结束后会将生成的HTML和其他数据再返回给客户端。（动态）

#### Cookie

Cookie在客户端保存信息，常被用于保存登录信息或网络购物中放入购物车的商品信息。

从Web服务器检查Cookie可以确认是否为同一对端的通信，从而存放于购物车的商品信息就不必要再保存到服务器了。

### 网管协议

#### SNMP

## 网络安全

### 防火墙

### IDS入侵检测系统

#### 反病毒/个人防火墙

### PKI公钥基础结构

![image-20200509144856566](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509144856566.png)

### 身份认证技术

### 安全协议

#### IPSec与VPN

使用VPN构造虚拟专用链路，最常使用的是IPSec。是指在IP首部的后面追加“封装安全有效载荷”和“认证首部”，从而对此后的数据进行加密，不被盗。

![image-20200509145232532](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509145232532.png)

#### TLS/SSL与HTTPS

![image-20200509145711540](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200509145711540.png)