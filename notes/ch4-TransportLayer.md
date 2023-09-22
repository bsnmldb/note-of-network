# 传输层基础

给在不同终端上面的两个应用提供逻辑通信的抽象

传输层协议运行在端系统当中

##### 作用

1. 提供端到端的通信(应用层面) -> 多路分解与多路复用

   (IP只提供了主机到主机的)

2. 提供更多的服务 -> 可靠通信 拥塞控制

   (IP只提供了尽力而为服务)

##### 套接字(socket)

每个进程有一个或多个套接字,相当于进程和网络之间的门户

1. 多路复用: 发送主机从不同套接字收集数据并封装首部发送
2. 多路分解: 接收主机将运输层报文段数据交付到正确的套接字

UDP套接字由二元组标识 <dst IP, dst port>

TCP套接字是一个四元组 <src IP, src port, dst IP, dst port>

> 特别地,在tcp中来自不同的源,去往相同的目的IP和端口的包会被交付到不同的套接字(服务器提供相同进程下不同的线程)
>
> 而udp会交付到相同的套接字

# 可靠设计

+ Checksums (错误检测)
+ Timers (丢包检测)
+ Acknowledgments
+ Sequence numbers (冗余, 窗口)
+ Sliding windows (效率)

## 停止-等待

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052048180.png" alt="image-20230605204448613" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052049437.png" alt="image-20230605204942655" style="zoom: 33%;" />
$$
throughput=\frac{DATA}{RTT}
$$


## pipeline

在`停止-等待`的基础上一次发多个($k$个)并停止等待

效率是上面的$k$倍
$$
throughput=\frac{k\times DATA}{RTT}
$$


## 滑动窗口

$$
throughput = min\{\frac{WinSZ\times DATA}{RTT},LinkBandwidth\}
$$

ACK方式

1. 累积ACK
2. 单个选择ACK

回退处理方式

1. 回退$M$步(适合错误率低)
2. 选择性重发(适合错误率高)

# UDP

user datagram protocol

最小(简单)传输层协议

1. 尽力而为服务:
   1. 可能丢包
   2. 不保证按序交付
2. 无连接的:
   1. 没有握手
   2. 每个UDP包独立

很快(不用建立连接),包头小,简单,不限速(没有拥塞控制)

##### 包头

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052107444.png" alt="image-20230605210731548" style="zoom: 50%;" />

> octet 字节

##### 检验和

包头+数据+伪包头

以2字节为单位,会`回卷`, 检验和为和取反

> checksum=0表示不使用检验和
>
> src port可选, 用于需要回信

# TCP

transmission control protocol

1. 可靠
2. 按需交付
3. 字节流

## 包头

##### TCP segment

IP packet 最大1500字节(`MTU`)

一般IP头=TCP头=20字节

`MSS `= `MTU`-IPheader-TCPheader = 1460

Maximum Segment Size TCP最大数据



<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052133539.png" alt="image-20230605213259653" style="zoom: 33%;" />

1. 端口号: 用于多路分解和多路复用

2. seq no: <font color=red>字节流偏移量而非数据报序号</font>

3. checksum: 包头+数据+伪包头, 必须

4. ack: 下一个期望的seq(累计确认机制)

   重复多个同一seq的ack表示后面陆续收到了 -> 快重传

5. HdrLen: 单位是word(4个字节),一般为5(20字节)

6. flags: SYN, ACK, RST, FIN

## 快重传

收到3个冗余ACK,立即快重传ACK指定的包

将3个冗余ACK看作(单独)丢包的信号

## 超时

如果超时未收到ACK,重传第一个包

+ 估计时间太短: 把成功的包误认为丢包
+ 估计时间太长: 吞吐量低

$$
EstimatedRTT = (1-\alpha)EstimatedRTT + \alpha SampleRTT
$$

### Karn/Partridge algorithm

不使用重传的样本RTT

$\alpha=0.125$

$RTO=2RTT$,一旦超时重传,$RTO=2RTO$(指数回避网络拥塞),直到新的有效的$RTT$到达,$RTO=2RTT$

### Jacobson/Karelsalgorithm

