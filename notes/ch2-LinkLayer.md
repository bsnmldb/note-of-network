



# 链路层服务

nodes: 终端和路由器

links: 连接nodes的信道

链路层实现在网络适配器(network adapter)/网络接口卡(network interface card,NIC) 和 cpu软件上

是软件和硬件的结合体,也是协议栈中软硬件交接的地方

链路种类

1. 有线点对点(Wired point-to-point)
2. 有线多路接入(Wired multiple access links, LANs)
3. 无线(Wireless links,WiFi)

服务

1. 成帧
2. 链路接入
3. 可靠交付
   1. 流控制
4. 差错检验和纠正

## 成帧(framing)

将数据报(datagram)封装成帧(frame)

增加 差错检测比特,流控制比特 等

## 链路接入(link access)

1. point-to-point

   MAC协议(这里指媒体访问控制)比较简单或者不存在

2. broadcast

   协调多个节点的帧传输

### 多路访问控制

在共享广播信道中,避免碰撞或者从碰撞中恢复

#### MAC协议

##### 期望

1. 分布式的算法

2. 控制信道的信号也要通过该信道传输

   Communication about channel sharing must use channel itself!

##### 想法

1. 切分(Channel partitioning)
   1. TDMA
   2. FDMA
   3. CDMA
2. 轮流(Taking turns)
   1. 轮询(polling)
   2. 令牌环(token ring)
3. 随机访问(Random access)
   1. ALOHA
   2. slotted-ALOHA
   3. CSMA
   4. CSMA/CD
   5. CSMA/CA

## 可靠交付

保证无差错地经链路层移动每个网络层数据报

主要对于易于产生高差错率的链路(无线链路)

### 流控制(flow control)

保证协调同步,不让发送方发送数据超过接收方承载能力,本质上是不让接收方的`buffer`溢出

想法

1. stop and wait
2. sliding window

#### stop and wait

协议

1. 发送方:发送数据,等`ACK`再发下一个
2. 接收方:接收数据并回复`ACK`

接收方通过不发`ACK`来停止流

Work well for large frames

#### sliding window

接收方的`buffer`大小`W`,发送方已知

发送方可以连续发`W`个帧而不需要`ACK`

帧模$2^k$编号,`ACK`包含下一个期待的帧的编号(表示自己已经收到多少号了)

> 要求:$W\leq 2^k$

##### 图示

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191425149.png" alt="image-20230319142240728" style="zoom: 33%;" />

可以看到`W=7`,`ACK`表示已经接收了5号

但是在窗口中已经发送了7号



<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191425639.png" alt="image-20230319142249941" style="zoom: 33%;" />

维护一个窗口,每发一个帧,`left++`,每有一个`ACK`,`right=ACK+W`

##### 差错处理(滑动窗口)

1. 回退`N`

   若有错误,发送`NAK`

   出错帧及以后所有帧重传

2. 选择性拒绝

   只有出错帧需要重传

   但是接收方必须维护很大的`buffer`

## 差错检测和纠正

差错检测和纠正比特(Error-Detection and-Correction,EDC),放在数据末尾

### 奇偶校验(parity checking)

偶校验:通过校验位维护1的个数为偶数

> e.g. 0111000110101011|1 
>
> 数据9个1,校验位1以维护10个(偶数个1)

#### 单比特奇偶校验

未检出差错的概率在50%,数据总是突发聚集出错

#### 二维奇偶校验

对于单比特错误,可以检测并纠正(包括校验比特本身)

也能够检测到两个比特差错的任意组合(不一定能纠正)

> 当两个比特差错在同一行或同一列时不能纠正

>前向纠错(Forward Error Correction,FEC)
>
>接收方检测和纠正差错的能力
>
>避免了不得不等待的往返时延

### 检验和(checksum)

将数据划分为多个16比特的数并相加(模$2^{16}$),和的反码作为检验和

检验方式:求和(包括检验和),要求最后为全1

