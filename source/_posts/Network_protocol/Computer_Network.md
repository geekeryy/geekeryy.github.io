---
title: 计算机网络
date: 2025-02-27 00:17:00
categories:
    - [网络协议,计算机网络]
tags: 
    - 网络协议
---

![画板](/img/network/network_mind.jpeg)

### 网络层次划分

#### OSI七层参考网络模型

1. 应用层：产生数据
2. 表示层：对应用层来的数据进行压缩，解压缩，加密，解密（翻译官）
3. 会话层：数据传输之前建立一个会话，传输过程中维持一个会话，传输结束终止这个会话
4. 传输层：标明上层是哪些应用程序（流控）（segment）
    1. 源端口号2字节、目的端口号2字节、SYN和ACK各1字节
5. 网络层：寻址 （packet）
    1. 源IP地址4字节、目的IP地址4字节、protocol字段1字节（标明上层所使用的协议）
    2. 1（ICMP）、6（TCP）、17（UDP）、88（EIGRP）、89（OSPF）
6. 数据链路层：起到一个承上启下的作用 （frame）
    1. 源MAC地址6字节、目的MAC地址6字节、type字段2字节（标明上层使用的协议）
    2. 0x0800（表示上层是IPV4）、0x0806（表示上层是ARP）、0x86dd（表示上层是IPV6）
7. 物理层：定义一些设备的接口以及传输速率

#### TCP/IP五层网络模型

1. 应用层：为操作系统或应用程序提供访问应用的接口，基本单位为报文
2. 传输层：提供端对端的数据传输（网关）
3. 网络层：路由选择，基本单位为IP数据报（路由器）
4. 数据链路层：为网络层提供可靠的数据传输，基本单位为帧，主要协议为以太网协议（网桥、交换机）
5. 物理层：确保原始数据可在各种物理媒体上传输（中继器、集线器）

### 网络设备

#### 交换机

维护MAC地址表，隔离冲突域、寻址和进行数据转发

交换机 会把广播、组播和未知单播帧从所有其他端口发送出去（除了发送帧的端口）

> 交换机智能的原因：

它可以学习以太网数据帧中的源MAC地址，交换机可以将学习到的MAC地址，记录到MAC地址表里面，MAC地址表里面会记录MAC地址是从哪个接口发来的。

> 交换机三种处理数据帧的方式： 

1. 收到未知单播帧，进行泛洪（发送到其他所有接口）
2. 收到已知单播帧，进行转发
3. 收到错误数据帧，进行丢弃（如果交换机从一个接口收到一份数据，又要立刻从该接口发出去，就会丢弃）

#### 集线器 

1. 信号放大，解决网线只能传输100m的问题
2. 无脑广播，从一个接口收到数据，会复制N份，从所有其他接口发送出去
3. 半双工，同一时间只能接收数据或发送数据，会使网络造成冲突，所波及的范围叫做冲突域
    1. CSMA/CD：带冲突检测的载波监听多路访问技术，（先听后发，边发边听，冲突停发，随机等待）相当于加锁，解决冲突问题，但会导致网络拥塞
        1. 网桥：通过纯软件实现智能转发，但性能差，没办法用更多的接口
            1. 交换机：通过硬件实现数据转发
4. 无线AP就是hub

#### 网卡

当一个数据包到达一台计算机时，网卡会检查该数据包的目的MAC地址，以判断是否是发往本机的。如果目的MAC地址与网卡MAC地址匹配，或者该数据是一个广播包，网卡就会接受该数据包，并将其传递给内核进一步处理，否则就会丢弃，但是，有一种特殊模式叫做“混杂模式”，这种模式下，网卡会接收所有到达的数据包，通常用于网络监控和抓包分析等场景

### 网络协议

#### 以太网协议

##### 以太网帧数据结构

前导码（8字节），目的MAC地址（6字节），源MAC地址（6字节），上层协议类型（2字节），数据（46-1500字节），帧校验码FCS（4字节）

![](/img/network/ethernet_struct.png)

#### IP协议

##### IP协议的特点

1. IP协议提供了数据报在源和目的地之间的透明传输机制
2. IP协议提供了对长的数据报的分段和重组机制，每个小的数据报被单独封装成以太网帧进行传输，以太网不直接处理大数据包
3. IP协议定义了在互联网的主机间传输数据报的机制，没有提供端对端的数据可靠性、流量控制等机制
4. IP协议可以利用其支持网络的服务，来提供各种类型和服务质量
5. IP协议实现两个基本的功能：寻址和分包
6. IP协议中每个数据报都是独立的，不存在连接或者逻辑链路
7. IP协议使用四个关键机制提供服务
    1. 服务类型：用于表明所希望的服务质量，服务的类型是一个抽象或广义的参数集，将被用于由网关来为特定网络选择世纪的传输参数，以用于下一跳，或者路由网际数据包时下一网关的网络
    2. 生存时间：是一个互联网数据报的寿命上限，它是由该数据报的发送者设置，并沿着处理它的节点递减，如果在数据报达到目的地前生存时间变为0，则数据报被丢弃
    3. 选项：在某些情况下非常有用，但不是必须的，这些选项包括时间错规定，安全和特殊路由
    4. 报头校验：用来验证数据报，如果校验失败，则丢弃报文

##### IP包头部

（20固定+40可选）

1. 版本号（Version）：4bit，标识目前采用的IP协议的版本号，一般值为0100（IPV4）0110（IPV6）
2. IP包头长度（Internet Header Length）：4bit，以32bit为单位的IP包头长度值，最大包头为1111*32bit=60Byte，因为IP包头中有变长的可选部分
3. 服务类型：8bit（Precedence、D、T、R、0、0）
    1. 0-2bit：优先权Precedence
        1. 000：普通
        2. 001：优先
        3. 010：立即发送
        4. 011：闪电式的
        5. 100：比闪电还闪电
        6. 101：CRITIC/ECP
        7. 110：网间控制
        8. 111：网络控制
    2. D：时延 0:普通，1:延迟尽量小
    3. T：吞吐量 0:普通，1:流量尽量大
    4. R：可靠性 0:普通，1:可靠性尽量大
    5. 0：最后两位被保留