引入偏移量的度量

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052201132.png" alt="image-20230605220100167" style="zoom:33%;" />

## 建立连接

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052208294.png" alt="image-20230605220843395" style="zoom:33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052208060.png" alt="image-20230605220858149" style="zoom:33%;" />

三次握手

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052214985.png" alt="image-20230605221414199" style="zoom: 50%;" />

- **第一次握手**：
  客户端将TCP报文**标志位SYN置为1**，随机产生一个序号值seq=x，保存在TCP首部的序列号(Sequence Number)字段里，指明客户端打算连接的服务器的端口，并将该数据包发送给服务器端，发送完毕后，客户端进入`SYN_SENT`状态，等待服务器端确认。

- **第二次握手**：
  服务器端收到数据包后由标志位SYN=1知道客户端请求建立连接，服务器端将TCP报文**标志位SYN和ACK都置为1**，ack=x+1，随机产生一个序号值seq=y，并将该数据包发送给客户端以确认连接请求，服务器端进入`SYN_RCVD`状态。

- **第三次握手**：
  客户端收到确认后，检查ack是否为x+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=y+1，并将该数据包发送给服务器端，服务器端检查ack是否为y+1，ACK是否为1，如果正确则连接建立成功，客户端和服务器端进入`ESTABLISHED`状态，完成三次握手，随后客户端与服务器端之间可以开始传输数据了。

> 我们假设client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。
>
> 假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。
>
> 或者服务器被之前已经关闭的SYN影响,而不理会现在真正的SYN

## 关闭连接

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306052232480.webp" alt="img" style="zoom: 50%;" />

挥手请求可以是Client端，也可以是Server端发起的，我们假设是Client端发起：

- **第一次挥手**： Client端发起挥手请求，向Server端发送标志位是FIN报文段，设置序列号seq，此时，Client端进入`FIN_WAIT_1`状态，这表示<font color=red>Client端没有数据要发送给Server端了</font>。
- **第二次分手**：Server端收到了Client端发送的FIN报文段，向Client端返回一个标志位是ACK的报文段，ack设为seq加1，Client端进入`FIN_WAIT_2`状态，Server端告诉Client端，我确认并同意你的关闭请求。
- **第三次分手**： Server端向Client端发送标志位是FIN的报文段，请求关闭连接，同时Client端进入`LAST_ACK`状态。
- **第四次分手** ： Client端收到Server端发送的FIN报文段，向Server端发送标志位是ACK的报文段，然后Client端进入`TIME_WAIT`状态。Server端收到Client端的ACK报文段以后，就关闭连接。此时，Client端等待**2MSL**的时间后依然没有收到回复，则证明Server端已正常关闭，那好，Client端也可以关闭连接了。

> 建立连接时因为当Server端收到Client端的SYN连接请求报文后，可以直接发送**SYN+ACK**报文。其中ACK报文是用来应答的，SYN报文是用来同步的。所以建立连接只需要三次握手。
>
> 由于TCP协议是一种面向连接的、可靠的、基于字节流的运输层通信协议，TCP是**全双工模式**。这就意味着，关闭连接时，当Client端发出FIN报文段时，只是表示Client端告诉Server端数据已经发送完毕了。当Server端收到FIN报文并返回ACK报文段，表示它已经知道Client端没有数据发送了，但是Server端还是可以发送数据到Client端的，所以Server很可能并不会立即关闭SOCKET，直到Server端把数据也发送完毕。当Server端也发送了FIN报文段时，这个时候就表示Server端也没有数据要发送了，就会告诉Client端，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。

##### Time Wait

client在发送最后的ACK后不会立即关闭

因为加入这个ACK丢失,server会重传FIN,然后再重发ACK

所以最后仍需等待一段时间才会真正结束

##### 意外关闭

A对所有报文回复RST

B一旦收到RST,不用ACK,关闭连接

# 流控制

限制发送方速率**防止超过接收方的缓存**(UDP没有流控制)

滑动窗口

1. 发送方:
   1. L = 最小未ack的包
   2. R = L + WinSZ
2. 接收方:
   1. L = 期望的数据包
   2. R = L + WinSZ

rwnd = 接收方buffer - 还在缓存中未处理的包(最后接收到的-最后处理的)