简单快速但是相对弱的差错保护

### 循环冗余检测(CRC)

协商一个固定的`生成多项式G`,有$r+1$位且最高位为1

给定数据段`D`,附加$r$比特的`CRC`

检验:`D-CRC`这$d+r$比特能够被`G`整除(模2算数)
$$
D\cdot 2^r\ xor\ R=nG\\
D\cdot 2^r=nG\ xor\ R\\
R=remainder\frac{D\cdot 2^r}{G}
$$
每个`CRC`能够检测小于$r+1$比特的突发差错,以一定概率检测到更大的差错

> e.g.<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303022042317.jpg" alt="img" style="zoom: 25%;" />

# 局域网(LAN)

局域网(Local Area Network,LAN)

拓扑结构<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191511357.png" alt="image-20230319151146324" style="zoom: 33%;" />

1. 总线(Bus)
2. 树
3. 环
4. 星

## 令牌环

token ring,一种局域网协议

### repeater

> repeater细节可略过

每个`repeater`通过单向传输链路与另外`repeater`连接

数据从一个`repeater`逐位传输到下一个`repeater`

`repeater`做数据插入 接收 删除 中继

1. 监听模式
2. 传输模式
3. 经过模式

### 协议工作模式

`token`绕环,每个节点等待`token`发送信息(改变`token`首位作为帧头,数据附在后面)

数据绕一圈回来,释放`token`

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191520093.png" alt="image-20230319152011950" style="zoom:33%;" />

## 以太网(ethernet)

simpler, cheaper, first widely used, kept up with speed race

### 以太网的发展

1. 物理拓扑结构:总线(bus)->星形(star)

   1. bus:所有节点在同一碰撞域
   2. star:中心交换,避免碰撞

2. 广播->交换

   1. 广播:共享信道,`CSMA/CD`

   2. 交换:主机-交换机,交换机-交换机 点到店连接

      ​	`CSMA/CD`->`self learning` & `spanning tree`

3. 帧结构不变

### 特点

面向网络层提供的服务

1. 提供不可靠服务:接收方不发`ACK`和`NACKS`(不管有没有通过`CRC`校验,仅做丢弃),依赖上层
2. 提供无连接服务:没有握手

### 以太网帧结构

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191532842.png" alt="image-20230319153214816" style="zoom:33%;" />

> ----8bytes-----6bytes-----6bytes---2bytes--46~1500bytes----4bytes-------

1. 前同步码(preamble) 8bytes
   1. 前7个字节均为`10101010`,用于时钟同步
   2. 最后1个字节为`10101011`,用于表示帧的开始
2. MAC地址 6bytes
3. 类型字段 2bytes 复用高层网络协议
4. 数据字段 46-1500bytes
5. CRC 4bytes

#### 前同步码

链路层确定每个帧的开始和结束位置,使用哨兵位同步

prob:哨兵位出现在帧内

solu:比特填充(bit stuffing)



#### MAC地址

网络适配器有关

扁平化空间(48比特)

网络适配器(网络接口)独一无二,硬编码(对比IP)

##### MAC地址发现协议

对于A->B的发送,A只知道自己的MAC地址

1. 自己的IP地址?
2. B的IP地址?(if B remote)
3. B的MAC地址?(if B local)
4. 第一跳路由器地址?
5. ...

链路层地址发现协议

1. `ARP` 地址解析协议(address resolution protocol)
2. `DHCP` 动态主机配置协议

核心思想

1. 广播
2. 缓存
3. soft state:过时遗忘,更新->稳健

###### DHCP

主机使用`DHCP`来发现自己的IP地址,子网掩码,局域网DNS名服务器IP地址,默认第一跳路由IP地址

基本过程

1. 客户在子网里**广播**`DHCP-discover`消息

2. 每个服务器可能回复`DHCP-offer`消息

3. 客户选择一个服务器,**广播**`DHCP-request`(包含服务器的IP)消息