4. IP包总长（Total Length）：16bit，以字节为单位计算的IP包长度（头部和数据），所以IP包最大长度65535=2^15 - 1
5. 标识符（Identifier）：16bit，该字段和Flags和Fragment Offset字段联合使用，对较大的上层数据包进行分段操作，路由器将一个包拆分后，所有拆分开的小包被标记相同的值，以便目的端设备能够区分哪个包属于被拆分开的包的一部分
6. 标记（Flags）：3bitt，该字段第一位不使用，第二位是DF（Don`t Fragment）不分段，第三位是MF（More Fragment）更多分段
7. 片偏移（Fragment Offset）：13bit，表示该IP包在该组分片包中的位置，接收端靠此来组装还原IP包
8. 生存时间（TTL）：8bit，当IP包进行传输时，先会对该字段赋予某个特定的值，当IP包进过每一个沿途的路由器时，将其减1，如果TTL减少为0，则该IP包丢弃。这个字段可以防止由于路由环路而导致的IP包在网络中不停被转发
9. 协议（Protocol）：8bit，标识上层所使用的协议
    1. 1：ICMP
    2. 2：IGMP
    3. 6：TCP
    4. 17：UDP
    5. 88：IGRP
    6. 89：OSPF
10. 头部校验（Header Checksum）：16bit，用来校验IP头部，因为每个路由器要改变TTL的值，所以路由器会为每个通过的数据包重新计算这个值
11. 源IP和目的IP（Source and Destination Address）：这两个字段都是32bit，要注意除非使用NAT，否则整个传输过程中。这两个地址都不会改变
12. 选项字段（Options）：变长的选项字段

##### IPv4头数据结构

![](/img/network/ipv4_struct.png)

##### IP地址的分类

1. 网络地址：IP地址由网络号（包括子网号）和主机号组成，网络地址的主机号全为0，网络地址代表着整个网络。
2. 广播地址：广播地址通常称为直接广播地址，是为了区分受限广播地址；广播地址与网络地址的主机号正好相反，广播地址中，主机号全为1，当向某个网络的广播地址发送消息时，该网络的所有主机都能收到该广播消息。（本机没有IP时，使用本地广播地址：255.255.255.255）
3. 组播地址：D类地址就是组播地址。

IP地址分类

1. A类地址：（0）+（7位网络地址）+（24位主机地址）0代表任何地址，127为回环测试地址，所以A类地址实际范围是1-126
2. B类地址：（10）+（14位网络地址）+（16位主机地址） 范围：127-191
3. C类地址：（110）+（21位网络地址）+（8位主机地址）范围：192-223
4. D类地址（组播地址）范围：224-239
5. E类地址（保留地址）范围：240-254

私有地址

1. A类私有地址：10.0.0.0~10.255.255.255
2. B类私有地址：172.16.0.0~172.31.255.255
3. C类私有地址：192.168.0.0~192.168.255.255



IP地址管理划分的方式

1. 有类IP地址：A、B、C、D（组播地址）、E（保留） 类地址，IP地址=网络号+主机号
    1. 弊端：ABC类网络对网络数和主机数都有固定的限制，使得网络管理与分配不方便，比如现在有1000台主机，为其分配一个C类网络则主机数（254）不足，分配一个B类网络则主机数（65534）浪费
2. 无类域间路由（CIDR）：它不区分ABC类地址，而是使用CIDR的值指定地址中作为网络ID的位数，基于可变长子网掩码（VLSM），使网络分配更加灵活



#### TCP协议

TCP（Transmission Control Protocol）：在不可靠的网络上为host-to-host的通信双方提供可靠的逻辑链路

##### TCP协议的特点

1. 基本数据传输：连接是双向通信的，字节流，字节封装成包，TCP自己决定何时停止和发送数据，如果需要确保已经提交的数据及时发送，可以使用push功能
2. 可靠性：TCP通过序号、确认、重传机制确保数据的可靠。TCP必须是可以恢复以下数据：受损的、丢失的、重复的、乱序的，每一个字节一个编号，接收者确认，如果确认超时未收到应该重发，接受者根据编号来排序错序的报文和去除重复的报文，受损的数据可以通过报文段的校验码来控制，丢弃受损的数据，超时重发
3. 流量控制：TCP使用滑动窗口机制进行流控制，以便适应网络的状况和接收方的处理能力。接受者控制发送者数据发送。窗口机制，每个ACK报文段返回一个窗口值，确认以后最多能发送的字节数
4. 多路复用：同一个机器上可以建立多个TCP连接，ip+port=socket，两个socket确认一个连接
5. 连接：描述当前数据流状态的一些信息的组合，由于需要在不可靠的主机和网络上进行通信，握手具有基于时钟的序列号机制用于避免连接的错误初始化
6. 优先级和安全性：TCP的用户可以表明他们的安全性和优先级
7. 分段：TCP在发送大数据流时，会将数据分段，每个段的大小通常不会超过MTU，以避免在网络层发生分片

##### TCP报文头部

（20固定+40可选）

1. 源端口：16bit，范围0-65535
2. 目的端口：16bit，范围0-65535
3. 数据序号（sequence number）：32bit，TCP连接中传送的数据流中每一个字节都编上一个序号，序号字段的值则指的是本报文段所发送的数据的第一个字节的序号
4. 确认序号（acknoledgement number）：32bit，期望收到对方下一个报文段的数据的第一个字节的序号
5. 数据偏移：4bit，它指出报文数据距TCP报头的起始处有多远（TCP报文头长度）
6. 保留字段：6bit，保留日后使用，目前置0处理
7. FLAGS
    1. 紧急比特（URG）：1bit，当URG=1时，表明紧急指针字段有效，它告诉系统此报文段中有紧急数据，应当尽快传送（相当于高优先级的数据）
    2. 确认比特（ACK）：1bit，只有当ACK=1时确认序号字段才有效
    3. 推送比特（PSH）：1bit，接收方TCP收到推送比特置1的报文段，就尽快地交付给接收应用程序，而不再等到整个缓存都填满了后再向上交付
    4. 复位比特（RST）：当RST=1时，表明TCP连接中出现严重差错（如由于主机崩溃或其他原因），必须释放连接，然后再重新建立连接
    5. 同步比特（SYN）：1bit，表示一个连接请求或连接接受请求
    6. 终止比特（FIN）：1bit，用于释放一个连接，表明此报文段的发送端数据已经发送完毕，并要求释放连接
8. 窗口大小：16bit，窗口字段用来控制对方发送的数据量，单位为字节，TCP连接的一端根据设置的缓存空间大小确定自己的接收窗口大小，然后通知对方以确定对方的发送窗口上限
9. 检验：16bit，检验字段检验的范围包括首部和数据这两部分，在计算检验时，要在TCP报文段的前面加上12字节的伪首部
10. 紧急指针字段：16bit，紧急指针指出本报文段中的紧急数据的最后一个字节序号
11. 选项字段：长度可变，TCP首部可以有多达40字节的可选信息，用于把附加信息传递给终点，或用来对齐其他选项，这部分最多包含40字节，因为TCP头部最长是60字节，其中包含前面讨论的20字节固定部分

![](/img/network/tcp_struct.png)

##### TCP连接的状态

1. LISTEN：等待远程的TCP连接请求
2. SYN-SEND：发送了建立连接的请求，等待确认消息
3. SYN-RECIVEVED：收到对方的建立连接请求，且回复了建立连接的请求，等待对方确认
4. ESTABLISHED：连接已经建立，可以正常进行数据传输
5. FIN-WAIT-1：等待对方确认刚刚发送的关闭连接请求
6. FIN-WAIT-2：收到关闭连接请求的确认，等待对方发送关闭连接的请求
7. CLOSE-WAIT：确认了对方的关闭连接请求，等待本地用户关闭连接指令
8. LAST-ACK：被动关闭的一方，在CLOSE-WAIT状态下收到本地用户关闭连接的指令，发送关闭连接请求，等待确认
9. TIME-WAIT：主动关闭连接的一方收到对方发送的对方关闭连接请求的确认消息后，等待2MSL以确保对方接收到ACK包，最后回到CLOSED状态，释放网络资源
10. CLOSED：关闭状态

##### 三次握手

1. 第一次握手：客户端向服务端发送连接请求包，标识位SYN=1，序号Seq=X
2. 第二次握手：服务器收到客户端发过来的报文，由SYN=1知道客户端要建立连接，向客户端发送一个SYN和ACK都置为1的TCP报文，设置初始序号Seq=Y，将确认序号Ack设置为客户端序号加1（Ack=X+1）
3. 第三次握手：客户端收到服务端发来的包后，检查确认序号Ack是否正确，若正确，客户端再次发送确认包ACK=1，Seq=X+1，Ack=Y+1，服务端收到确认序号则表示连接建立成功
4. SYN; Seq=X
5. SYN,ACK; Ack=X+1,Seq=Y
6. ACK; Ack=Y+1,Seq=X+1

![](/img/network/three_times_handshake.png)

##### 四次挥手

1. 第一次挥手：服务器端发出FIN、ACK，用来断开服务器到客户端的数据传送，进入FIN-WAIT-1状态
2. 第二次挥手：客户端收到服务器端的FIN后，发送ACK确认报文，进入CLOSE-WAIT状态
3. 第三次挥手：客户端发出FIN、ACK，用来断开客户端到服务器端的数据 传送，进入LAST-ACK状态
4. 第四次挥手：服务器端收到客户端的FIN后，发送ACK确认报文，进入TIME-WAIT状态，服务器等待两个最长报文过期时间后进入CLOSE状态，客户端收到ACK后，立即进入CLOSE状态
5. FIN,ACK; Ack=Y,Seq=X; FIN-WAIT-1（客户端发起断连请求，表示当前没有数据需要发送）
6. ACK; Ack=X+1,Seq=Y; CLOSE-WAIT（服务端回复收到，进入关闭等待状态，不再发送数据）
7. FIN,ACK; Ack=X+1,Seq=Y; LAST-ACK（客户端没有数据发送后，发出断连请求，等待回复）
8. ACK; Ack=Y+1,Seq=X+1; TIME-WAIT（服务端回复收到，等待两个MSL进入CLOSE状态，客户端收到ACK立即CLOSE）

为什么需要四次挥手

    1. TCP是全双工模式，需要两边的连接全部关闭，此TCP会话才算完全关闭，四次挥手使得TCP的全双工连接能够可靠终止
    2. 四次挥手时，当收到对方FIN报文时，仅仅表示对方不再发送数据但是还能接收数据，己方时否现在关闭数据通道，需要上层应用来决定，因此己方ACK和FIN一般都会分开发送

为什么服务器端最后还需要等待2MSL（Maximum Segment Lifetime 报文最大生存时间）

    1. 当TCP的一端发起主动关闭，在发出最后一个ACK包后就进入来TIME_WAIT状态，必须在此状态上停留两倍的MSL时间，等待2MSL时间主要目的是怕最后一个ACK包对方没有收到，那么对方在超时后将重发第三次挥手的FIN包，主动关闭端接到重发的FIN包后可以再发一个ACK应答包
    2. 2MSL确保在关闭连接后，来自该连接的旧的报文在网络中消失，也为了防止最后一个ACK没有被对方收到，对方会继续发送FIN，然而如果此时自己已经关闭则无法收到FIN以及发送ACK，对方就一直处于LAST-ACK状态

![](/img/network/four_waves.png)

##### TCP状态机

![](/img/network/tcp_state.png)



##### 可靠传输

为了保证不丢包。对于发送的包都要进行应答，但是这个应答也不是一个一个来的，而是会应答某个之前的ID，表示都收到了，这种模式称为累计确认或者累计应答（cumulative acknowledgment）

自适应重传算法：TCP通过采样RTT（往返时延）的时间，然后进行加权平均，算出一个值，而且这个值还是要不断变化的。

快速重传算法：当发送方收到3个连续的重复确认时，它会立即重传丢失的报文段，而不需要等待重传计时器超时。这样一来，快速重传算法能够在较短时间内检测到报文段丢失并触发重传，提高了数据传输的效率

##### 流量控制

通过滑动窗口算法实现流量控制，滑动窗口可以动态调整发送方和接收方的数据传输速率，避免接收方因处理能力不足而被发送方发送的大量数据淹没

##### 拥塞控制

TCP 的拥塞控制主要来避免两种现象，包丢失和超时重传。一旦出现了这些现象就说明，发送速度太快了，要慢一点。

BBR拥塞算法：BBR的主要思想是利用瓶颈链路的带宽和往返时间（RTT）来调整发送速率，而不是依赖丢包信号。BBR通过测量链路的带宽和最小往返时间来估算网络的传输能力，然后根据这些信息调整发送速率。

#### UDP协议

UDP（User Datagram Protocol）：用户数据报协议，面向数据报的传输层协议，不建立连接，不提供可靠性、传输速度快

##### UDP的特点

1. 无连接无状态：UDP是一种无连接、无状态的协议，它不会对数据包进行分段或组装，如果应用层数据大于MTU，那么网络层会发生分片，UDP协议在传输数据前不需要建立连接
2. 尽最大努力交付的：也就是说UDP协议无法保证数据能够准确交付到目的主机
3. 面向报文的：UDP协议将应用层传输下来的数据封装在一个UDP包中，不进行拆分或合并，因此，传输层在收到对方UDP包后，会去掉首部后，将数据原封不动的交给应用进程
4. 没有拥塞控制：UDP协议的发送速率不受网络的拥塞度影响

##### UDP报文头部

（8字节）

    1. 源端口：16bit，范围0-65535
    2. 目的端口：16bit，范围0-65535
    3. 长度：UDP报文的整个大小，最小为8字节（仅为首部）
    4. 检验：16bit，检验字段检验的范围包括首部和数据这两部分，在计算检验时，要在UDP报文段的前面加上12字节的伪首部（4字节的源IP、4字节的目的IP、1字节的0、1字节的17、2字节的UDP报文长度）

![](/img/network/udp_struct.png)

#### ICMP协议

ICMP是啥；ICMP（互联网控制报文协议），是一个介于网络层和传输层之间的一个协议，主要功能是传输网络诊断信息

##### ICMP头部

1. 类型type：1字节，表示较大范围类型分类的ICMP报文
    1. 0：ping应答
    2. 3：目的不可达
        1. code=0：网络不可达
        2. code=1：主机不可达
        3. code=2：协议不可达
        4. code=3：端口不可达
        5. code=4：要分片但设置了不分片，路由器MTU限制需分片但无法分片
    3. 4：源端抑制，表示阻塞
        1. 属于差错信息，如果某个源主机向目的主机快速地发送数据包，但主机来不及处理，就会向源主机发出该类型的ICMP包，提醒主机放慢发送速度
    4. 5：重定向，表示最优的路径
        1. 属于差错信息，如果源主机向网络发送一个IP包，路径中某个路由器收到这个IP包，对照其路由表，发现自己不应该接收该包（包需要原路返回，或者不是最佳路径），就会向源主机发送该类型的ICMP包，提醒源主机修改自己的路由表，下次路由到另外一个更好的路由器
        2. code=1：主机重定向数据报，指示一个可选的路由器/主机
    5. 8：ping请求
    6. 9：路由器通告，告知路由器地址
    7. 10：路由器请求，请求路由器通告
    8. 11：超时，TTL=0
        1. 属于差错信息，超时定义了数据包在网络中存活的最长时间，会随着经过的路由器而递减，当减为0时，就认为该IP包超时，然后当前减为0的路由器会向源主机发送ICMP包
        2. code=0：在传输期间时间超时，跳数限制/TTL超时
        3. code=1：分片重组时间超时，重组计时器超时之前，有分片未到达
    9. 12：参数问题，有问题的报文
2. 代码code：1字节，表示较小范围类型分类的ICMP报文
3. 校验和checksum：2字节，校验头部加数据部分

##### 相关命令

1. 路径追踪命令：traceroute 
    1. 确定通信双方路径上经过的路由器设备
        1. traceroute向目的地发送IP包，刚开始的时候将TTL设置为1，当经过第一个路由器时，TTL-1=0引发超时错误，第一个路由器回复ICMP超时报文，源主机就可以知道第一个路由器的信息，TTL=1的报文会重复请求三次
        2. traceroute向目的地发送IP包，将TTL设置自增再请求，获取沿途所有的路由器信息（不是所有路由器都会返回ICMP超时报文，管理员会主动配置，所以traceroute不一定能拿到所有的路由器信息）
    2. 确定UDP包是否成功到达目的地
    3. 发现路径MTU
2. ping命令：ping
    1. ping包从一台主机发送到另一台主机，包括请求包和应答包，其中，如果MAC地址未知的话，需要先发出ARP请求拿到，然后再进行封装

#### 路由协议

##### 路由选择

距离矢量路由协议：RIPv2，把自己的路由表发送给邻居，收敛速度慢

链路状态路由协议：OSPF，把自己的链路状态通告（LSA）发送给邻居，收敛速度快


RIPv2：基于跳数选择路由，传递路由表，不计算带宽

OSPF：开放式最短路径协议，传递接口信息，可扩展性好，每一个路由器都有一个完整的拓扑，网络发生变化时能很快响应


两条目的网络相同的路由：选AD值最小的，如果AD值一样选择Metric小的，都一样则负载均衡

不同网络之间的选路（都能到达目的网络）：最长掩码匹配原则

mac地址学习：一个mac地址只能对应一个接口，后学习到的会覆盖先学习到的

##### 生成树协议（STP）

通过算法算出来阻塞哪个接口，进而消除环路，而且，当正常的链路断掉之后，阻塞的接口会自动打开

PVSTP+：思科私有协议，每个vlan一个stp，解决vlan次优问题

1. 一个根桥：每个二层拓扑中，必须要有一个根桥（一个特殊的交换机）
2. 两种度量
    1. ID：
        1. BID（bridge ID）：BID由两部分组成，第一个部分是16bit的priority（默认是32768），第二部分是48bit的MAC地址（交换机接口中最小的MAC地址），优先级最小的就是根设备，如果优先级一样，那么比较MAC地址，MAC地址小的就是根设备，BID小的就一定是根设备，根桥的BID就叫做RID（root ID）
        2. PID（port ID）；PID由两部分组成，第一部分是接口优先级（默认128），第二部分是接口号
    2. Cost：开销，只有把数据从接口发出去才考虑开销
        1. 路径开销：在交换机上把数据从接口发出去的开销，思科设备开销：10M贷款=100cost，100M带宽=19cost，1000M带宽=4cost
        2. 根路径开销RPC（root path cost）：非根设备到达根设备的路径开销之和，一般我们说的非根设备开销都是最小开销
3. 三要素选举
    1. 在每个非根设备上选举一个根端口（RP->root port），每个非根设备上有且只能有一个RP（离根最近的接口）
    2. 在每段链路上都要选择一个指定端口（DP->design port），在每段链路上有且只有一个DP，在一段链路里要比较每个设备的根路径的开销，根路径开销小的会成为DP端口，如果根路径开销一样，那么比较BID，BID小的会成为DP
    3. 既不是RP端口也不是DP端口，就将其阻塞掉
4. 四个比较原则：BPDU报文
    1. RID
    2. RPC
    3. 发送者的BID
    4. 发送者的PID
5. 五种端口状态
    1. disable：生成树关闭
    2. blocking：阻塞接口的最终状态
    3. listening：设备开机后，生成树接口默认处于的状态
    4. learning：该状态是学习MAC地址的状态
    5. forwarding：转发状态（只有处在该状态才能转发数据）

#### ARP协议

Address Resolution Protocol 地址解析协议

是一种在TCP/IP网络中用于将网络层的IP地址映射到数据链路层的物理地址的协议。它通常运行在本地网络的所有设备上

1. 发送ARP请求：当一台设备要与另一台设备通信时，它首先检查本地的ARP缓存，如果没有找到对应的物理地址，则会发送一个广播到本地网络上的所有设备，询问指定IP相对应的MAC地址
2. 接收ARP请求的设备响应：本地网络上的所有设备都会接收到ARP请求，但只有IP地址与请求中指定的相同的设备才会响应，其余设备会忽略请求，响应的设备会向请求的设备发送一个ARP响应，其中 包含自己的物理地址
3. 更新ARP缓存：当请求的设备接收到响应后，它将更新自己的ARP缓存，将响应中的IP地址与物理地址存储在缓存中，下次与该设备通信时，请求设备会使用此缓存中的物理地址，而不需要再次发送ARP请求
4. ARP欺骗：ARP欺骗时一种攻击方法，它通过发送虚假的ARP响应来欺骗其他设备，攻击者可以伪装自己的MAC地址，使其他设备将数据发送到攻击者的设备上，从而进行窃听或者篡改网络流量，为防止ARP欺骗，网络管理员可以使用静态ARP表或者动态ARP检测等方法进行防范
5. 代理ARP：当电脑没有网关时，ARP请求目标跨网段，ARP直接询问目标IP对应的MAC地址，网关设备收到ARP请求，会用自己的MAC地址返回给请求者，这便是Proxy ARP；当电脑有网关时，ARP只需询问网关IP对应的MAC地址，采用正常ARP；无论正常ARP还是代理ARP，电脑最终都拿到同一个目标MAC地址，即网关MAC

### 名词解释

1. MAC地址：以太网的数据帧中，目的MAC地址决定了该数据帧是单播还是多播
    1. 如果MAC地址中，第8个bit为0，那么这就是一个单播的MAC地址
    2. 如果第8个bit为1，就是一个多播的MAC地址，全部MAC地址为1是广播，不全为1是组播
2. 网关：为局域网内的用户提供了一扇门，通过该门，可以访问到别的网络，这个门就叫做网关，路由器的每个接口都代表一个不同的网络
3. 泛洪与广播
    1. 广播是向同一子网内所有端口（包括自己的那个端口）发送消息
    2. 泛洪是在所有端口中不包含发送消息的那个端口发送消息
    3. SYN泛洪攻击：利用TCP三次握手机制，攻击端利用伪造的IP地址向被攻击端发送请求，而被攻击端发出的响应报文将永远发送不到目的地，那么被攻击端在等待关闭这个连接 的过程中消耗了资源
4. DNS（Domain Name System）域名系统
    1. 用于命名组织到域层次结构重的计算机和网络服务
5. DHCP（Dynamic Host Configuration Protocol）动态主机设置协议
    1. 基于UDP，给内部网络或网络服务供应商自动分配IP地址，给用户或内部网络管理员作为对所有计算机作中央管理的手段
6. 传输介质
    1. 同轴电缆：铜芯，绝缘塑料，信号屏蔽线圈，塑料外壳（不再使用）
    2. 光纤：玻璃光芯
    3. 双绞线
        1. T568B线序：橙白、橙、绿白、蓝、蓝白、绿、白棕、棕 （橙绿蓝棕）
7. VLAN（虚拟局域网）：控制广播域范围，增强局域网安全性
8. VXLAN（虚拟可扩展局域网）：
    1. VXLAN是一种基于IP网络构建逻辑拓扑，采用“MAC in UDP”封装方式的二层VPN技术，VXLAN能够为分散的物理站点提供二层和三层互联，并能为不同用户提供业务隔离服务
    2. VXLAN建立在原来的IP网络之上，只要三层可达的网络就能部署VXLAN，在每个端点都有一个vtep负责vxlan协议报文的封包和解包，也就是在虚拟报文上封装vtep通信的报文头部

### EVE-NG

#### eve-ng安装

1. 导入EVE-student_2.0.3.ova，win和mac共用
2. 安装SecureCRT（也可以用其他，需要配置启动命令为telnet）和wireshark，用于配置路由器和抓包
3. 安装RCDefaultApp-2.1.X.dmg，会在mac设置的菜单栏添加一个预设应用程序，将URLs中的telnet设置为SecureCRT
4. 安装UNL_WiresharkV2，用于连接eve和wireshark
5. 在eve-web上添加路由器抓包试试吧！

##### 静态IP配置

```plain
cat /etc/network/interfaces 
auto eth0 
iface eth0 inet static 
address 192.168.1.1 
netmask 255.255.255.0 
network 192.168.1.0 
gateway 192.168.1.254 
broadcast 192.168.1.255 
dns-nameservers 223.5.5.5 

