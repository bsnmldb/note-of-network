提纲:

重点:

RSA

Diffie-Hellman Key Exchange

可靠的邮件加密

# 密码学初步

## 安全要求

1. 可用性(availability) -> interruption
2. 机密性(confidentiality) -> interception
3. 完整性(integrity) -> modification
4. 身份鉴别(authenticity) -> fabrication



## 网络攻击

### 攻击种类

##### 正常情况

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311710959.png" alt="image-20230531171028851" style="zoom:50%;" />

##### interruption

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311710471.png" alt="image-20230531171002389" style="zoom:50%;" />

被中间中断了 -> 可用性

##### interception

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311710942.png" alt="image-20230531171050045" style="zoom:50%;" />

窃听

内容泄露 流量分析 -> 机密性

##### modification

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311710781.png" alt="image-20230531171010861" style="zoom:50%;" />

内容修改 -> 完整性

##### fabrication

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311719711.png" alt="image-20230531171057355" style="zoom:50%;" />

虚构伪装 -> 身份鉴别

### 消极-积极

##### passive attack

窃听 流量分析

难以检测但是易于预防

##### active attack

伪装 重放(replay) 修改 终止服务

难以预防但是易于检测

## 安全处理

### 1. 加密

1. 对称加密系统
2. 不对称加密系统

#### 加密要求

##### a. 强加密算法

即使知道一些(明文,密文)pair,也不能因此得到解密算法

##### b. 安全密钥分配

#### 加密攻击

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311728096.png" alt="image-20230531172830170" style="zoom:50%;" />

### 2. 报文鉴别码(MAC)

### 3. 数字签名

## 传统加密技术

### 替换算法

substitution

#### 凯撒密码

环移位 模运算

只有26种可能

暴力破解

#### 单字母密码

单表查询替换 映射

$N!(26!)$种可能

字母出现频率破解

#### 维吉尼亚密码

有密钥,使用凯撒密码,密钥每一位都表示当前使用的凯撒密码

以密钥长度为一个周期进行交叉循环的凯撒密码替换

破解: 先找到密钥长度(循环周期),再单个暴力破解

### 置换算法

transposition

改变顺序

#### 篱笆密码

将明文按列优先写,然后行优先得到密文

<img src="C:/Users/bsnmldb/Documents/Tencent%20Files/806679225/Image/C2C/Image1/2CB269E839938DFA29525FBA3A6E61BC.jpg" alt="img" style="zoom: 25%;" />

破解关键在于多少行,容易破解

#### 行列密码

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311756663.png" alt="image-20230531175603452" style="zoom: 50%;" />

明文按照行优先写,然后按照密钥的顺序,按列优先得到密文

如上先找标1的列: cman

如果已知一个(明文,密文)pair就很好破解

## 现代加密算法

块密码: 将明文分为固定大小的块,分别加密成同样大小的密文块

### 对称密钥系统

常用算法:

1. DES(data encryption standard)
2. TDES(triple DES)
3. AES(advanced encryption standard)

##### DES

在现代计算机下,DES已经失效

##### TDES

慢 块大小小

##### AES

对称密钥

强度更高,效率更高

参数可选(块大小 密钥长度)

### 非对称密钥系统

一对密钥

1. 公钥(public key): $K^+$
2. 私钥(private key): $K^-$

加密可以使用任一钥匙,但是解密必须用另外一个

##### 系统性质

加密解密算法大家都知道

给定公钥和算法也很难找到私钥

很难解密公钥加密的密文

给定(明文,密文)pair和公钥,很难找到私钥

#### RSA

信息都是一串比特流,将其整体看作一个数字

选择两个很大的素数(约$2^{500}-2^{1024}$)$p$和$q$

$N=p\times q$

$\Phi=(p-1)\times (q-1)$,这是欧拉函数,表示小于$N$且与$N$互素的整数个数

随便找一个素数$e$,s.t. $gcd(e,\Phi)=1$与欧拉函数互素

然后计算$e$在模运算下的逆$d$,$d\times e\ mod\ \Phi=1$

公钥:$(e,N)$

私钥:$(d,N)$

> 欧拉定理: 任意数$a$的$i$次方除以$N$的余数的周期为$\Phi(N)$
>
> 即对于$i$,一定有$a^i\ mod\ N=a^{i+\Phi}\ mod\ N$
>
> 于是对于任意数$m$,$m^1\ mod\ N=m^{1+k\Phi}\ mod\ N=m^{ed}\ mod\ N$
>
> $(m^e\ mod\ N)^d\ mod\ N=m^{ed}\ mod\ N= m\ mod\ N$
>
> 其中$(m^e\ mod\ N)$为密文

加密: 给定明文$m<N$,计算密文$C=m^e\mod\ N$

解密: 给定密文$C<N$,计算明文$m=C^d\ mod\ N$

##### 分析

1. $p$和$q$应该大约长度相同
2. $p$和$q$应该无关
3. $e$应该比较小(不给加密增加负担)
4. $d$应该很大(防止暴力破解)