4. 被选择的服务器提供一个绑定,和`DHCP-ACK`一起发送

   客户根据`DHCP-ACK`设置自己的配置参数

5. 客户通过`DHCP-release`放弃自己的绑定

6. 绑定会自动过期,需要更新

Soft state under failure(鲁棒性?)

1. 如果客户挂掉,绑定会自动过期
2. 如果一个服务器挂掉,更新时就会找到新的服务器/没有`ACK`
3. 链路(某对server-client之间)挂掉,找其他的

###### ARP

MAC地址解析协议,已知目标的IP地址,获得其MAC地址(链路层&网络层协议)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191907374.png" alt="image-20230319190736965" style="zoom:33%;" />

local过程

1. 每个主机维护一个`ARP表`(IP->MAC映射)(自动配置,即插即用)
2. 发送方
   1. 查表,若无
   2. `ARP request`广播帧--插入<sender IP, sender MAC, destination IP>
   3. 缓存接收方(会过时)
3. 接收方
   1. 对比IP,若相同
   2. `ARP reply`标准帧--插入<destination IP, destination MAC>
   3. 缓存发送方

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191911108.png" alt="image-20230319191110843" style="zoom:33%;" />

remote过程(每台主机仅有一个IP地址和一个适配器(MAC地址);路由器每个接口都有一个IP地址和一个适配器)

1. 帧中MAC地址填第一跳路由地址
2. 怎么知道目标IP不在本地->子网掩码(by DHCP)
3. 怎么知道第一跳路由MAC->DHCP

对比

1. `ARP` IP->MAC,仅在局域网,同一个子网

2. `DNS` 域名->IP,在因特网的任何地方

   >e.g. baidu.com -> its ip

#### 类型字段

知道上层用的什么协议,如`ARP`,`IP`

## 无线LAN

