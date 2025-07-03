---
title: 深入理解iptables和netfilter
date: 2025-07-03 17:37:00
categories:
    - [Linux,网络]
tags: 
    - Linux
    - 网络
    - iptables
---
![Netfilter-packet-flow](/img/network/Netfilter-packet-flow.svg)

### iptables是什么

iptables是使用很广泛的防火墙工具之一，它基于内核的包过滤框架netfilter。
netfilter：netfilter提供了5个hook点，包经过协议栈时会触发内核模块注册在这里的处理函数，触发哪个hook取决于包的方向、包的源地址和目标地址、包在上一个hook点是被丢弃还是拒绝等等

不同场景下，包的游走流程：

- 收到目的是本机的包：PREROUTING -> INPUT
- 收到目的不是本机的包：PREROUTING -> FORWARD -> POSTROUTING
- 本地产生的包：OUTPUT -> POSTROUTING
- 本地发给本地的包：OUTPUT -> POSTROUTING -> lo网卡 -> PREROUTING -> INPUT

![netfilter.png](/img/network/netfilter.png)

### iptables的表和链

iptables使用table来组织规则，根据用来做什么类型的判断标准，将规则分为不同table（决定优先级）

- nat：网络地址转换，当包进入协议栈的时候，这些规则决定是否以及如何修改包的源/目的地址，以改变包被路由时的行为。通常用于将包路由到无法直接访问的网络
- filter：包过滤，用于防火墙规则
- mangle：修改包的IP头，例如修改包的TTL增加或减少可以经过的跳数，这个table可以对包打只有内核有效的标记，并不修改包本身，后续的table或工具处理的时候可以使用到这些标记
- raw：连接相关，提供一个让包绕过连接跟踪的框架（建立在netfilter之上的连接追踪特效使得iptables将包看作已有连接或会话的一部分，而不是一个独立、不相关的包组成的流）

在每个table内部，规则被进一步组织成chain，内置的chain是由内置的hook触发的，chain基本上能决定规则何时被匹配

- PREROUTING：接收到的包进入协议栈后立即触发（存在于nat、mangle、raw）
- INPUT：接收到的包进过路由判断，如果目的是本机，将触发（存在于filter、mangle）
- FORWARD：接收到的包进过路由判断，如果目的是其他机器，将触发（存在于nat、filter、mangle）
- OUTPUT：本机产生的准备发送的包，在进入协议栈后立即触发（存在于nat、filter、mangle、raw）
- POSTROUTING：本机产生的准备发送的包或者转发的包，在经过路由判断之后触发（存在于nat、mangle）

### iptables规则

- 持久化：iptables-save > /etc/sysconfig/iptables
- 加载规则：iptables-restore < /etc/sysconfig/iptables
表优先级：raw > mangle > nat > filter

#### 基本命令

- 规则操作
  - --append -A chain 添加一个规则到链末尾
  - --insert -I chain [rulenum] 根据给出的规则序号向所选链中插入一条或多条规则，默认为1，在链头部插入
  - --delete -D chain [rulenum] 删除指定链或指定链的某一条规则
  - --check -C chain 检查一条链是否存在
--replace -R chain rulenum 修改指定链中的某一条规则
- 链操作
  - --list -L [chain [rulenum] ] 列出指定链中的规则
  - --list-rules -S [chain [rulenum] ] 打印出指定链中的规则
  - --flush -F [chain] 删除指定链中的规则
  - --zero -Z [chain [rulenum] ] 把指定链或表中所有链的所有计数器清零
  - --policy -P chain target 该表某条链的默认策略
- 自定义链（其他链引用时触发 -j CUSTOM）
  - --new -N chain 创建一条用户自定义链
  - --delete-chain -X [chain] 删除一条用户自定义链
  - --rename-chain -E old-chain new-chain 修改用户自定义链的名称
选项参数
  - --protocol -p proto 规则或者包检查的协议，tcp、udp、icmp中的一个或多个，默认匹配所有协议
  - --source -s address[/mask] 指定源地址，可以是主机名、网络名、IP地址，mask是网络掩码
  - --destination -d address[/mask] 指定目的地址
  - --jump -j target 执行指定的动作，自定义chain、ACCEPT、DROP、REJECT等等
  - --goto -g chain 跳转到指定链
  - --numeric -n 以数字的形式显示IP地址和端口
  - --in-interface -i input_name 匹配由指定网络接口进入的数据包
  - --out-interface -o output_name 匹配由指定网络接口发出的数据包
  - --line-numbers 当列表显示规则时，在每个规则前面加上行号，与该规则在链中的位置相对应

#### 扩展匹配

--match -m match 扩展匹配

- -m iprange
  - -src-range <ip>-<ip> 指定连续的源地址范围
  - -dst-range <ip>-<ip> 指定连续的目的地址范围
- -m multiport
  - --source-port --sport [!] [port[:port][,port]] 源端口或范围，若首端口号忽略默认为0，若尾端口号忽略默认为65535
  - --destionation-port --dport [!] [port[:port][,port]] 目标端口或目标端口范围
- -m comment
  - -m comment "Allow SSH" 添加注释
- -m state
  - --state 匹配连接状态：NEW、ESTABLISHED、RELATED、INVALIED
- -m limit
  - --limit n/[second | minute | hour | day] 最大平均匹配率，基于收发报文的速率做检查
  - --limit-burst n 通行证数量，默认5
- -m connlimit
  - --connlimit-upto n 如果连接数小于或等于n，则进行匹配
  - --connlimit-above n 如果连接数大于n，则进行匹配

#### 处理动作

- ACCEPT：允许数据包通过
- DROP：直接丢弃数据包，不给任何回应信息，客户端等到超时才有反应
- REJECT：拒绝数据包通过。必要时会给发送端一个响应信息，客户端刚请求就会收到拒绝信息
  - --reject-with type type可以是icmp-netunreachable、port-unreachable（默认）等
  - --echo-reply 它只能用于指定ICMP ping包的规则中，生成ping的回应
  - --tcp-reset 可以用于INPUT链中，或自INPUT链调用的规则，将回应一个TCP RST包
- MASQUERADE：是SNAT的一种特殊形式，适用于动态的、临时会变的IP上
- SNAT：源地址转换，解决内网用户用同一个公网地址上网的问题
  - --to-source [:port-port]
- DNAT：目的地址转换
  - --to-destiontion [:port-port] 可以指定IP地址，IP地址范围、也可以附加一个端口范围
- REDIRECT：在本机做端口映射
  - --to-ports [port-port] 使用指定的目的端口或端口范围
- LOG：在/var/log/messages 文件中记录日志信息，然后将数据包传递给下一条规则，仅仅做记录

### 参考资料

- [深入理解iptables和netfilter架构](https://arthurchiao.art/blog/deep-dive-into-iptables-and-netfilter-arch-zh/)
- [NAT - 网络地址转换](https://arthurchiao.art/blog/nat-zh/)
- [深入架构原理与实践](https://arthurchiao.art/blog/nat-zh/)