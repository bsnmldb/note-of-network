# 网络层 overview

## 功能

##### 1. swithcing/routing

决定分组采用的路由或路径, 控制平面

路由选择算法计算路由表(forwarding table)

##### 2. forwarding

根据转发表, 将分组从一个输入链路转移到适当的输出链路, 数据平面

错误处理 排队 调度

##### 3. connection setup

`IP`不提供

`ATM`,`frame relay`, `X.25`等

路由器参与的,在两个主机之间的连接

## 服务模型

1. 确保交付
2. 具有时延上界的确保交付
3. 按序交付
4. 确保最小带宽
5. 安全性
6. 间隔抖动

`IP`提供尽力而为服务(`best effort`)

# 数据平面

##### CIDR&子网

##### 最长前缀匹配



##### 调度策略

1. FIFO+丢弃尾部超缓冲区部分
2. 优先级队列
3. 轮询队列
4. FQ: 分大小的轮询队列
5. WFQ

##### 交换结构

1. 内存
2. 总线
3. 互联网络

# 两种网络

## 虚电路网络

virtual circuit 

建立连接, 保留资源状态, 保证服务

sb终端-复杂内部

## 分组交换网络

datagram network

共享网络资源(弹性服务), 存储转发, 资源争夺, 拥堵

智能终端(适应不同的网络情况)

# 网络层协议

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071620803.png" alt="image-20230607162003617" style="zoom: 33%;" />

## IPv4

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071624217.png" alt="image-20230607162454383" style="zoom: 33%;" />

1. version(版本): 4

2. IHL(Internet Header Length): 报文头长度, 单位为word(4字节), 一般为5(20字节)

3. ToS(服务类型)

4. Total Length(数据报长度): 单位为字节, 仅计数据的长度

5. Identification, Flags, Fragment Offset

   用于IP分片

6. TTL: 确保数据报不会永远在网络中循环

7. Protocol: 传输层协议类型 

8. Header Checksum(首部检验和): 随着TTL的改变, 路由器需要重新计算checksum

##### 分片

MTU=1500B(Maximum Transmission Unit) 链路层帧能承载的最大传输单元(包括IP头)

对于较大的报文, 在链路层传输前分片, 在目的地运输层以前重新组装

1. identification: 表示片属于哪个包
2. offset: 标识片在包内的偏移(可能片丢失) (单位8字节)
3. flags: 标识最后一片之类的

##### IP地址

$8\times 4=32$位地址

全0网络号, 全1广播号

子网与子网掩码(主机号需预留上面两种地址) -> CIDR表示

路由聚合(路由摘要), 最长前缀匹配

## NAT

Network Address Translation 网络地址转换

目的

1. 充当防火墙
2. 内部分配更多的IP地址
3. 隔离组织与网络

实现种类

1. 静态(static NAT): 一个私有IP -- 一个公共IP(静态映射)

2. 动态(dynamic NAT): 公共IP池动态分配

3. 单地址(single address): 一个公共IP通过port映射到多个私有IP

   对外抽象为一个主机的不同端口, 实际上是一个内部的不同机器的不同端口

问题

1. 地址经常变化(P2P怎么办)
2. 依赖中继

## DHCP ARP

见链路层

## ICMP

Internet Control Message Protocol

传递错误报文和控制报文

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071702598.png" alt="image-20230607170258525" style="zoom:50%;" />

1. ping: echo request, echo reply
2. traceroute: TTL expire
3. path MTU: don't fragment

## Mobile IP

移动设备移动 -> IP变化 -> TCP连接关闭重连?

三角路由!

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071714487.png" alt="image-20230607171429259" style="zoom: 33%;" />

1. discovery: 移动代理和移动节点定期告知主代理
2. registration
   1. 移动节点向移动代理发送registration request
   2. 移动代理转发给主代理
   3. 主代理发送registration reply给移动代理
   4. 移动代理转发给移动节点
3. tunneling:
   1. 主代理将移动节点的IP与自己的MAC绑定广播ARP
   2. 隧道转发给移动代理

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071722330.png" alt="image-20230607172201505" style="zoom:33%;" />

## IPv6

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071723918.png" alt="image-20230607172324599" style="zoom: 50%;" />

1. Version: 6
2. Traffic Class: 流量类型(ToS in IPv4)
3. Flow Label: 流标签, 流标识
4. Payload Length: 有效载荷, 定长40字节首部后的所有部分(附加包头+数据)
5. Next Header: 下一层协议(Protocol in IPv4)
6. Hop Limit: (TTL in IPv4)

与IPv4

1. 20字节 - 40字节
2. 地址32字节 - 128字节
3. 分片相关字段移除
4. 首部检验和移除
5. 只有payload表示长度
6. 新增流标签
7. ToS -> Traffic Class
8. Protocol -> Next Header
9. TTL -> Hop Limit
10. 64字节对齐

优点

