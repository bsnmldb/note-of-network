##### 组成

1. 网络边缘: 终端 (笔记本电脑 手机 智能家具 服务器)
2. 接入网络: 物理媒介 链路 (双绞线 <font color=red>无线路由器</font> 同轴电缆 光纤)
3. 网络核心: 路由器 (路由器 <font color=red>交换机</font>)

##### 电路交换、分组交换、虚电路

![image-20230614130842371](https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306141308931.png)

##### 协议栈

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306141313488.png" alt="image-20230614131158079" style="zoom: 33%;" />

从上到下是应用层 运输层 网络层 链路层 物理层

1. 应用层:存留网络应用程序和应用层协议,向应用程序提供网络服务传递报文
2. 运输层:在应用程序端点(端系统)之间传送应用层报文
3. 网络层:将数据报从一台主机移动到另一台主机
4. 链路层:把信息组织为帧,将分组从一个节点移动到路径上的下一个节点
5. 物理层:将帧的一个个比特从一个节点移动到下一个节点

##### 网络性能

丢包

吞吐量 -> 传输时延

1. 处理时延
2. 排队时延: $L=A\times W$
3. 传输时延(Transmission)
4. 传播时延(Propagation)