通过ACK发送rwnd动态调整

发送方维护 WinSZ = rwnd $\geq$ 最后发送的 - 最后ack的

##### 死锁问题

超时机制

# <font color=red>拥塞控制</font>

$$
发送速率=\frac{WinSZ}{RTT}\\
SenderWSZ=min\{CWND,RWND\}
$$

**保证不要占有超过网络的负载**

##### 检测拥塞

1. 冗余ACK: 单独丢包
2. 超时: 更严重,更多丢包

##### 动态调整

###### 慢启动

初始化 `CWND=1`, 即初始速率为$\frac{MSS}{RTT}$

乘性增长: 每个RTT乘2 `CWND=2CWND`, 即每收到一个`ack`, `CWND+=1`

一直到`CWND = ssthresh`, 慢启动停止

> 如`ssthresh=40`, `CWND`的状态为 1 2 4 8 16 32 40 慢启动停止

###### 拥塞避免

AIMD 线性增乘性减

每个RTT加1 `CWND+=1`, 即每收到一个`ack`, `CWND+= 1/CWND`

1. **对于3冗余ACK**

   `ssthresh = CWND/2`,`CWND = ssthresh`

   继续拥塞避免

2. **对于超时事件**

   `ssthresh = CWND/2`, `CWND = 1`

   进入慢启动

> why AIMD
>
> 高效 公平(平衡点)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071021525.png" alt="image-20230607102106979" style="zoom:50%;" />

##### 快恢复(fast recovery)

1. 进入快恢复: 遇到冗余ack时

   `ssthresh = CWND/2`, `CWND = ssthresh + 3`

   并且在快恢复阶段遇到新的冗余ack, 都会将`CWND += 1`

2. 退出快恢复: 收到新的非冗余的ack

   `CWND = ssthresh`, 进入拥塞避免

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071028627.png" alt="image-20230607102856648" style="zoom:50%;" />

## 版本

1. TCP-Tahoe

   CWND=1 on 3 dupACKs

2. TCP-Reno

   CWND =1 on timeout
   CWND = CWND/2 on 3 dupACKs

3. TCP-newReno

   TCP-Reno + `fast recovery`

4. TCP-SACK

   选择性ack



## throughput

一个周期发送包
$$
period=\frac{3}{8}W^2+\frac{3}{4}W
$$
丢包率
$$
L=\frac{1}{period}
$$
平均速率
$$
v=\sqrt{\frac{3}{2}}\frac{1}{RTT\sqrt{L}}
$$

# 路由器帮忙的拥塞控制

通知拥塞 限制速率 保证公平

机制:

1. 抑制分组(choke packet)
2. 反压(backpressure)
3. 警告位(warning bit)
4. 随机早丢弃(random early discard)
5. 公平队列(FQ,WFQ)

##### 1. choke packet

拥塞节点路由器向 发送方 发送 抑制分组

(告诉发送方别发啦)

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071059066.png" alt="image-20230607105930035" style="zoom: 25%;" />

##### 2. back pressure

逐跳抑制, 传播时间 > 传输时间(拥塞节点离发送方很远)

choke packet效果不好

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071100946.png" alt="image-20230607110031019" style="zoom: 25%;" />

##### 3. warning bit

由路由器设置包头

接收方设置ack

提前预警

##### 4. random early discard

路由器在缓冲区完全满之前就随机丢弃数据包

##### 5. FQ

###### Max-Min fairness

对于几个流$r_1,r_2,..,r_k$,链路带宽为$C$,提供的带宽服务为$a_1,a_2,..,a_k$

先求$C/k$满足在此之下的所有流,然后用剩下的均分给其它未满足的流

$a_i=min(f,r_i),\quad \sum a_i=C$

###### fair queuing

引入包大小的考虑, 流量是逐位提供的

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306071111046.png" alt="image-20230607111113233" style="zoom: 50%;" />

更多的,有WFQ

###### 与FIFO比较

1. 隔离: 欺骗flow不会获利
2. 带宽: 与RTT无关
3. flow可以获得适应性的流量

但是更加复杂

并没有解决拥塞,只是保证公平