1. 扩充地址容量
2. 改进选项机制
3. 支持资源分配
4. 更有效和稳健的移动性机制
5. 更加安全(内置强大的IP层加密和认证)

### 适配

1. dual stack: IPv6信息丢失
2. tunneling: 

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071733109.png" alt="image-20230607173338800" style="zoom:33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071734765.png" alt="image-20230607173359484" style="zoom: 25%;" />

# 路由算法

划分

1. 集中式路由选择算法: 拥有完整的 全局性的信息 (链路状态LS算法)
2. 分散式路由选择算法: 迭代 分布式的 与邻居交换信息

1. 静态路由选择算法: 通常人工配置
2. 动态路由选择算法: 随网络流量负载或拓扑发生变化而改变路由选择路径

1. 负载敏感算法
2. 负载迟钝算法

## LS - dijkstra

集中式路由算法

震荡: 非同步运行路由算法, 链路代价更新随机化

## Bellman-Ford

距离向量路由选择算法(Distance-Vector, DV)

迭代 异步 分布式

##### 初始化

到自己为0,到邻居为边权,不可达节点为无穷

##### 迭代

$$
L_{h+1}(y\neq s)=min_{v}\{L_h(v)+w(v,y)\}
$$

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071934258.png" alt="image-20230607193432434" style="zoom:33%;" />

$L_h$经过所有长度为$h$的路,最短距离

##### 链路开销改变

DV改变,通知邻居,动态迭代调整

good news travles fast, bad news travels slow

造成无穷计数问题

使用`毒性逆转`避免无穷计数

如果z通过y路由到x,z将告诉y它到x无穷距离

没有完全解决,在3个或更多节点的环路上无法检测

## 比较

###### 总览

1. LS: 每个节点广播链路开销,让每个节点都获得全局的链路信息,然后分别计算单源最短路
2. DV: 每个节点仅与其邻居交谈,分布式异步地进行迭代

###### 1. 报文复杂性

1. LS: 每个节点需要知道整个网络状态(每条链路的开销),需要发送$O(|V||E|)$广播报文

   ​    并且在每次链路开销改变时(或者周期性允许时)都需要相同的代价

2. DV: 作为迭代算法,其报文复杂性取决于迭代速度,依赖于许多因素

   ​	当链路开销改变时,这份改变会迭代传播给所有依赖该条链路的节点,需要的代价也不同

###### 2. 收敛速度

1. LS: 在知道链路状态后,一般而言需要$O(n^2)$的时间复杂度才会收敛

   ​	在堆的优化下仅需要$O(n\log n)$就能收敛

   ​	此外在负载敏感时会遭遇链路振荡的问题

2. DV: 收敛速度较慢

   ​	可能遭遇路由环路,无穷计数的问题

###### 3. 鲁棒性

1. LS: 分别地从广播报文中获得链路状态,在状态破坏时一定程度上不会有太大的影响
2. DV: 依赖于邻居传输的距离向量,有坏人时破坏会依次迭代传播下去,扩散到整个网络

# 路由协议

分层路由

`AS`(autonomous systems): 自治系统

+ IGP: `AS`内部自治,运行相同的路由协议
+ EGP: `AS`之间网关路由器运行AS间路由协议

## IGP

Interior Gateway Protocol

Inrea-AS-routing

### RIP

routing information protocol

使用DV算法

app层面实现路由表维护, UDP



### OSPF

open shortest path first

使用LS算法

信息直接IP传播,flooding(洪泛)到整个`AS`

#### 层次化OSPF

`AS`被划分为多个区域

区域内路由器只知道区域内LS信息

1. 将洪泛交换链路状态信息的范围局限于每一个区域而不是整个的自治系统
2. 在一个区域内部的路由器只知道本区域的完整网络拓扑,而不知道其他区域的网络拓扑的情况
3. 主干区域用于连通其他下层区域

### 比较

RIP

1. 配置简单，适用于小型网络（小于15跳）
2. 可分布式实现
3. 收敛速度较慢
4. 网络是一个平面，不适用于大规模网络

OSPF

1. 收敛速度快，无跳数限制
2. 支持不同服务类型选路
3. 支持身份认证
4. 支持层次式网络,适用于大规模复杂网络
5. 集中式算法
6. 每个节点需要维护全局拓扑
7. 配置复杂

## EGP

Exterior Gateway Protocol

Inter-AS-routing

使用BGP(Border Gateway Protocol) 边界网关协议

1. 最好而不是最短

2. path vector

   1. 避免环路
   2. 灵活可选

3. 选择性路由通告

4. 支持CIDR, 可以进行路由聚合

   <img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306072030895.png" alt="image-20230607203002547" style="zoom:25%;" />

+ eBGP: 边界路由器之间 -- 学习外部路由
+ iBGP: 边界路由器与本`AS`之间 -- 在内部分发从外部学习的路由

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306072035054.png" alt="image-20230607203533120" style="zoom: 33%;" />

# 组播

略