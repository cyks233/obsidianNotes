概念：将三层路由映射成二层交换路径的交换方式
全称多协议标签交换技术
# MPLS网络架构
## 设备类型
- **LSR**：标签交换路由器，运行了MPLS的设备，路由器和三层交换机，构成MPLS域（domain），通常称为MPLS/IP骨干网，属于所有用户的共享的底层公网，用户可以构建自己的私网（MPLS VPN）
- **LER**：标签边缘路由器，MPLS的边界设备
- **核心LSR**
 - **LSP**：标签交换路径，IP报文在MPLS域内转发的路径，一条LSP可以看作一条MPLS隧道，专用于一类IP报文的传输
>ingress（入LSR，上游节点)；transit（中间节点）；egress（出LSR，下游节点）
>LSP是单向的，所以需要一条相反的LSP，ingress和egress互换



## FEC (转发等价类)
ip---MPLS（LER），根据目的IP找到对应的FIB（转发信息表）
tunnel id 不等于0，需要进行MPLS标签转发；等于0，进行IP路由转发
 - ingress起到<font style="color:yellow">压入标签</font>的作用,在二层协议报头和IP报头之间插入一个本地设备为该LSP分配的MPLS标签（L1 (lable 1)对应的出接口和下一跳传输给下游的LSR）
- transit起到<font style="color:yellow">交换标签</font>的作用（L1标签换成L2标签，L2传输给下游的LSR） ^e9d833
- egress起到<font style="color:yellow">移除标签</font>的作用（移除L2标签，还原成IP报文，进行转发）