#重启⽹络服务 
/etc/init.d/networking restart 
systemctl restart networking 

参考：https://codeantenna.com/a/8NHnblhWfx
```

##### 镜像导入

1. 上传镜像到特定目录（镜像保存目录：/opt/unetlab/addons/ ）
    1. /opt/unetlab/addons/dynamips：Dynamips镜像保存目录
    2. /opt/unetlab/addons/iol：iol镜像保存目录
    3. /opt/unetlab/addons/qemu：qemu镜像保存目录
        1. linux镜像下载：[https://www.eve-ng.net/index.php/documentation/howtos/howto-create-own-linux-host-image/](https://www.eve-ng.net/index.php/documentation/howtos/howto-create-own-linux-host-image/)
2. 如果是[IOL镜像](https://www.eve-ng.net/index.php/documentation/howtos/howto-add-cisco-iol-ios-on-linux/ )则需要创建license，使用[CiscoIOUKeygen.py](https://github.com/pnetlabrepo/ishare2/blob/2480e0b93457a935527232f4192550d2f15059a0/iol/bin/CiscoIOUKeygen.py#L4)创建，或使用虚拟机内置的脚本python2 /opt/unetlab/addons/iol/bin/crack.py（这段代码随着eve服务器的hostname变动，计算的license也不同）
3. 修正镜像权限（/opt/unetlab/wrappers/unl_wrapper -a fixpermissions ）
4. 在eve-web上试试吧！

| linux-alpine-3.18.4 | telnet | root/eve |
| --- | --- | --- |
| linux-centos-8/9-stream | vnc | user/Test123 |

##### 资源列表

1. eve 镜像列表：[https://www.eve-ng.net/index.php/documentation/howtos/howto-add-cisco-iol-ios-on-linux/](https://www.eve-ng.net/index.php/documentation/howtos/howto-add-cisco-iol-ios-on-linux/)
2. eve cisco iol 镜像下载：
    1. [http://dl.nextadmin.net/dl/EVE-NG-image/iol/bin/](http://dl.nextadmin.net/dl/EVE-NG-image/iol/bin/)
    2. [http://dl.nextadmin.net/dl/EVE-NG-image/qemu/](http://dl.nextadmin.net/dl/EVE-NG-image/qemu/)
3. eve-ng教程：
    1. [https://blog.csdn.net/liuhuayeyu/article/details/124082469](https://blog.csdn.net/liuhuayeyu/article/details/124082469)
    2. [http://www.zh-cjh.com/wenzhangguilei/1164.html](http://www.zh-cjh.com/wenzhangguilei/1164.html)

##### 故障解决

1. 解决由于公钥不可用，无法验证以下签名问题
    1. apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 40976EAXXXXXXXX
2. 实验设备的运行目录/opt/unetlab/tmp/0/，每个实验都会将所有用到的设备文件复制到这个目录

#### 思科路由器使用

##### 三大模式

用户模式：设备开机后进入用户模式（Router>）">"表示用户模式

特权模式：用户模式输入“enable”进入特权模式（Router#）"#"表示特权模式，disable退出特权模式，大部分查看命令需要特权模式

全局模式：特权模式输入"configure terminal"（Router(config)），exit退出全局模式，大部分写命令需要全局模式，特权模式下的命令前加 "do" ，就可以在全局模式下运行

##### 命令帮助

tab： 直接补全命令

<command>? ：列出命令补全列表 

? ：显示命令提示，空格翻页， <cr>代表可以回车了

##### 常用命令

```plain
路由器改名 全局模式下: hostname <name>

