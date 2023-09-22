主要内容

1. 多媒体网络应用
2. 实时要求的传输协议
3. 服务质量

<font color=red>重点:桶</font>

# 多媒体网络应用

## 媒体介质

1. 视频
   1. 高比特率
   2. 视频压缩
   3. 质量等级(multiple version)
2. 音频
   1. 离散采样 量化 编码解码

## 应用类型

1. streaming, stored 流式存储
2. conversational 会话式
3. streaming, live 流式实时

> streaming: 可以在下载完整个文件之前开始播放
>
> stored(at server)
>
> conversational: 人类交互 时延敏感 容忍丢包

## 流式存储视频

###### 流程

服务器发送视频块(不同视频块时间间隔可以相同) -> 客户接收视频块(时间间隔取决于网络) -> 视频块播放

###### 挑战

时延: 一旦用户开始播放,要求后续的视频块应该在播放到它时到达

用户的交互: 暂停 快进 倒退 跳过视频

网络问题: 丢包 重传

客户端缓冲`client-side buffer` :补偿网络延迟和抖动

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305172019884.png" alt="image-20230517201926868" style="zoom:50%;" />

###### 客户端缓存-建模

1. 播放延迟: 初始化提前缓存`buffer`
2. 开始播放
3. 恒定速率播放,可变速率接收

平均输入速率$x$,恒定输出速率$r$

1. $x<r$,`buffer`最终为空,造成视频停滞,缓存等待另一个播放延迟初始化
2. $x>r$,`buffer`不会空,只要初始播放延迟足够吸收$x$的抖动

### 流实现

#### UDP流

服务器以稳定速率发送数据

`RTP`实时传输协议

依赖应用层实现错误回复和时间协同

可能会被防火墙阻挡

#### HTTP

依赖`HTTP GET`

使用`TCP`

由于`TCP`的拥塞控制 重传等机制,填充速率波动

较大的播放延迟: 平稳的`TCP`交付率

更容易通过防火墙

#### DASH

Dynamic Adaptive Streaming over Http

###### server

1. 将视频文件分为多个块, 每个块存储，以不同的速率编码

2. 清单文件：为不同的块提供URL

###### client

1. 定期测量服务器到客户端的带宽
2. 咨询清单, 一次请求一个块
   1. 选择最大的编码速率可持续给定的当前带宽
   2. 可以在不同时间点选择不同的编码速率(取决于当时的可用带宽)

###### 客户端智能

client决定

1. 什么时候请求块
2. 请求什么质量的块
3. 向哪个服务器请求

## IP语言(VoIP)

###### 服务模型

1. 端到端时延要求

   可以对话的

2. 会话初始化(session initialization)

3. 扩展功能

4. 紧急会话

###### 网络影响

1. 丢包 network loss
2. 时延 delay loss
3. 丢包容忍 loss tolerance

##### 会话

supernode

peer to peer

# 实时会话式传输协议(应用层)



## RTP

基于`UDP`,`RTP`数据报文封装在`UDP`的数据中

用于流媒体数据,在传输层

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305172052000.png" alt="image-20230517205212100" style="zoom: 25%;" />

1. 流媒体类型(编码格式 payload type): 编码解码
2. 序列号: 丢包 恢复分组
3. 数据戳: 流媒体播放
4. 同步源标识符(SSRC): 数据来源

在`UDP`上简单封装提供流媒体传输的基本功能,封装端系统可见

提供尽力而为服务,没有提供特别的服务质量

## RTCP

RTP Control Protocol

在`RTP`会话中周期性发送

汇报,数据统计,同步信息

控制码率,检测故障

服务质量监测

## RTSP

Real Time Streaming Protocol

应用层协议

支持快进快退,跳转(交互式请求)

可以基于`UDP`或者`TCP`

操作: 建立连接,播放,暂停,关闭

## SIP

Session Initiation Protocol

发起和初始化一个会话连接,定位接收方

###### 操作

建立会话

1. 地址定位
2. 连接请求 协商编码方式

管理会话

# 服务质量(重点)

QoS Quality of Service

下面有两种服务质量保证的框架

1. ISA
2. DS 

## ISA

Integrated Services Architecture 综合服务体系

针对路由器实现,对每个流区别对待,实现相应的要求

<font color=red>资源预留</font>

### ISA Function

1. 路由算法(新的路由协议): 服务质量参数放入代价计算
2. 排队原则: 提供服务质量保障的优先队列
3. 丢包原则: 根据优先级
4. 资源预留协议(`RSVP`)
5. 接入控制: 控制哪些流可以接入,资源请求
6. 传输控制数据库
7. 管理调度

### RSVP

资源分配和资源预留协议

建立连接: 所有路由器预留资源,服务质量声明

接收方发起,逐个路由器传播,声明资源要求,每个路由器允许和预留(或者拒绝),一直到发送方

### 缺点

重新改写路由器,对路由器要求高

对每个流都要预留虚拟状态(维护软状态)

复杂

灵活性 扩展性

## DS

Differentiated Services 区分服务

分类别,提供不同的服务质量保障

网络核心简单,边缘路由器复杂

使用`IPv4/6`的包头服务质量字段

SLA: service level agreement 服务质量协议

路由器查看IP包头类别,提供区分服务

###### DS domain

有运营商提供区分服务,只有在域内才能保证服务质量

边缘路由器-内部路由器

###### DS architecture

边缘路由器: 分类 标记 整型

内部路由器: 根据标记实施调度策略

###### marking 标记

对每个报文,标记其服务类别,`IPv4/6`

###### policing 流量整型

限制流量

1. 平均速率
2. 峰值速率
3. 突发长度

###### scheduling 调度

1. FIFO

   丢包策略

   1. 丢弃尾巴 tail drop
   2. 按照优先级
   3. 随机丢弃(如随机早丢弃算法)

2. WFQ

### Leaky Bucket

恒定速率发送

1. 若没有要发的,do nothing
2. 有,恒定速率

### Token Bucket

实现

1. 桶最多$b$个令牌
2. 每秒$r$个令牌添加(满则溢出)
3. 分组向网络传输需要等令牌

优点

1. 最大突发长度$b$
2. 最大速率$(rt+b)/t$

### WFQ

Weighted Fair Queuing

权重比例分配带宽

### 服务质量保障实现

token bucket + WFQ

实验保障