## MPLS标签
特点：短而定长（开销很小）；本地有效
整数形式的数字标识符
唯一标识一个分组所属的分类（类似于IP路由中的tag标记）
一个FEC中的分组数据在同一台设备上都将以等价（相同）的方式进行处理，分配相同的MPLS标签，即具有相同特性的报文归纳为同一个FEC。特性包括SIP, DIP, SPort, Dport, VPN实例, Qos策略等因素。
通常是根据目的IP地址基于IP路由进行划分
比如采用同一条路由的所有报文就是一个转发等价类
每台MPLS设备上，每个FEC于MPLS标签之间都有一个映射关系，针对同一个FEC，每台设备可以分配相同或不同的标签
[[MPLS#^e9d833|标签交换]]：MPLS报文从本地设备发出时，用本地设备为某FEC分配的出标签（也就是下游节点给该FEC分配的入标签）替换报文中原来携带的本地为该设备分配的入标签（上游节点的出标签）。（链路上不会变标签）
<font style="color:#de1c31">MPLS标签分发的方向和LSP方向是相反的</font>；MPLS标签最初是由目的FEC对应的egree节点分配的，这时分配的标签也是egree设备上的入标签，然后再有该egress设备向其上游节点进行通告（发送标签映射消息），上游节点收到该通告后会把它当作本地该FEC的出标签，以此类推。
上游节点的出标签于本地设备的入标签必须相同，每台设备需要为每个FEC分配一个标签（入标签）；ingress节点可以不分配
### MPLS标签格式
- Lable：20bit，标签值字段，真正MPLS标签取值的部分
- EXP：3bit，标识MPLS报文的优先级，0-7。值越小，优先级越低，用于阻塞优先发送优先级高的报文
- S：1bit，栈底标识，为了识别哪个标签是MPLS报文的最后一个标签（剥离时）；为1时代表时栈底标签，其他层都为0；当栈底标签被剥离，意味着报文中不再携带MPLS标签
- TTL：8bit，用于防环，限制MPLS报文传输的距离；当TTL值为0时，报文不能被传输。初始值可能是255，也可能是从IP报文中复制TTL
### 标签分发
#### 静态LSP
管理员手工配置
  LSP是单向的，需要两台静态LSP
  1. 利用IP路由进行FEC划分，同时需要用IP路由确保下一跳可达
  2. LSR上配置LSR-id，使能全局和接口的MPLS能力
     ```
		mpls lsr-id 1.1.1.1
		mpls
			quit
		int g0/0/0
			mpls
		```
  3. 在ingress设备上配置目的地址，下一跳，出标签；在transit设备上配置入接口，于上游设备出标签相同的入标签，对于下一跳，出标签；在egress上配置入接口，与上游设备出标签相同的入标签
     ```
		[R1]static-lsp ingress lsp1-3 destination 3.3.3.3 32 nexthop 12.1.1.2 out-label 22
		
		[R2]static-lsp transit lsp1-3 incoming-interface g0/0/0 in-label 22 nexthop 23.1.1.2 out-label 33
		
		[R3]static-lsp egress lsp1-3 incoming-interface g0/0/0 in-label 33	
		```
		![[{45F36DB0-2FF2-4A01-8019-1A3634855C2C}.png]]
#### 动态LSP
LDP，标签分发协议，自动分配
LDP, MP-BGP, RSVP可以形成动态LSP
FEC的分类，MPLS标签的分配，LSP的动态建立和维护
- 基本概念
	1. LDP对等体
			互相之间存在直接的LDP会话，可以直接使用LDP来交换标签消息（标签请求消息和标签映射消息）的两个LSR
			通过对等体之间安定LDP会话就可以获取下游对等体对于某FEC分配的mpls入标签，当作本地的出标签
			LDP对等体之间可以直连，也可以非直连，类似BGP
	2. LDP邻接体
		当一台LSR收到对端发来的hello包，两边就是邻接体关系（邻居）
		两种类型：
		1. 本地邻接体（local）：组播发送hello包（224.0.0.2，代表本地子网内所有路由器），链接hello包
		2. 远端邻接体（remote）：单播形式发送hello包，可以直连或非直连，目标hello包
	3. LDP会话
		用于在LSR之间交换标签映射，释放会话等消息
		1. 本地LDP会话：直连的本地邻接体关系
		2. 远端LDP会话：直连的本地邻接体关系，非直连的远端邻接体关系
	4. LDP会话消息
		- 发现discovery消息（UDP   S,Dport=646）：通告和维护网络中LSR的存在，hello消息
		- 会话session消息（TCP）：建立，维护和终止LDP对等体之间的会话，init初始化报文，keepalive保持活跃报文
		- 通告advertisement消息（TCP）：用于创建，改变和删除FEC的标签映射，标签映射消息
		- 通知notification消息（TCP）：用于提供建议性的消息和差错通知
		下面三个消息分主动方和被动方：
		主动方：Sport=随机，Dport=TCP 646
		被动方：Sport=TCP 646，Dport=随机
	5. LDP工作阶段
		1. LDP会话的建立：发Hello包，表明希望继续维持邻接关系；keepalive（维持LDP会话）
			LDP发现机制
			- 基本发现机制：用于之间发现直连链路上的LSR，组播
			- 扩展发现机制：用于发现非直连链路上的LSR，单播
			互相交互Hello包，为了建立TCP连接
			互相交互LDP会话初始化init信息，为了协商参数
			互相交互keepalive，为了建立LDP会话
		2. LDP LSP的建立
			将FEC和标签绑定，并且将这种绑定结果通告给上游设备的过程
			基本规则：
			要为对应的FEC分配标签
			- 入标签的分配按<font style="color:yellow">由大到小</font>的顺序分配
			- 同一个链路上游设备为FEC分配的入标签一定要与下游设备为该FEC分配的出标签一致
			- 在直接连接某FEC对应的网段（缺省仅为32位掩码的主机路由），仅仅创建一个包含入标签的LSP（无出标签，无 入/出接口）
			- 在其他节点上都会对非直连网段FEC同时创建两个LSP
			  其中一个是用于<font style="color:#de1c31">指导从本地访问FEC所代表的目的主机的LSP，仅包含出标签和出接口</font>
			  另一个是以本地节点作为中间节点（transit）的LSP，用于<font style="color:#de1c31">指导上游设备访问FEC所代表的目的主机，同时包括入标签，出标签和出接口</font>
##### LDP标签分发和管理
1. 标签分发方式
	是否要等到上游向自己主动发送请求消息后再回复
	自下游向上游进行，先通过下游设备再本端设备上为对应的FEC分配出标签，然后本段设备再为该FEC分配入标签
	- 下游自主（DU）：无需从上游LSR获得标签请求消息即可自主进行标签分配，比较简单，是华为设备默认方式
	- 下游按需（DOD）：只有收到上游LSR发送的标签请求消息后才会向上游分配
2. 标签分配控制方式
	是否要等等到下游向自己发送了某FEC的标签映射消息才为该FEC继续分配入标签，并通知
	- 独立标签分配方式：LSR在RIB中发现一个FEC，就会立马为该FEC分配标签
	- 有序标签分配方式
3. 标签保持方式
	LSR对收到的标签映射消息的处理方式
	- 自由标签保持方式：对于从邻居LSR收到的标签映射，无论邻居LSR是不是自己的下一跳都保留
	- 保守标签保持方式：只有当邻居LSR是自己的下一跳时才保留收到的LSR的标签映射

DU+Ordered+Liberal：华为默认组合方式，当LSR在收到下游标签映射后，可自主向其上游分配标签，并且收到的标签全保留
#####  LDP配置的基本步骤
1. LSR ID，尽量使用环回口，传输地址（默认使用LSR ID），后续用于建立TCP会话的通讯地址，保证地址可达
2. 全局使能MPLS
3. 全局使能MPLS LDP
4. 配置LDP会话
	- 本地LDP会话：在接口下开启MPLS和MPLS LDP
	- 远端LDP会话：主要多见于L2 VPN，在全局下设置

dis mpls ldp session all ，当 status 为 operational，成功
```
---开启mpls ldp功能---
[R1]mpls lsr-id 1.1.1.1
[R1]mpls
[R1]mpls ldp
[R1]int g0/0/0
		mpls
		mpls ldp
```
可选配置
1. 配置LDP传输地址（默认是LSR-id），接口下配置
   ```
   mpls ldp transport address [...]
	```
	![[{3F857A40-F0ED-4736-ADEF-34C3533F380E}.png]]
1. 配置LDP的会话定时器
   ```
   mpls ldp timer hello-send [...]
	```
	![[{A861E69D-D3EC-4F56-AEA1-DFE7C4E73CA1}.png]]
1. 配置PHP特性
2. 配置LDP标签分配控制方式，ldp视图下配置
   ```
   mpls ldp
	   label distribution control-mode [...]
	```
1. 配置LDP标签发布方式
   接口下配置，DU,DOD
   ```
   mpls ldp advertisement [...]
	```
	![[{4E8B45BD-1048-43B8-8256-F8C991D68C8F}.png]]
2. 配置LDP自动触发DOD请求功能
3. MPLS MTU
4. MPLS对TTL的处理
LDP报文![[{3DE0E23B-C421-4E88-9472-B0F5FFE7F864}.png]]
### MPLS特殊标签
  0-15特殊标签
  1. 0：显式空标签，表示标签立即被弹出
  2. 1：router alter label，只有出现再非栈底标签中才会生效
  3. 2：IPv6的显式空标签
  4. 3：隐式空标签，次末跳弹出，倒数第二台设备上弹出标签
  
  16-1023：静态LSP分配
  1024以上：动态LSP分配\
## MPLS工作原理
1. MPLS的标签动作
	- push 压入
	- swap 交换
	- pop 弹出（php次末跳弹出，减少最后一跳的负担，使最后一条直接进行IP路由转发或者下一次标签转发）
2. 基本概念
	- tunnel-id：32bit，本地有效，为了给隧道的上层应用提供一个统一接口
	- NHLFE：下一跳标签转发表项，用于指导mpls的转发，包括tunnel-id，出接口，下一跳，出标签，标签操作类型等
	- FTN：FEC与NHLFE的映射，查看fib表中tunnel id部位0x0的转发表项，只在ingress上存在，因为只需要再ingress上用到FEC的分类信息来查找对应压入的标签，后面的设备只需要按照标签交换进行转发即可
	- ILM：入标签映射，入标签和NHLFE的映射，使得本地的入标签，出标签和tunnel id建立一个关联；在transit上的作用是将入标签和NHLFE绑定
3. MPLS对TTL值的处理
	1. uniform 统一
		默认处理模式，在离开mpls域时，TTL减少的是实际经过的mpls的 设备的跳数
		复制IP报头的TTL到MPLS报文中，把每台mpls网络中的每台设备当作IP网络中的单跳来处理
	2. pipe 管道
		IP报文中的TTL字段会忽略所经过的MPLS网络的中间节点，将其看作两端边缘节点通过一个管道的直连；无论在mpls网络中经过多少台三层设备，从mpls域入节点到出节点之间，TTL只减1
		出于安全考虑，需要隐藏MPLS骨干网络结构时可以使用
## MPLS体系结构
![[{BF27A637-A1F5-4224-B317-B976B6652D8C}.png]]
控制平面：控制协议报文的转发，IP路由和MPLS标签
IP路由协议---RIB
LDP---------LIB：标签和FEC的对应关系
<font style="color:#de1c31">所有的入标签必须不同，每个FEC对应的入标签必须唯一，同一个FEC对应的入标签必须一致，不管有多少个上游路径</font>
对于下一跳也相同的相同路由，出标签必须一致
对于下一跳不同的路由，出标签必须不同
转发平面：
FIB-----指导IP报文转发，是从RIB中提取的
LFIB----标签转发信息表，指导MPLS报文转发，从LIB表中提取的
例：RIB:192.168.10.0/24,出接口:nexthop
	控制平面
	LDP：入标签 1088，出标签1012
	LIB：192.168.10.0/24   1088 | 1012
	转发平面
	FIB：192.168.10.0/24   nexthop
	LFIB：192.168.10.0/24   1088 | 1012  nexthop
# MPLS VPN
[[VPN#MPLS-VPN配置|MPLS VPN配置>>>]]
MPLS VPN设备：
- P：骨干核心设备，只需要具备MPLS功能即可
- PE：骨干边缘设备，对VPN的操作都是发送在PE上，对性能要求高
- CE：客户边缘设备，不感知VPN存在的，也不需要支持MPLS

BGP/MPLS IP VPN 概念：使用BGP在ISP骨干网中发布用户的私网VPN路由，并且在骨干网中传递转发VPN路由
1. BGP部分：MP-BGP，多协议BGP，在骨干网中发布私网IPv4路由
	1. P节点通过 BGP 确保PE和PE之间能够完成私网路由传递
	2. MP-BGP支持多种地址族，如IPv4单播，IPv4组播，vpnv4(vpn-ipv4)，IPv6单播 等
2. MPLS：起到隧道的作用，按照标签转发
3. ip：隧道里需要承载的是IP报文
4. vpn：属于vpn中的一种解决方案

VPN实例（VRF）
VPN路由转发表，每一个VPN实例互相隔离，也与公网路由表相互独立
不同site的VPN实例本地有效，但名字必须唯一。
- RD：路由标识符，Route Distinguisher，使不同VPN实例可以地址重叠
  长8个字节，用于区分使用相同地址空间的IPv4前缀
  PE从CE接收到IPv4路由后，在IPv4前缀前加上RD，转换为<font style="color:yellow">全局唯一</font>的<font style="color:yellow">VPN-IPv4</font>路由
- RT：分为import RT 和 export RT，影响路由的进入

HUB（中心），SPOKE（中心）：HUB对SPOKE进行一个集中控制

MPLS VPN的特点：
- 扩展强，只需要修改提供该站点业务的边缘节点的配置
## MPLS VPN路由交互
### MP-BGP
MP-BGP是BGP-4的多协议扩展，采用不同的网络层协议，既可以支持传统的IPv4地址族，又可以支持其他地址族
MP-BGP新增两种路径属性：
1. MP_REACH_NLRI：多协议可达NLRI，用于发布可达路由及下一跳信息
2. MP_UNREACH_NLRI：多协议不可达NLRI，用于撤销不可达路由
### 路由传递
- MP-BGP将VPNv4传递到远端PE之后，远端PE需要将VPNv4路由导入正确的VPN实例
- MPLS VPN使用32位的BGP扩展团体属性 RT 来控制VPN路由信息的发布与接收
- 本地PE在发布VPNv4路由前附上RT属性，对端RT在接到VPNv4路由后根据RT将路由导入对应的VPN实例
- PE根据VPNv4路由所携带的RT将路由导入正确的VPN实例后，VPNv4路由的RD值剥离，将IPv4路由通告给相应的客户的CE设备
- 站点B和站点D的CE设备就能学到去往各自远端站点的路由。也可以实现同一用户（同一VPN）不同站点之间的路由互通

## MPLS VPN报文转发
![[{102AF2C7-B3E6-4D69-9C7C-CF6249B7B399}.png]]
1. CE1发送一个目的地址为NET1的IPv4报文
2. PE2收到报文后进行MPLS标签的封装，先封装VPN标签V1，再封装外层标签T2，然后将此报文发送给P
3. P进行标签交换，把外层标签T2换成T1，然后将此报文发送给PE1
4. PE1收到后去掉所有标签，将IPv4报文转发给CE1.
## MPLS VPN跨域
跨域场景下，VPN的工作原理不变，但是会产生以下问题：
1. AS之间不会运行LDP协议，因此AS之间无法建立外层隧道
2. PE之间没有运行IGP协议，缺省情况下无法建立BGP邻居关系，进而无法传递VPNv4路由

对于以上问题，有三种解决方案：OptionA,B,C
### OptionA
 ASBR之间交换IPv4路由，采用IPv4数据包转发数据。
 VRF to VRF，两端设备互为PE和CE
![[{91CC1CF5-B336-4386-B159-78F30F6DC8B1}.png]]
AS内跑OSPF和BGP，配置MPLS，和MP-BGP
PE上创建VPN实例。R4和R5在实例里peer对端的地址，VPN实例不需要一样
两端都需要import对应的RT值

```
[R4]bgp 100
	ipv4-family vpn-instance vpn111 
		peer 45.1.1.2 as-number 200 

ip vpn-instance vpn111
	route-distinguisher 200:1
	vpn-target 200:1 export-extcommunity
	vpn-target 200:1 100:1 300:1 import-extcommunity

```
### OptionB
ASBR之间交换VPNv4路由，采用携带一层MPLS标签的方式转发数据包
单跳EBGP
- ASBR之间要使能MPLS（识别并传递私网标签）
- ASBR上要`undo policy vpn-target`，不用RT的方式转发路由

`apply-label per-nexthop `：让ASBR按下一跳为VPNv4路由分配标签（起到节约标签的作用）
- ASBR会为具有相同路由下一跳和出标签的路由分配一个标签
- 缺省情况下是ASBR只在向其他MP-BGP对等体发布VPNv4路由时，为每条路由分配一个标签
在经过ASBR时会剥离公网标签，只剩私网标签
在AS内同时拥有公网和私网标签![[{38E8659F-1C06-4180-936E-D212772E1063}.png]]
到了ASBR之间就只有一层私网标签![[{877439F2-ED51-47E8-B5CE-658A206269C3}.png]]

### OptionC
PE之间构建多跳EBGP邻居，直接传递VPNv4路由，采用携带多层MPLS标签的方式转发数据包
#### OptionC方式一
>ASBR之间通过BGP发布各自的PE环回口路由
>ASBR之间不维护VPNv4路由,靠PE设备维护
##### 方式一配置思路
1. 各AS内MPLS骨干网上配置接口IP和路由协议，实现PE和ASBR-PE的互通
2. 各AS内启用MPLS和MPLS LDP，建立MPLS LSP
3. PE和ASBR-PE之间建立MP-IBGP邻居
4. 各AS内的PE要和CE建立VPN实例，并且将接口绑定进VPN实例
5. PE和CE之间的基于实例的路由协议（比如BGP）
6. 不同AS的PE之间建立多跳MP-EBGP邻居，在PE和ASBR上使能标签IPv4路由交换能力
7. 在ASBR上配置路由策略，相对端ASBR发布的BGP路由，为其分配私网路由标签，向本端PE设备发布的路由，如果是带标签的，那么就分配新的私网路由标签 
##### 方式一配置过程
拓扑![[{A4D2CE8A-E532-4B10-8FD7-D8EC1CF6726E}.png]]

```
// 前5步基础配置，打通AS内的路由
// 两边AS的PE互相建立多跳EBGP邻居
bgp 100
	peer 77.1.1.1 as 200
	peer 77.1.1.1 connect-interface LoopBack0
	peer 77.1.1.1 ebgp-max-hop 255       //ebgp多跳
	peer 77.1.1.1 label-route-capability //使能标签路由转发
	ipv4-family vpnv4
		peer 77.1.1.1 enable
		peer 44.1.1.1 enable             //PE与ASBR之间建立MP-IBGP邻居
	network 22.1.1.1 32                  //ASBR上net从PE上学到的路由，一定是32位的
// ASBR上建立ebgp邻居
bgp 100
	peer 45.1.1.2 as 200
	peer 45.1.1.2 label-route-capability
// 创建路由策略，并应用在ASBR上
route-policy p1 permit node 10           //匹配全部路由，分配新的标签
	apply mpls-label
route-policy p2 permit node 10           //如果路由过来时有标签，则分配新的标签
	if-math mpls-label                   
	apply mpls-label
bgp 100
	peer 45.1.1.2 route-policy p1 export
	peer 22.1.1.1 route-policy p2 export
```

R2,G0/0/1上的标签: | 1025 | 1029 | 1026 | icmp | ![[{A10BDE99-7E36-4096-829C-2F6943AA13A1}.png]]
R4,G0/0/1上的标签: | 1029 | 1026 | icmp |![[{87D4953F-6BBC-47E7-9ABB-93B0E1BE095B}.png]]
R7,G0/0/0上的标签: | 1026 | icmp |![[{0359CF01-90CE-466E-98DB-A875F03B9C04}.png]]
#### OptionC方式二
>PE和ASBR之间不用配置IBGP邻居关系
>当ASBR从对端的ASBR学到对端AS内的带标签的BGP公网路由后
>LDP通过在ASBR上将**BGP路由引入到IGP内**,为这些路由分配标签
>会触发建立跨域的LDP LSP

方式一需要三层标签, 方式二只需要两层标签
##### 方式二配置思路
1. 各AS内MPLS骨干网上配置接口IP和路由协议，实现PE和ASBR-PE的互通
2. 各AS内启用MPLS和MPLS LDP，建立MPLS LSP
3. 各AS内的PE要和CE建立VPN实例，并且将接口绑定进VPN实例
4. PE和CE之间的基于实例的路由协议（比如BGP）
5. 将域内PE的路由发布给对端PE,先配置ASBR之间的MP-EBGP邻居,然后再本端ASBR上通过BGP将域内PE的路由发布给对端ASBR,再远端ASBR上将BGP路由引入到骨干网的IGP路由表中
6. 在ASBR上配置路由策略, 对于向对端ASBR发布的路由,分配MPLS标签,使能ASBR之间的标签路由交换能力
7. 在ASBR上配置为带标签的公网BGP路由建立LDP LSP
8. 在不同AS间的PE上建立多跳MP-EBGP邻居, 并设置最大跳数
##### 方式二配置过程
```
//基础配置与C1差别不大
//PE和ASBR之间不需要建立BGP邻居
bgp 100
	undo peer 44.1.1.1
//ASBR上,在OSPF中引入BGP
ospf 1
	import-route bgp
//在ASBR上,为带标签的公网BGP路由建立LDP LSP
mpls
	lsp-trigger bgp-label-route
```

标签: | 1026 | 1029 | icmp |![[{8C45BC45-37AE-4EE6-9989-0E03553465C2}.png]]

标签: | 1024 | 1029 | icmp |![[{86B657D2-2925-41E5-98EF-083AFBD59D8B}.png]]

标签: | 1029 | icmp |![[{031763C8-B72F-4316-9169-F10DA92FD323}.png]]