RSA的强度依赖于大素数构造很容易,但是分解很困难

但实际解密运算代价高(破解代价更高)

##### 签名

用自己的密钥加密块$M$得到$S$

将$S$和$M$一起发送,别人能用我的公钥解到$K^+(S)=M$就能证明是我的签名

##### 会话密钥

在建立会话时通过RSA传输对称密钥

然后使用对称密钥通信

# Authentication & Integrity

## 困难场景

直接认证--直接伪造

ip认证--带ip伪造

加密认证--重放攻击

nonce加密认证--对称密钥的分发

nonce非对称加密认证--中间人攻击



<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311926522.png" alt="image-20230531192632639" style="zoom: 33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311928195.png" alt="image-20230531192829153" style="zoom:33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311929621.png" alt="image-20230531192914643" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311930082.png" alt="image-20230531193023266" style="zoom:33%;" /><img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311932995.png" alt="image-20230531193207304" style="zoom:33%;" />

> `nonce`一个始终只使用一次的数字

##### 中间人攻击

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311933247.png" alt="image-20230531193313317" style="zoom:50%;" />

两边伪装,难以检测

`Bob`以为`Trudy`是`Alice`

`Alice`以为`Trudy`是`Bob`

##### 目的

确认这个报文来自`Alice`

1. 欺骗(spoofing)
2. 修改(modification)
3. 重放(replay)

##### MAC function

MAC message authentication code 报文鉴别码

固定大小,追加在报文后面

种类

1. 加密算法: 如CBC-MAC
2. 哈希算法: 有密钥, 如: MD5,SHA-1

要求

1. 适用于任意长度
2. 输出MAC固定长度
3. 计算简单
4. one-way: 给定鉴别码$Y$,难以找到内容$X$,s.t. $Y=MAC(X)$
5. weak-collision-resistance: 给定内容$X_1$,难以找到内容$X_2$,s.t. $MAC(X_1)=MAC(X_2)$
6. strong-collision-resistance: 难以找到内容$X_1,X_2$,s.t. $MAC(X_1)=MAC(X_2)$

##### digital signature

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305311946882.png" alt="image-20230531194650943" style="zoom: 50%;" />

哈希报文内容,再用私钥加密哈希值

# key distribution

$Alice$和$Bob$怎么共享对称密钥

$Alice$怎么确认这个公钥就是$Bob$的呢

#### Diffie-Hellman key exchange

全世界都知道的一个大素数$P$和一个generator $g\in Z^*_P$

模$P$互质同余群$Z^*_P=\{1\leq a\leq P-1|gcd(a,P)=1\}$

> 特别地,对于素数$P,Z^*_P=\{1,2,..,P-1\}$

`A`随机选一个$X\in Z^*_P$,给`B`发送$g^X\ mod\ P$

`B`随机选一个$Y\in Z^*_P$,给`A`发送$g^Y\ mod\ P$

`A`计算密钥$(g^Y)^X\ mod\ P=g^{XY}\ mod\ P$

`B`计算密钥$(g^X)^Y\ mod\ P=g^{XY}\ mod\ P$

#### CA

trusted certification authority

确定双方的有效性,提供会话密钥

1. 会话密钥(session key): 单次逻辑连接使用
2. 永久密钥(permanent key): 用于会话密钥分发

security service module (SSM)

##### Needham-Schroder Protocol

`A B`均与`KDC`(密钥分发中心)共享其密钥$K_{AS},K_{BS}$

1. `A`->`KDC`: $id_A,id_B,N_A$

   `A`创建随机数$N_A$,随同自己和想联系的人的$id$发给`KDC`

2. `KDC`->`A`: $K_{AS}(N_A,K_{AB},id_B,K_{BS}(K_{AB},id_A))$

   `KDC`产生一个随机密钥$K_{AB}$并进行加密

3. `A`->`B`: $K_{BS}(K_{AB},A)$

   `A`发送从`KDC`收到的证明自己的部分以及会话密钥的信息

4. `B`->`A`: $K_{AB}(N_B)$

   `B`表示已经得到会话密钥

5. `A`->`B`: $K_{AB}(N_B-1)$

   `A`表示ok

##### one-time session key

`Bob`使用一个临时创建的`session key`加密信息

再用`Alice`的公钥加密`session key`

一起发

##### 公钥验证

public key certificate

`CA`签名证书

1. 其他人不能伪造`CA`
2. 公钥 主人id(信息等)
3. 可能会过期(timestamp)

# 安全电子邮件(应用层安全协议)

##### 机密性 

+ `Alice`: $K_S(m)+K_B^+(K_S)$
+ `Bob`: $K_S=K_B^-(K_B^+(K_S))$  $m=K_S(K_S(m))$

`Alice`产生`session key`,然后公钥加密`session key`

`Bob`先私钥解密`session key`,然后解密信息

##### 鉴别和完整性

+ `Alice`: $m+K_A^-(H(m))$
+ `Bob`: 比较$H(m)=K_A^+(K_A^-(H(m)))$

