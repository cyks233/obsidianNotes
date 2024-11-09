名字：开放式最短路径优先
OSPF是一种LS型协议
使用SPF算法，基于TCP/IP协议开发的 ^a4dbf3
#### OSPF的区域
区域分为骨干区域和非骨干区域，所有非骨干区域都要与骨干区域相连，所以也称为花瓣型协议
区域的划分以接口为单位


#### 链路状态信息和ospf交互报文类型
![[Pasted image 20240701160911.png]]
以组播形式发送Hello包，周期性发送
![[Pasted image 20240701161325.png]]
A发给B，B会认为A是init状态；B发回A，A认为B是2-way状态；A再发给B，B认为A是2-way状态
两边现在是**邻居**关系 ^157870

![[Pasted image 20240701161046.png]]
exstart状态，互相交互空的DD报文
![[Pasted image 20240701191253.png]]
![[Pasted image 20240701191353.png]]
exchange状态，大范围交互DD报文，报文中携带信息 ^93bfc5

![[Pasted image 20240701192022.png]]
![[Pasted image 20240701192035.png]]
当交换DD报文时发现自己的链路状态数据库缺少某些LSA，就会发送LSR报文，用来请求这些缺失的LSA；同时会进入loading状态。
![[Pasted image 20240701192615.png]]
LSU中存储了缺失的LSA。在对端收到LSR报文后，就会发送LSU，包含了请求的LSA。
![[Pasted image 20240701192844.png]]
LSACK用来确认收到了LSU中的LSA信息 ^3e71f2

#### 路由器优先级
DR:指定路由器
BDR:备份指定路由器
DRother:其他路由器
优先级默认是1，取值范围0-255；优选优先级大的，数值越大越优
如果优先级设为0，则丧失选举权，直接成为DRother
其次选择R-ID大的
DR不具备抢占，DR挂了，BDR才成为DR
存在邻居和邻接两种关系，可以减少邻接数量，减少LSA的交互次数

#### OSPF认证
1. 邻居认证
   对邻居关系的认证，认证不通过无法建立邻居
2. 接口(链路)认证
   在某个特定接口下开启认证
3. 区域认证
   将属于这个区域的所有接口都开启认证
4. 虚链路认证
   在做区域0认证后必须做的认证
#### 接口类型
##### Broadcast(MA)
Hello时间：10s
死亡时间：40s
以组播形式建立邻居，存在DR和BDR，路由掩码正常
##### P2P(点到点)
Hello时间：10s
死亡时间：40s
以组播形式建立邻居，<font style="color:#de1c31">不存在DR和BDR</font>，路由掩码正常
##### NBMA(非广播型以太网)
Hello时间：30s
死亡时间：120s
以<font style="color:#de1c31">单播</font>形式建立邻居，存在DR和BDR，路由掩码正常
##### P2MP(点到多点)
Hello时间：30s
死亡时间：120s
以组播形式建立邻居，<font style="color:#de1c31">不存在DR和BDR</font>，路由掩码<font style="color:#de1c31">都是32位</font>

|           | Broadcast        | P2P              | NBMA             | P2MP             |
| --------- | ---------------- | ---------------- | ---------------- | ---------------- |
| Broadcast | Full<br>路由正常学习   | Full<br>路由不能正常学习 | 邻居失败<br>路由不能正常学习 | 邻居失败<br>路由不能正常学习 |
| P2P       | Full<br>路由不能正常学习 | Full<br>路由正常学习   | 邻居失败<br>路由不能正常学习 | 邻居失败<br>路由不能正常学习 |
| NBMA      | 邻居失败<br>路由不能正常学习 | 邻居失败<br>路由不能正常学习 | Full<br>路由正常学习   | Full<br>路由不能正常学习 |
| P2MP      | 邻居失败<br>路由不能正常学习 | 邻居失败<br>路由不能正常学习 | Full<br>路由不能正常学习 | Full<br>路由正常学习   |
hello时间一致时邻居可以达成Full状态；接口类型一致时，路由才能正常学习

#### LSA类型
ipv4环境共有6种LSA
LSA不是独立的报文，需要LSU报文来承载
![[Pasted image 20240703151900.png]]
LSDB表
![[Pasted image 20240703152422.png]]
![[Pasted image 20240703152428.png]]
![[Pasted image 20240703152434.png]]
![[Pasted image 20240703152442.png]]
![[Pasted image 20240703152415.png]]

#### OSPF路由
OSPF路由分为域内路由，域间路由，域外路由
域内路由(O)包括1类和2类LSA，路由协议为OSPF，路由优先级为10
域间路由(OIA)包括3类LSA，路由协议为OSPF
域外路由(OE)包括5类LSA，路由协议为O_ASE，路由优先级为150

#### OSPF特殊区域
末节区域，用来过滤远端传过来的LSA，精简路由表
	stub：过滤4类和5类LSA，同时产生3类LSA的默认路由
	total stub：在ABR上配置，过滤3,4,5类LSA，同时生成3类LSA的默认路由
	stub的缺点是无法进行路由引入
	NSSA：过滤4类和5类LSA，同时产生7类LSA的默认路由
	total NSSA：在ABR上配置，过滤3,4,5类LSA，同时产生7类LSA的默认路由
	NSSA可以进行路由引入 ^fa10cb

#### OSPF开销计算
cost叠加是计算<font style="color:yellow">路由流向的入方向，数据流向的出方向</font>

#### OSPF优化
1. 静默(slient)接口：只宣告路由，不发送OSPF报文，可以在环回口和接pc的接口上配置
2. 路由汇总，目的是精简路由表
   手动汇总：只能进行域间和域外汇总，不支持域内汇总
   域间汇总在路由所在的ABR上做，域外汇总在路由引入的ASBR上做
   OSPF不支持自动汇总
3. 下发默认路由
   ```
   default-route-advertise [always]
    ```
    没有always，表示必须要本设备存在静态默认路由才能下发5类默认路由
    有always，表示强制下发，即使本设备不存在静态默认路由