# Overview

体系结构

1. CS(客户-服务器)
   1. 客户开始对话
   2. 服务器永远在线, IP周知
2. P2P
   1. 对等
   2. 高度可扩展
   3. 难以管理



# DNS

Domain Name Service 域名解析系统 = DNS服务器+DNS协议(默认运行在UDP)

将 域名(主机名) 映射到 IP地址

##### 层次结构

1. 根DNS服务器: 提供TLD服务器IP地址
2. 顶级域(TLD)服务器: 提供权威DNS服务器地址
3. 权威DNS服务器: 提供公共可访问的DNS记录
4. 本地DNS服务器: 代理 缓存(DHCP)

##### 迭代查询

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202306072049537.png" alt="image-20230607204917742" style="zoom: 25%;" />

##### 特点

1. 主机名独特
2. 可扩展
3. 分布式 自主管理
4. 高度可用
5. 快查询

# 电子邮件

1. local user agent -> local SMTP server (`SMTP`)
2. local server -> remote SMTP server (`SMTP`)
3. remote user agent -- mailbox on remote server (`POP3`, `IMAP4`, `HTTP`)

## SMTP

TCP握手

持续连接(比如两个server之间有很多mail)

只支持7-bit ASCII(历史遗留)

## Mail access protocol

1. POP3: 跨会话无状态
2. IMAP: 允许管理远程文件夹, 追踪文件状态
3. HTTP

# FTP

File Transfer Protocol

不同的OS和fs

对远程文件系统进行访问控制

# HTTP

对象与URL

报文头为文本形式,而非字段编码

底层使用TCP,三次握手(最后一次握手附带HTTP请求)

## 状态

默认是无状态的(stateless)

+ server容易实现
+ 负载小
+ 顺序无关

使用cookies实现状态(隐私泄露)

+ 使得server可以得到user的很多信息
+ 搜索引擎获得信息

## 性能

1. 用户
   1. 快下载
   2. 高可用
2. 提供服务者
   1. 快乐的用户
   2. 低成本
3. 网络
   1. 避免过载, 保持畅通

### HTTP协议优化

基于文本(HTTP/1.1) -> 二进制(HTTP/2)

并行?

非持续连接 -> 持续连接的

+ 不用多次建立 关闭TCP连接
+ 比起只有两个RTT的TCP, 更长的连接使得TCP机制接入(如拥塞控制)

非持续连接, 一次时间 $Total=2RTT+Transmission$

###### 小文件

+ One-at-a-time: ~2n RTT
+ m 并行: ~2[n/m] RTT
+ 持续连接: ~ (n+1)RTT
+ 流水线: ~2 RTT
+ 流水线/持续连接: ~2 RTT first time, RTT later

###### 大文件 size=F

+ One-at-a-time: ~ nF/B
+ m 并行: ~ [n/m] F/B
+ 流水线/持续连接: ~ nF/B

此时只能增加带宽

### 缓存

过期机制与last modified

1. 客户本地
2. 代理服务器
3. CDN

# CDN

Content Distribution Networks 内容分发网络

大文件从origin分发到离client近的server

分布式服务器就近提供内容

##### 技术支持

1. DNS: 一个域名 定位到就近的服务器
2. Routing
3. URL重定向: 重定向到认证服务器
4. 负载均衡 网络时延