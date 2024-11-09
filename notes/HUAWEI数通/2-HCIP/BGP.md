#### BGP简介
BGP：边界网关协议
路由协议分为两种 IGP,EGP。IGP是内部网关协议，OSPF，ISIS等；EGP是外部网关协议，BGP
BGP通常用在大型企业，数据中心和ISP骨干网
BGP协议的本质：TCP179
#### AS
自治系统，可以划分0-65535个AS。
划分AS的原因是IGP无法承载所有的路由，以及公网要尽量避免震荡（BGP只有触发更新）
0-64512是公有AS(类似公网IP)，剩下的是私有AS(类似私网IP)
AS的划分：以路由器为单位，一个路由器只能属于一个AS
#### BGP邻居
EBGP：建立在两个不同AS之间的邻居关系
IBGP：建立在相同AS之间的邻居关系
跨设备建立BGP邻居时建议使用环回口建立，更稳定，但是要修改连接接口
默认是用物理口，建立EBGP邻居时需要修改连接跳数，因为默认是1跳(EBGP多跳)，IBGP邻居跳数设置为255
##### BGP邻居状态
1. Idle：初始状态，拒绝所有连接
2. Connect：等待TCP三次握手完成
3. Active：尝试与邻居建立TCP连接
4. Openset：发送open报文
5. Openconfirm：发送keepalive
6. Established：最终状态，邻居可以交互信息
##### BGP路由表
![[Pasted image 20240710131906.png]]
图中 * 代表下一跳可达，> 代表最优，i 代表这条路由是从IBGP邻居学到的
| -- * > 的作用是将该路由放进全局路由表，并将该路由继续传递给下一个BGP邻居
IBGP邻居学习到的路由的下一跳都是不变的，所以当有EBGP邻居时，对所有IBGP邻居修改下一跳为自己`peer 44.1.1.1 next-hop-local` 
##### BGP报文
idle到active状态，TCP三次握手![[Pasted image 20240710134323.png]]
openset状态，open报文![[Pasted image 20240710134427.png]]
opencomfirm状态，keepalive报文![[Pasted image 20240710134642.png]]
Establish状态，update报文![[Pasted image 20240711124620.png]]
#### BGP防环机制
1. EBGP防环：从一个BGP邻居收到路由后，如果该路由所携带的AS-PATH中有自己的AS，则丢弃该路由
   解决方法：`peer 44.1.1.1 allow-as-loop 1` 允许传递过来的路由中所携带的和自己本地相同的AS号出现几次（最大10次）
2. IBGP防环：水平分割，从一个IBGP邻居收到的路由不会再传递给任何一个IBGP邻居，因为IBGP邻居只能一跳
   解决方法：
   1. FULL-MESH：两两建立邻居（非常繁琐）
   2. RR反射器：有三种角色：RR反射器，RR客户端，RR非客户端
      从RR非客户端收到的路由不会反射给RR非客户端，其他情况都能反射
      `peer 23.1.1.2 reflect-client` 配置在哪台设备，那台设备就是RR反射器；命令中peer谁，谁就是RR客户端；没有peer谁，谁就是RR非客户端
    3. 联邦：将多个大AS分成小AS（相对少用）
#### BGP属性
BGP有很多属性，有不同的作用
官方分为两种：
1. 公认属性：所有厂商设备必须识别
   1. 公认必遵：必须携带在路由信息![[Pasted image 20240711130733.png]]![[Pasted image 20240711131213.png]]
      - **NextHop**：下一跳，全0代表自己产生，非0代表从邻居学习，前提是下一跳可达，选路优选自己产生的
      - **AS-Path**：描述这条路由经过的AS路径
      最左边（234）代表邻居AS，最右边（57）代表始发AS，用于EBGP防环，选路优选AS-Path中AS少的
      - **Origin**：起源属性，描述这个路由如何被宣告进BGP；i 代表network，? 代表重分布
   1. 公认任意：可以不携带
      - **Local_Pref**：本地优先级，选路优选数值大的；
        路由信息中携带的LP值会在出AS时被清除，对端AS的设备在收到该路由时会为其赋予本地LP值，默认是100，可以修改。
      - **Atomic_aggregate**：原子聚合，和路由汇总有关系