在[无线网](#无线网)中详细介绍

# 多路访问控制(MAC)

位置:`链路层服务-链路接入-广播协调控制`

多路访问控制(multiple access control)

期望的特性

1. 当仅有一个节点发送时,有$R\ bps$的吞吐量
2. 当$M$个节点发送时,有$R/M\ bps$的吞吐量(平均)
3. 分散的(distributed)
   1. 没有特殊节点协调控制
   2. 没有时钟 时隙同步
4. 简单

回顾上面介绍的三类协议`Channel Partitioning`,`Taking turns`,`Random Access`

## 信道划分(Channel Partitioning)

### TDMA

时分多路复用(time division multiple access,TDMA)

未使用的时隙被闲置

### FDMA

频分多路复用(frequency division multiple access,FDMA)

未使用的频段被闲置

### CDMA

码分多路复用(code division multiple access,CDMA)

用于无线广播信道,共享频段

编码-解码(利用正交,可以同时发射)

## 轮流(Taking turns)

### 轮询(polling)

主节点挨个询问其他节点是否要发数据

concerns

1. 轮询开销
2. 延迟
3. 主节点单点故障

> e.g.蓝牙

### 令牌(token passing)

concerns

1. 令牌的开销
2. 延迟
3. 令牌单点故障

> e.g.令牌环

## 随机接入(Random Access)

节点享受满速率传输(full channel data rate $R$)

没有提前的节点协调

协议专注于

1. 检测/避免 碰撞
2. 从碰撞中恢复

### ALOHA

1. 发送方

   1. 有帧时,发送

   2. 有`ACK`,nice!

      反之,以概率$p$重发,概率$1-p$等待

   3. 重发后依然没有`ACK`,摆烂(give up)

2. 接收方

   如果ok(通过检验),就发`ACK`

帧可能损坏,由于噪音或者碰撞(同时传输or有重叠)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191611187.png" alt="image-20230319161130904" style="zoom:33%;" />

### slotted ALOHA

做出下列假设

1. 帧大小相同,传输时间也相同($L$)
2. 时间被划分为时隙($L/R$)
3. 节点仅在时隙起点传输帧(同步的)
4. 如果在一个时隙中有碰撞,则所有节点在时隙结束之前检测到(帧之间要么不碰撞,要么完全碰撞)

传输

1. 新帧,下一个时隙立即传输
2. 碰撞,在每个时隙以概率$p$重传直到成功

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191615521.png" alt="image-20230319161553386" style="zoom:33%;" />

### CSMA

载波侦听多路访问(carrier sense multiple access,CSMA)

基本思想:在传输前检测信道是否空闲

1. 空闲,传输
2. 不空闲,抑制传输

由于传播时延(propagation delay),不能避免碰撞,只能减少碰撞

每次碰撞导致整个传输时间被浪费

碰撞概率取决于距离和传播时延

#### 不同种类

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191629832.png" alt="image-20230319162917624" style="zoom:33%;" />

##### 非持续的

Nonpersistent CSMA

1. 空闲,传输
2. 不空闲,等待一段随机时间再检测

谦让的协议,会导致信道更多的空闲(都在等随机时间)

##### 1-持续的

1-persistent CSMA

1. 空闲,传输
2. 不空闲,一直听直到空闲立即传输

自私的协议,如果有2个及以上的站点在等的话,一定会碰撞

> 对比CSMA/CA

##### p-持续的

p-Persistent CSMA

尝试折衷

1. 减少碰撞
2. 减少信道空闲

规则

1. 空闲,概率$p$传输,概率$1-p$等一个`tiem unit`
2. 不空闲,等到空闲执行step1
3. 等完`time unit`,执行step1

> `time unit`= 最大传播时延

$p$的取值:节点多,$p$小,等的时间多

一般会带来很大的等待延迟

#### CSMA/CD

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191642614.png" alt="image-20230319164220009" style="zoom:33%;" />

CD,collision detection

碰撞发生在传播时,于是边听边传

1. 如果空闲,传输;否则step2

2. 如果不空闲,一直等空闲,等到立即传输(1-持续)

3. 检测到碰撞,发送阻塞信号(jam signal),然后终止

   指数随机退避,再step1

> jam signal:全1信号,加强冲突,使其他设备易于检测
>
> 32比特 or 48bit

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191634999.png" alt="image-20230319163407959" style="zoom: 33%;" />

##### 传播距离与帧长

为了保证一定能在传输时检测到碰撞,有一定的限制

B-带宽

L-链路长度

V-传播速度

S-帧长

则$S\geq 2LB/V$

##### 指数随机退避

在1-持续中的算法

Binary Exponential Backoff

在经历了$n$次碰撞后,从{$0,1,2,..,2^n-1$}中随机选一个数$K$,等待$k\cdot 512 $比特时间

更多的细节

1. 10次碰撞后,$n$保持不变
2. 16次碰撞后,不管了

## 性能评价

性能检测指标(Performance Metric)

1. 利用率(Media Utilization): $U=\frac{帧传输时间(transmission)}{帧总时间}$
2. 相对传播时间(Relative Propagation Time): $a=\frac{传播时间(propagation)}{传输时间(transmission)}$

将从以下几个网络进行分析

1. 无竞争的
   1. 点到点
   2. 环
2. 随机接入
   1. ALOHA
   2. slotted ALOHA
   3. CSMA/CD

在正规化下分析,传输时间为$1$,传播时间为$a$

### Point-to-Point Link with No ACK

不管是`large frame`还是`small frame`,$max-U=\frac{1}{1+a}$(Max Utilization)

1. $a<1$,large frame
2. $a>1$,small frame

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191654440.png" alt="image-20230319165415970" style="zoom: 33%;" />

### Ring

释放令牌 = 传输完所有数据(传输时间) and 收到第一个比特(传播时间)

帧总时间 = max{传输,传播} + 释放令牌到令牌被下一个站点接收的平均时间

$total\ time=max\{1,a\}+a/N$($N$为站点个数)

$max-U=1/(max\{1,a\}+a/N)$

1. $a<1$,large frame,$U=\frac{1}{(1+a/N)}$
2. $a>1$,small frame,$U=\frac{1}{(a+a/N)}$

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191701595.png" alt="image-20230319170117439" style="zoom: 33%;" />

### slotted ALOHA

#### 概率分析

假设$N$个节点有很多帧要传输,每个传输概率为$p$

1. 成功传输(有其仅有1个):$A=Np(1-p)^{N-1}$
2. 不成功的(没有人传or两个及以上传输)

当$p=\frac{1}{N}$时,$A$最大为$(1-\frac{1}{N})^{N-1}$

#### 成功传输的利用率

$U_s=\frac{1}{1+2a}\approx 1(a\ll 1)$

> 关于a
>
> 1. Before data transmission, it takes a to detect collision
>
>    检测碰撞
>
> 2. After transmission, it takes a to make sure the transmission of the last bit
>
>    保证最后1比特的传输

#### 总利用率

$U=U_s\times A\approx (1-\frac{1}{N})^{N-1}$

当$N \rightarrow \infty$

$U\approx e^{-1}=0.367879$

### Pure ALOHA

成功概率$A=N\times P(单个节点在当前传输)\times P(大家在之前不传输导致重叠) \times P(没有其他节点在之后不传输导致重叠)$

$U\approx A=Np\cdot (1-p)^{2N-1}$

当$p=\frac{1}{2N}$时,最大$U\approx \frac{1}{2}(1-\frac{1}{2N})^{2N-1}\approx \frac{1}{2e}=0.183940$

### CSMA/CD

考虑p-坚持,2进制指数退避的情况

假设大家都有帧要发

令$contention\ interval= 2a$,碰撞发生时整个时隙的间隔

成功发送概率$A=Np(1-p)^{N-1}\rightarrow (1-\frac{1}{N})^{N-1}$

经过$j$个竞争时隙才发送成功一次的概率(几何分布)$Pr(j)=(1-A)^{j}\cdot A$

竞争时隙个数的期望$E[j]=\sum_{j=1}^{\infty}jPr(j)=\frac{1-A}{A}$

$total\ time =transmission + propagation + average\ contention\ interval$

$total\ time=1+a+2aE[j]$

$max-U=\frac{1}{1+a\frac{2-A}{A}}$

当$N\rightarrow \infty$,$A\rightarrow e^{-1}$,$U\approx \frac{1}{1+4.44a}$



# 局域网互联技术(以及连接到广域网)

单个局域网(LAN)的扩张

局域网与广域网(WAN)的连接

## 集线器(hub)

物理层设备,作用于比特而非帧,当有主机发送,`hub`重复信号给所有其他连接

星形结构的中心

每个节点与`hub`有两条线连接,传输和接收

Physically star, logically bus

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191953574.png" alt="image-20230319195314479" style="zoom: 33%;" />

## 网桥(bridge)

### 期望

1. 存储转发(Store and Forward)

   > 所谓存储,是指发送方的速率可能暂时超过接收接口的链路容量,于是网桥/交换机提供暂时缓存

2. 透明(Transparent)

3. 即插即用(Plug-and-play),自学习(self-learning)--不需要配置

### 架构

使用MAC地址

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191920650.png" alt="image-20230319192015531" style="zoom: 33%;" />

### 工作机制

#### 广播

发送者在局域网上广播发送,网桥广播到另一个局域网上->即插即用,不需要配置

#### 广播风暴

> 由于网络拓扑的设计和连接问题，或其他原因导致广播在网段内大量复制，传播数据帧，导致广播数据充斥网络无法处理，并占用大量网络带宽，导致正常业务不能运行，甚至彻底瘫痪

拓扑结构有环时会发生广播风暴->去环

#### 生成树(spanning tree)协议

不需要配置,分布式,自愈

算法总过程

1. 选一个根

   大家广播id(MAC addr),选择id最小的网桥为根

   其他网桥使用到根的最短路上的边

2. 计算最短路

   把不需要的边关闭即可

   多个最短路->选择具有更小id的邻近网桥的路

分布式地对于每一个节点

1. 广播(Y,d,X)[消息从X发出,选择Y为根(最初为自己),X到Y距离为d]
2. 选择接收到的id最小的Y作为根
3. 选边(通过最短路)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191933771.png" alt="image-20230319193326615" style="zoom: 33%;" />

后续

1. 根定时发出"我还活着"的消息
2. 如果没收到"根还活着"的消息,重新生成树

> 因为在树上,如果根还活着但是某个非叶节点挂掉了,那么后续的节点接收不到"根还活着"的消息,也会重新生成树
>
> 即也能处理有节点挂掉的情况
>
> 如果是叶节点挂掉则不影响

#### 信息洪泛(flooding)

在生成树上,网桥向所有端口转发报文而不是目标端口(我不知道往哪个端口发啊)

虽然解决了广播风暴的问题,但依然有些浪费

solu:自学习

#### 地址学习

address learning, self-learning

1. 维护`地址-端口-时间`表

   包到达,学习源主机的`地址-端口-时间`

2. 超时过期(aging time)

过程

1. 帧到达

2. 搜索目标`地址-端口`

   1. 若有,则只转发该端口(如果没有阻塞`blocking`)

   2. 特别地,如果来自到达端口,则丢弃

   3. 否则转发所有其他端口

3. 学习

> blocking:用于形成生成树

## 2层路由器(layer-2 switch)

1. 连接主机和局域网(更多的连接 a lot of v.s. 2-4 in bridge)

2. 网桥功能

3. 无碰撞(collision free),**全双工(full duplex)**

   每个链路本身是一个自己的碰撞域

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303191953895.png" alt="image-20230319195335710" style="zoom: 33%;" />

### 好处

1. No change to attached stations to convert bus/hub LAN to switched LAN

   For Ethernet LAN, each station uses Ethernet MAC protocol

2. Station has dedicated capacity equal to original LAN

   Assuming switch has sufficient capacity to keep up with all stations

3. 容易扩充

### 类似地

广播风暴->生成树

信息洪泛->自学习

## 3层路由器(layer-3 switch)

站点数量增多,广播开销大

1. layer-2 switch使大家都在同一个广播下
2. 在flooding时,layer-2 switch沦为集线器
3. IP导致许多广播(ARP,DHCP,IGMP...)

layer-2 switch使用生成树->可靠性和性能收到限制

加上`路由`功能(router functions),将局域网改造成路由连接多个子网,广播仅在子网内进行

# 无线网

两件事

1. 无线链路(wireless)
2. 移动性(mobility)
   1. 非无线链路可能也需要处理移动性
   2. 切换(handoff):切换基站

## 无线网组成元素

1. 无线主机(wireless hosts)

   1. 运行应用
   2. 固定(stationary) or 移动(mobile)

2. 无线链路(wireless link)

   多路访问控制(multiple access protocol)

3. 基站(base station)

   1. 与有线网络连接
   2. 作为有线网络和无线设备的中继
   3. 如蜂窝塔(cell tower)/`AP`

## 模式

1. 基础设施模式(Infrastructure mode):基站中继
   1. handoff
2. 临时模式/自组织模式(Ad-hoc mode):无线主机自组织通信
   1. 点对点交流
   2. 彼此路由(所有主机都做路由的工作)

## 无线链路特点

与有线链路不同的

1. 信号衰减(Decreased signal strength):有线链路中信号几乎没有衰减

   > FSPL(自由空间路径损耗,Free Space Path Loss)--$d^2$
   >
   > 反射、衍射、吸收、地形轮廓（城市、农村、植被）、湿度

   > SNR(信噪比,Signal-to-noise ratio)---BER(Bit error rate)
   >
   > SNR源于初始信号的退化和环境噪音的结合,较大的SNR使接收方更容易提取信息
   >
   > tradeoff 
   >
   > 1. 给定物理层,功率越大,SNR越高,BER越低(消耗更多能量,更干扰别人传输)
   > 2. 给定SNR,更低的传输速率,更小的BER(选择合适的)
   > 3. 由于移动性,SNR会动态变化,物理层调制技术动态选择
   >
   > <img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303202258897.png" alt="image-20230320225800435" style="zoom: 25%;" />

2. 多径效应(Multipath propagation)

   无线电经过反射折射等,经过不同路径先后到达接收方,形成自我干扰(Self-interference)

3. 通信干扰(Interference from other sources)

   共享标准化的无线网络频率,不同的网络形成干扰;环境中的电磁噪声

其他

4. 广播媒介(Broadcast medium):任何人都可以听到和干扰

5. 半双工(Half-duplex):不能同时传输和接收

   > 现在有全双工的无线网了,本课程不考虑

6. 误码率高
7. 多路访问碰撞
8. 隐藏终端问题(Hidden terminal problem)

### 多路访问问题

1. 隐藏终端(Hidden terminal):可能有位置的终端造成碰撞而不知道
2. 信号衰减(Signal fading):同上
3. 暴露终端(Exposed terminal):不会碰撞但是以为要碰撞

#### 4-frame exchange

solu to hidden terminal prob

1. 源发送`RTS`(Request to Send)
2. 目的主机回应`CTS`(Clear to Send)表示可以传
3. 收到`RTS`后,传`data`
4. 收到`data`后回`ACK`[`SIFS`]

通过抑制传输来避免碰撞

1. `RTS`警告其他设备i am busy
2. `CTS`告诉其他设备i am free now

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192049931.png" alt="image-20230319204913434" style="zoom:33%;" />

可以不用`RTS`和`CTS`,但是使用`ACK`--链路层确认重传方案

> RTS&CTS
>
> 1. 缓解问题,短的控制帧碰撞消耗小
> 2. 利于长data帧的传输
> 3. 引入了时延及消耗了信道资源

> 链路层确认重传(ARQ)方案,主要由于无线信道相对较高的误比特率

## 配置图

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192024065.png" alt="image-20230319202421804" style="zoom: 33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192024347.png" alt="image-20230319202429019" style="zoom: 33%;" />

## 802.11架构(Architecture)

1. `station`站点,链路层或者物理层设备
2. `Access Point(AP)`接入点/基站,通过无线媒介提供分配系统(`DS`)的访问
3. `Basic Service Set(BSS)`基本服务集,一个基站(`AP`)提供的服务区
4. `Extended Service Set(ESS)`
   1. 许多`BSS`通过`DS`互联
   2. 形成单一逻辑局域网(single logical LAN)
   3. Portals (routers)提供因特网访问
5. `Distribution System(DS)`
   1. 可以是一个交换机/有线网/无线网
   2. 互联形成`ESS`

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192033802.png" alt="image-20230319203355992" style="zoom: 50%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192034152.png" alt="image-20230319203421924" style="zoom:33%;" />

通信场景

1. `BSS`内
   1. 直接通信
   2. 通过`AP`
2. 跨`BSS` `AP`-`DS`-`AP`
3. 与广域网通信 `AP`-`DS`-网关

## 主机与AP连接

### 信道划分

2.4G 频率划分出11个信道(部分重叠,相近频率->通信干扰)

`AP`管理员选择合适的频率和一个服务集标识符(service set identifier,SSID)

### 连接(associate)

找一个`AP`相连--创建一个虚拟链路

1. 扫描(scan)11个信道
   1. passive scanning
      1. `AP`定期发送信标帧(beacon frame),包含`AP`的`SSID`和`MAC`
   2. active scanning
      1. 广播探测帧(Probe Request frame)
      2. `AP`回应探测相应帧(Probe response frame)
2. 选择(select) 选一个`AP`
   1. 主机-关联请求帧(Association Request)
   2. `AP`-关联相应帧(Association Response)

3. 认证(authentication) 可能会有认证
4. 配置(DHCP)

## 802.11多路访问控制协议

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192050073.png" alt="image-20230319205036677" style="zoom: 33%;" />

1. `DCF`(distributedcoordination function),分布式协调功能

   用于传输异步数据,优先级最低

2. `PCF`(point coordination function),点协调功能,集中式控制

   用于发送实时数据,优先级仅次于控制帧

### 3层优先级(3-level Priority)

1. `SIFS`(Short Inter Frame Space)

   最短帧间隔,最高优先级

   管理 控制帧(ACK,RTS,LLC,PDU,Poll,CTS...)

2. `PIFS`(point coordination function IFS)

   中等长度

   `PCF`集中式轮询帧

3. `DIFS`(distributed coordination function IFS)

   最长帧间隔

   争夺访问权的随机异步帧

控制逻辑

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192056058.png" alt="image-20230319205625744" style="zoom:33%;" />

### PCF

集中式轮询/点对点协调(centralized polling master/point coordinator)

1. 用`PIFS`来发布轮询
2. 在进行轮询和接收回应时可以做到介质隔离和异步流量锁定(seize medium and lock out all asynchronous traffic)

适合实时性要求比较高(time-sensitive)的服务为

1. round-robin
2. `SIFS`回应轮询

### DCF

使用协议`CSMA/CA`(collision avoidance)

#### 为什么不用 collision detection

1. 由于信号衰减,信号很弱,很难在传输的同时监听
2. 无法分辨噪音
3. 由于隐藏终端问题和和信号衰减问题,不能检测到所有的碰撞
4. 用`ACK`表示成功传输就不用`CD`了

#### CA过程

sender

1. 信道空闲`DIFS`,传输,等`ACK`
2. 不空闲
   1. 随机退避(二进制指数退避)
   2. 空闲时计数,信道忙时暂停计数
   3. 计数完成再发(只可能在信道空闲时)
3. 如果没有`ACK`,更大的指数退避,重复2
4. 如果有`ACK`,要发送下一帧,则进行重置的随机退避再发

receiver

接收到并且ok则在`SIFS`后发`ACK`

### 超级帧(super frame)

前一段`PCF`无竞争地轮询,后一段`DCF`允许异步通信量争用接入

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192115258.png" alt="image-20230319211518893" style="zoom:33%;" />

### 协议帧结构(802.11)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192125770.png" alt="image-20230319212543326" style="zoom:50%;" />

1. addr1:接收方/`AP`的MAC地址
2. addr2:传输方/`AP`的MAC地址
3. addr3:连接路由端口的`AP`的MAC地址
4. addr4:仅在`ad-hoc mode`使用

> 为什么需要addr3 -> `AP`是链路层设备,看MAC,不看IP -> 过程
>
> 802.3以太网帧与802.11帧的转换

> 5. 序号(seq control):接收方区分新传输的帧和以前帧的重传
> 6. 持续期(duration):用于`RTS/CTS`信道预约
> 7. 帧控制(frame control)

### 802.11 同一子网的移动性

为了增加无线局域网的物理范围,在同一个`IP`子网中部署多个`BSS`,无线设备如何切换`BSS`

是否保持`IP`,如何处理交换机

### 802.11 Rate adaptation

基站、移动端在移动过程中信噪比(`SNR`)发送变化,自适应地选择合适的物理层调制技术(速率适应),来获得合适的速率和`BER`

### 802.11 功率控制

睡眠和唤醒状态

节省能源

## 蜂窝网络(Cellular Network)

### 组成元素

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202303192134446.png" alt="image-20230319213430208" style="zoom:33%;" />

### 共享信道

1. combined FDMA/TDMA
2. CDMA

### 发展

1. 2G(voice)

2. 3G(voice+data)

3. 4G

   更快 更可靠 更高性能 轻松漫游 低成本

4. 5G

> 划重点
>
> CRC, token ring, 