进入接口配置: interface <接口：e0/0> 

配置ip: ip address <IP> <掩码> 

启停接口（命令前加 no 表示取消命令）: shutdown 或 no shutdown 

查看接口详细信息(ip、是否开启): show interface <接口>

查看多个接口IP地址简洁信息: show ip interface brief 

显示曾经配置过的命令: show running-config 

查询逐行匹配: show running-config | include <搜索字符串>

查询匹配一段: show running-config | section <搜索字符串>

查看路由表：show ip route

关闭路由器路由功能：no ip routing

保存正在运行的配置: copy running-config startup-config 或 write 

清空下次开机的配置文件(以下命令等同): write erase 或 erase startup-config 或 erase nvram 

设置特权模式密码: enable password <密码> 

ping命令：ping <目的IP> [source <出口接口/IP>] [size <包大小>] [repeat <重复次数>]

静态路由：ip route <目的网络> <掩码> <下一跳> [管理距离，用于选路]

查看vlan表：show vlan

查看mac地址表：show mac address-table

查看trunk接口：show interfaces trunk

创建vlan：vlan <n1>[,<n2>]

vlan改名 vlan模式下：name <vlan新名>

接口模式批量执行命令：interface range <en/n-m,en/m>

配置网关：ip default-gateway <网关IP> 