`Alice`数字签名

`Bob`根据$m$算出$H(m)$,与解密的`MAC`比较

##### 合起来

$K_S(m+K_A^-(H(m)))+K_B^+(K_S)$

`CA`验证公钥

# SSL(传输层安全协议)

1. TCP的一个强化版本

   SSL secure socket layer 安全套接字层

2. SSL的修改版本

   TLS transport layer security 运输层安全性

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312105507.png" alt="image-20230531210509465" style="zoom:50%;" />

`SSL`是一个应用层协议,提供`API`

但在研发者的角度,`SSL`是一个提供`TCP`服务的运输协议,相当于加强版`TCP`,在传输层



## SSL初步

1. 握手(handshake)
2. 密钥导出(key derivation)
3. 数据传输(data transfer)
4. SSL记录(control info)

##### 1. 握手

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312111031.png" alt="image-20230531211106155" style="zoom:50%;" />

1. `TCP`三次握手
2. `SSL`hello
3. 证书(含公钥)
4. 公钥加密的会话密钥

`MS` master secret 会话主密钥

`EMS` encrypted master secret

##### 2. 密钥导出

虽然有主密钥,但是使用不同的密钥会更加安全

1. $K_c$: `client`->`server`用的密钥
2. $M_c$: `client`->`server`的`MAC`密钥
3. $K_s$
4. $M_s$

分别用于不同的方向,报文信息和鉴别码

##### 3. 数据传输

如何加密数据流?

data record

将数据流分割成`记录`,对每个记录附加`MAC`,然后将`record+MAC`一起加密

1. 计算`MAC`: $MAC=H(M,record)$
2. 加密: $f(K,record+MAC)$

需要序号! 阻止重排序 重放等中间人攻击

$MAC=H(M,sequence,record)$

##### 4. 控制信息

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312124329.png" alt="image-20230531212434365" style="zoom:50%;" />

关闭会话:

如果仅使用`TCP`关闭会话 -> 截断攻击

使用`type`字段`SSL`关闭会话,虽然是明文,但是有`MAC`做鉴别

##### 初步结果

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312125649.png" alt="image-20230531212511616" style="zoom:50%;" />

## real SSL

协议簇

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312131927.png" alt="image-20230531213102986" style="zoom: 67%;" />

handshake protocol

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312134551.png" alt="image-20230531213430100" style="zoom: 67%;" />

鉴别双方

协商加密算法 `MAC`算法 密钥

4 round

1. 建立`SSL connection`
2. 服务器鉴别和密钥交换
3. 客户端鉴别和密钥交换
4. 确认 共识

# IPsec(网络层安全)

IP Security IP安全

VPN virtual private network 虚拟专用网

##### 形式

1. 运输模式(transport mode): 端系统软件支持

   端到端加密传输

2. 隧道模式(tunnel mode): 数据再封装

   路由器/防火墙加密传输

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312148431.png" alt="image-20230531214809572" style="zoom:50%;" />

##### 协议簇

1. AH 鉴别首部 or ESP 封装安全性载荷 (ESP有加密,AH仅保证完整性)
2. IKE 密钥交换

> 下面主要介绍 `隧道+ESP`

##### SA

security association 安全关联

单向 参数索引表

使用信息进行通信加密

##### 数据报

<img src="https://raw.githubusercontent.com/bsnmldb/tuchuang/main/img/202305312154974.png" alt="image-20230531215457832" style="zoom:50%;" />

##### IKE

因特网密钥交换协议

自动交换 删除 修改`SA`

`PSK`or`PKI`

# LAN安全(链路层)

## WEP

wired equivalent privacy 有线等效保密

早期协议,有安全漏洞,现在已经fail

手工设置共享密钥

1. 密码验证

   > 不重数防止重放攻击

2. 简单加密通信

## 802.11i 加强

服务器认证与`AP`分离

# 防火墙

功能

1. 防止拒绝服务攻击(DoS)
2. 只允许授权的接入(认证)
3. 防止防止恶意访问和篡改

实施方式

1. 无状态分组过滤
2. 有状态分组过滤
3. 应用层网关

问题

1. IP欺骗: 路由器不知道这个IP是不是真的,影响过滤
2. 网关强大但是不透明,需要手动设置
3. 不能很好地处理`UDP`,无状态
4. 总是事后才会设置新的规则
5. tradeoff: 安全-自由

##### packet filtering

单个`packet`视角

只依据`ip` `TCP/UDP port` `icmp信息` `TCP SYN/ACK` 等

##### ACL

Access Control List 访问控制列表

(白名单/黑名单)

##### stateful packet filtering

跟踪每条`TCP`连接,建立维护其状态信息

如`SYN`建立,`FIN`结束,timeout释放

##### stateful ACL

每个流维护信息和规则

维护成本高

##### application gateway

所有的流量都要经过服务器转发,顺便做过滤和审查

代理服务器: 应用软件层面做,但是流量有限