2. 可选属性：不强制所有厂商能够识别
	1. 可选传递：虽然不能识别，但是可以传递给邻居
	   - **Aggregator**：聚合者，和路由汇总有关系
	   - **Community**：社团属性，类似于一个标记，给路由赋予一个标记从而产生一些用处
		1. 标准社团属性：AA : NN tag，众所周知的属性
			   - internet：默认，可以将路由传递给邻居
			   - no-advertise：不将该路由传递给任何BGP邻居
			   - no-export：不将该路由传递出本AS
			   - no-export-subconfed：不将该路由传递出本AS（联邦中的小AS）
		2. 扩展社团属性：[[VPN#MPLS-VPN配置|MPLS VPN]]
	1. 可选不传递：不识别，也不传递
	   - **MED**：metric/cost，开销，用来选路
#### BGP选路
IGP选路看开销，等级或非等价
BGP选路有很多因素，共有13条选路规则，匹配到就停止，前提是下一跳可达
1. 优选Preference_Value/Weight值最高的路由
   PV是华为私有，Weight思科私有
   PV默认是0，取值范围0-65535，比大；本路由器有效，只能在 <font style="color:#de1c31">in</font> 方向修改
   修改方式：
   1. 全局修改：影响所有从该邻居过来的路由`peer 12.1.1.2 preferred-value 11`
   2. 精确修改：ACL（匹配路由）+ Router-policy（修改值）
		```
		acl number 2000
			rule 5 permit sourse 33.1.1.1 0
		route-policy PV permit node 10 //默认大号拒绝
			if-match acl 2000
			apply preferred-value 111
		route-policy PV permit node 9999 //大号放行其他路由
		bgp 100
			peer 12.1.1.2 route-policy PV import
		```
2. 优选本地优先级（LocPrf）最高的路由
	默认100，本AS有效， 不能做AS出方向
	LP如果显示为空，说明这条路由是从EBGP收到的，空代表本地LP值，可以修改
	```
	bgp 100
		default local-preference 200
	```
3. 优选 手动聚合 > 自动聚合 > network > import >从对等体学到的
	本地产生的优于从邻居学到的
	聚合（AGG）：
	1. 手动聚合，`aggregate 33.0.0.0 8`
		- 不会抑制明细路由（通告给邻居的明细，不是本地的明细），建议在始发设备上配置
		`aggregate 33.0.0.0 8 detail-suppressed`  抑制明细
		- 此汇总路由下一跳为null接口（黑洞路由，防环）
		- 会回传给邻居，一般建议在始发设备上做汇总
	2. 自动聚合，只能汇总重分布路由
	   ```
	   [R3-bgp]summary automatic
	   Info: Automatic summarization is valid only for the routes imported through the import-route command.
		```
4. 优选AS_Path短的路由
	经过的AS数量越少越优，人为添加的AS尽量是AS-Path已经存在的，防止EBGP防环
5. 起源类型 IGP>EGP>Incomplete
	i 优于 e 优于 ?
6. 对于来自同一AS的路由，优选MED值小的
	MED：仅限于相邻的AS，默认是0，比小
7. 优选从EBGP学来的路由（EBGP>IBGP）
	协议优先级：EBGP 20   IBGP 200
8. 优选AS内部IGP的Metric最小的路由
	路由下一跳的metric
9. 优选Cluster_List最短的路由
	RR反射的次数
10. 优选Orginator_ID最小的路由
	RR-RID
11. Router-ID最小的路由器发布的路由
12. 优选具有较小IP地址的邻居学来的路由