将交换机接口设置为三层接口（可以配置IP）：no switchport

查看生成树：show spanning-tree

进入rip路由模式：router rip
  修改rip的版本：version 2 
  关闭自动汇总：no auto-summary
  宣告网络：network <网络>
  
查看NAT转换表项：show ip nat translations

终止命令: ctrl+shift+6 

全局模式大于特权模式 
大部分查看命令在特权模式执行 
全局模式下do <command> 可以执行特权模式的命令
```

##### telnet管理远程设备

1. 只需要被控端设置即可
2. 前提是网络必须要通
3. 设置特权模式密码，才能在telnet中进入特权模式：R(config)# enable password my-pwd-enable
4. 进入线路模式：R(config)# line vty 0 4
    1. 开启telnet：R(config-line)# transport input telnet
    2. 设置telnet密码：R(config-line)# password my-pwd-telnet
    3. 启用登录验证，要求用户登录时输入密码：R(config-line)# login

##### vlan配置

背景：终端设备太多广播域太大，使用vlan划分广播域

1. access链路
    1. 创建vlan：sw(config)#vlan 10
        1. 创建后进入vlan模式，exit退出后生效
    2. 将交换机接口加入vlan
        1. 设置交换机接口模式为access：sw(config-if)#switchport mode access
        2. 将交换机接口加入vlan10：sw(config-if)#switchport access vlan 10
    3. access链路只能承载一个vlan，一个接口只能转发一个vlan的数据，所以整个vlan所有途径的交换机都需要有一个接口设置为相同vlan
2. trunk链路
    1. 启用802.1q协议：sw(config-if)#switchport trunk encapsulation dot1q
        1. 必须启用802.1q协议才能将交换机接口设置为trunk模式
        2. 802.1q协议封装以太网数据帧加入了四个字节的数据，用于存放vlan相关信息，其中12位存放vlan号
        3. tunck链路上的数据包都会封装802.1q协议，到达access链路时会卸载掉
        4. 本征vlan：trunk链路允许有一个vlan不用封装802.1q协议，就是本征vlan（Native vlan），默认为vlan1
            1. 修改本征vlan：sw(config-if)#switchport trunk native vlan 20
            2. 链路两端都需要修改为相同本证vlan，仅仅当前段传递本征vlan的数据不封装802.1q协议
            3. 设置本征vlan也封装802.1q协议：sw(config)#vlan dot1q tag native
    2. 设置交换机接口模式为trunk：sw(config-if)#switchport mode trunk
    3. trunk模式接口允许交换机转发多个vlan的数据，解决了access链路只能承载一个vlan的问题。每个交换机都需要创建允许转发的vlan

##### 单臂路由

多臂路由：路由器和交换机之间连接多条网线，每条网线对应一个vlan

单臂路由：路由器和交换机之间连接一条网线（使用子接口替换有限的物理接口）

1. 路由器和交换机之间需要承载多个vlan的数据：使用trunk链路
2. 路由器的物理接口不能认识带有vlan标签的数据（802.1q）：终结子接口
    1. 在子接口上使用encapsulation dot1Q 10，让子接口支持识别和封装802.1q协议
    2. 子接口收到一个带标签的数据，会摘掉自己所认识的标签；发送一个数据的时候，如果该数据不带标签，打上自己所认识的标签
3. 路由器的物理接口只能配置一个IP地址：子接口可以配置IP地址，可以作为网关

##### 三层交换机

背景：单臂路由需要经过路由器，使用三层交换机直接在交换机路由数据更方便

SVI（switch virtual interface）：交换机虚拟接口（将多个接口逻辑成一个）

1. 启动交换机的路由功能：ip routing
2. 创建SVI：interface vlan <n>
3. SVI配置IP：ip address <ip> <mask>
4. 仅仅有接口能传递当前vlan的数据的时候，SVI接口才会up

三层交换机概念：

1. 有IP处理能力，能配置一个以上的SVI接口的交换机就叫做三层交换机
2. 某些二层交换机是可以配置SVI接口的，但是只能配置一个，通常是vlan1的SVI接口，一般用于配置IP地址进行telnet远程管理

##### DHCP配置

1. 配置DHCP服务器
    1. 配置一个IP地址池：ip dhcp pool <pool name>
    2. 在IP地址池里配置一个IP地址段：network <网段> <掩码>
    3. 配置DHCP服务器在分配IP地址的同时分配网关地址（可选）：default-router <ip>
    4. 配置DHCP服务器在分配IP地址的同时分配DNS地址（可选）：dns-server <ip>
    5. 配置一个IP地址的租期（可选）：lease <天数> <小时数> <分钟数>
2. 配置DHCP客户端
    1. 将PC获取地址的方式更改为自动获取（DHCP方式）：ip address dhcp
    2. 开启接口：no shutdown

##### NAT配置

共有地址不能重复，私有地址可以在不同内网重复

1. 静态NAT：一对一转换，一旦配置会立刻生成NAT表项
    1. 指明e0/0为内部
        1. R(config-if)#int e0/0
        2. R(config-if)#ip nat inside
    2. 指明e0/1为外部
        1. R(config-if)#int e0/1
        2. R(config-if)#ip nat outside
    3. 配置静态NAT的一对一转换：inside local转化成inside globle
        1. R(config)#ip nat inside source static <内部IP> <外部IP>
2. 动态NAT：多对多转换，配置一个ip池（在内部访问外部时，才将NAT表项纪录下来）
    1. 指明e0/0为内部
        1. R(config-if)#int e0/0
        2. R(config-if)#ip nat inside
    2. 指明e0/1为外部
        1. R(config-if)#int e0/1
        2. R(config-if)#ip nat outside
    3. 通过ACL来选出一部分inside-local地址
        1. ip access-list standard <acl name>
        2. permit <网络> <特殊掩码>
    4. 做一个nat的global地址池pool
        1. ip nat pool <pool name> <起始IP> <结束IP> netmask <掩码>
    5. 将ACL选出来的inside-local和nat地址池做一个关联
        1. ip nat inside source list <acl name> pool <pool name>
        2. 转变为动态PAT：ip nat inside source list <acl name> pool <pool name> overload
3. PAT：多对一转换，端口地址转换，通过在映射表中加入端口实现IP地址的复用，一段时间不用会自动清除转换表
    1. 指明e0/0为内部
        1. R(config-if)#int e0/0
        2. R(config-if)#ip nat inside
    2. 指明e0/1为外部
        1. R(config-if)#int e0/1
        2. R(config-if)#ip nat outside
    3. 通过ACL来选出一部分inside-local地址
        1. ip access-list standard <acl name>
        2. permit <网络> <特殊掩码>
    4. 将ACL选出来的inside-local和外部接口做一个关联
        1. ip nat inside source list <acl name> interface e0/1 overload

##### ACL配置

1. ACL（访问控制列表）：用于标记流量，允许或拒绝流量通过
    1. 对于一个路由器，假如收到一个数据
        1. 我应该把这个数据转发到哪？：查询路由表
        2. 我应该转发这个数据吗？：查询ACL
        3. 我该怎么转发这个数据？：QoS，根据优先级入队
    2. ACL的表项是由上至下有序的逐条匹配，如果匹配上了，就不需要匹配了，如果都没匹配上，就默认拒绝

### Linux路由管理

#### 路由管理

Linux最多可以支持255张路由表，其中3张表是内置的

+ 表255本地路由表（Local table）：本地接口地址，广播地址，以及NAT地址都放在这个表，该路由表由系统自动维护，管理员不能修改
+ 表254主路由表（Main table）如果咩有指明路由所属表，所有的路由都默认放在这个表里，一般来说，旧的路由工具（route）所添加的路由都会加到这个表，一般是普通的路由
+ 表253默认路由表（Default table）一般来说默认的路由都会放在这张表，但是如果是特别指明放在这张表的，也可以是所有的网关路由

还有一张表0是保留的，在文件/etc/iproute2/rt_tables可以查看和配置路由表的TABLE_ID以及路由表名称

> 查看路由信息

格式：ip route show [table table_name]

+ default：表示这是默认路由
+ via：指定数据包的下一跳
+ dev：指定通过哪个网络接口发送数据包
+ proto：描述了路由的来源或如何得到这条路由，kernel表示内核自动添加，statuc表示静态配置
+ metric：度量值，在到统一目标的多条路由时，metric值越小会被首先选择
+ scope link：表示该路由时直连路由

```plain
[root@VM-12-4-centos ~]# ip route show
default via 10.0.12.1 dev eth0 proto dhcp metric 100
10.0.12.0/22 dev eth0 proto kernel scope link src 10.0.12.4 metric 100
10.88.0.0/16 dev podman0 proto kernel scope link src 10.88.0.1
```

> 添加路由

格式：ip route add <目的网络CIDR | 目的主机> via <网关> dev <发送数据的网络接口> [table table_name]

> 删除路由

格式：ip route del <目的网络CIDR | 目的主机> via <网关> [table table_name]

> 修改路由

格式：ip route change/replace <目的网络CIDR | 目的主机> via <网关> [table table_name]

> 查看指定网段内路由

格式：ip route show root <CIDR> table <table_name>

> 清空路由表

格式：ip route flush table <table_name>

> 路由持久化

route和ip route命令的路由操作都是临时生效

+ 通过将路由信息写入/etc/sysconfig/network-scripts/route-<interfacename>实现持久化
+ 文件内容格式：<目的网络CIDR | 目的主机> via <网关> dev <发送数据的网络接口>

> 接口地址配置

接口添加IP地址：ip address add <CIDR> broadcast <广播地址> dev <网络接口>

接口删除IP地址：ip address del <CIDR> broadcast <广播地址> dev <网络接口>

查看IP地址：ip address show

#### 策略路由

不同IP包可以选择不同的路由表进行路由

> 查看所有规则：ip rule list

格式：优先级： 匹配条件 采取动作

默认情况下进行路由时，会首先根据最小规则进行路由匹配，如果路由失败，就会匹配下一个不为空的规则

```plain
[root@VM-12-4-centos ~]# ip rule list
0:	from all lookup local
32766:	from all lookup main
32767:	from all lookup default
```

> 添加自定义规则

格式：ip rule add <from|to|tos|dev|fwmark> <value> table <table_name> [pref <优先级>][prohibit|reject|unreachable]

table：指明所使用的表，也可以使用lookup

若不指定优先级，默认介于本地路由表和主路由表之间

> 匹配条件

+ from：匹配来源地址
+ to：匹配目的地址
+ tos：匹配IP包头的TOS（type of service）域
+ dev：匹配物理接口
+ fwmark：匹配防火墙MARK标记

> 采取动作

+ nat：透明网关
+ prohibit：丢弃该包，并发送COMM.ADM.PROHIBIT的ICMP信息
+ reject：单纯丢弃该包
+ unreachable：丢弃该包，并发送NET UNREACHABLE 的ICMP信息

```plain
ip rule add from 192.168.1.10 table 10  
ip rule add from 192.168.2.0/24 table 20 
ip rule add to 168.95.1.1 table 10  
ip rule add to 168.96.0.0/24 table 20
ip rule add dev eth2 table 1  
```

> 持久化

```plain
echo "252 storage" >> /etc/iproute2/rt_tables
echo "ip rule add from 192.168.3.0/24 table storage" >> /etc/rc.local
echo "ip route add default via 192.168.3.1 table storage" >> /etc/rc.local
chmod +x /etc/rc.d/rc.local
```

#### 旧的路由工具（route）

> 查看路由信息

格式：route -n （不加n会显示主机名，加上-n会显示具体IP地址）

+ Destination：目标地址
+ Gateway：下一跳的地址。如果目的地是直连，Gateway字段为0.0.0.0
+ Genmask：子网掩码
+ Flags：U表示路由是活跃的，G表示路由使用了网关，H表示路由是一个主机而不是网络
+ Metric：度量值，表示到达目的地的代价或距离，到达统一目标地址的路由，metruc越小越优先
+ Use：该路由被使用的次数，并不准确
+ Iface：指示那个网络接口用于该路由

```plain
[root@VM-12-4-centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.12.1       0.0.0.0         UG    100    0        0 eth0
10.0.12.0       0.0.0.0         255.255.252.0   U     100    0        0 eth0
10.88.0.0       0.0.0.0         255.255.0.0     U     0      0        0 podman0
```

> 添加路由

route add -net <目的网络> netmask <掩码> gw <网关> dev <发送数据的网络接口>

route add -host <目的主机> gw <网关> dev <网络接口>

> 删除路由

route del -host <目的主机> gw <网关>

route del -net <目的网络> netmask <掩码> gw <网关>

### 参考资料

[^1]: [IP定义RFC791](https://www.rfc-editor.org/rfc/rfc791.txt)

[^2]: [TCP定义RFC793](https://www.rfc-editor.org/rfc/rfc793.txt)

[^3]: [UDP定义RFC768](https://www.rfc-editor.org/rfc/rfc768.txt)

[^4]: [Wireshark抓取TCP三次握手报文](https://blog.csdn.net/happylzs2008/article/details/103267910)

[^5]: [Wireshark抓取TCP四次挥手报文](https://blog.csdn.net/u022812849/article/details/107100864)

[^6]: [网络协议](https://zhangbinalan.gitbooks.io/protocol/content/1wang_luo_xie_yi.html)

[^7]: [Ubuntu 网卡配置](https://www.myfreax.com/set-custom-dns-servers-on-ubuntu-18-or-20/)

[^8]: [eve-web连接wireshark抓包](https://www.802101.com/unetlab-and-wireshark-for-osx-update/)

[^9]: [eve-ng中文社区](https://www.emulatedlab.com/)
