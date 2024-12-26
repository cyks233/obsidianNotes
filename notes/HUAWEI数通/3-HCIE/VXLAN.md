### 概念
VLAN：隔离广播域，虚拟化，是横向的；可以做堆叠，m-lag；交换机上的vlan是全局的，只有4096个
VXLAN：数据中心使用；使用vlan只能有4k个应用和租户；BD（bridge domain 桥接域），BD是本地概念
NVo3（network virtualization over layer 3）：NV，网络功能虚拟化，在一台交换机上虚拟出多个虚拟交换机为引入的租户在相同的物理设备上虚拟出不同的虚拟设备，这些设备都可以通过vxlan互通，叠加网络（虚拟化交换机 BD；虚拟化路由器 VRF；虚拟的防火墙 vFW；虚拟的负载均衡设备 LB）
VNI：24bit，有2^24个

叠加网络
顶层/上层：overlayer，NV
底层/下层，underlayer，三层的IP网络，高可用性，高并发，高吞吐，高扩展

CLOS：胖树架构，扩展性极强，全互联

Spine---leaf（多归接入spine）

NVO3：underlay 3层网络（ospf，isis）

数据中心设备连线密密麻麻的，称为织物（fabric）

TOR，连接叶子交换机（leaf），连接的路由器称为脊柱（spine，骨干），再连接的设备称为servire leaf

BD
vxlan网络中转发数据报文的二层广播域
二层广播域：将VNI以1：1的方式映射到广播域BD，BD将成为vxlan网络转发数据报文的实体

VNI：vxlan network identifier，vxlan的网络标识，类似vlan id，用于区分不同的vxlan段，不同vxlan段的虚拟机不能进行直接二层通信；一个VNI代表一个租户，即使多个终端用户属于同一个VNI，也表示是同一个租户的

分布式网关部署中，VNI分为二层VNI和三层VNI
- 二层是普通VNI，1：1映射到BD，实现vxlan同子网转发
- 三层VNI与VPN实例关联，用于跨子网通信
- VBDIF：基于BD创建三层逻辑接口，可以配置地址实现不同网络通信（类似vlanif）

NVE
network virtual edge 网络虚拟边缘节点，实现网络虚拟化的实体
报文经过NVE封装转换后，NVE之间就可以基于三层基础网络建立二层虚拟化网络
基于NVE部署位置的不同，分为三种模式
1. 硬件模式：NVE都部署在支持NVE的设备上，所有vxlan报文的封装与解封装都是在设备上运行（CE12800）
2. 软件模式：NVE部署在Vswitch上，所有vxlan报文的封装与解封装都是Vswitch发生的
3. 混合模式

VTEP（vxlan tunnel endpoints）
vxlan隧道的端点（地址，一般用环回口地址，前提是路由可达）
一对VTEP（源+目的）地址就对应了一个vxlan隧道

VAP（virtual access point 虚拟接入点）
vxlan下行业务接入点，可以是二层子接口或者vlan
当接入点是二层子接口时，需要设置二层子接口的流封装模式实现不同的接口接入不同的数据报文，将二层子接口关联广播域BD后，可实现数据报文通过BD转发
报文流封装模式：
1. dot1q
   上来的报文携带一层vlan标签，这种类型值接受与指定vlan tag匹配的报文
   限制：二层子接口封装的VID，不能与对应的二层主接口（物理口）允许通过的vlan相同；二层接口的VID不能与三层子接口封装的VID相同，同一个二层主接口下的子接口不能重叠VID
1. untag
   允许接口接收所有报文，不区分报文中是否携带了vlan tag
   限制：必须确保对应的主接口没有加入vlan--trunk或者hybrid，undo allow vlan 1；仅支持二层物理口创建default二层子接口；一个主接口仅允许创建一个default类型的二层子接口，创建后将不允许再创建其他类型的二层子接口

当接入点是vlan，只需要将vlan绑定到BD中，就可以通过BD转发

### 同网段静态vxlan
#### 原理
1. PC1 ping 100.1.1.2
   与运算，ICMP echo-reque | SIP: 1.1 DIP: 1.2 | SMAC 1.1 DMAC ?
   查ARP，发送ARP，request | SMAC: 1.1 DMAC: 全F（广播包） | sender ip：1.1 sendermac：1.1
   target ip：1.3，target mac：全F
2. SW1的g0/0/3口收到一个untag报文，收，打上vlan id 10，向属于vlan10的接口泛洪
	- 学习源mac，构建mac地址表
	- 打vlanid 10
	- 向g0/0/1口泛洪
	- g0/0/1是trunk口，所以报文保持tag 10向leaf1发送
3. leaf1
   leaf1通过VAP口（g1/0/1.10二层子接口）收到这个广播帧，发现过来的tag是10，本身自己接口封装模式是dot1q，关联vid是10，所以这个报文会收，并且将该报文的tag剥离，如果封装是defalut，则保留tag
   进入BD域内（广播域/交换机）
	- 基于源mac学习，构建BD100的mac地址表
	     一个是从VAP收到的客户mac地址
	     一个是从对端VTEP收到的对端客户的mac地址
	- 转发，NVE隧道里去
	  头端复制表项![[{B3043AEF-CA00-4654-A168-F6869373769F}.png]]
	  经过NVE封装为vxlan报文，转发到三层的underlay网络
	  vxlan报文的最外层的SIP是11.1.1.1，DIP是33.1.1.1，leaf1，查路由表，ospf
4. leaf3，收到以太帧（DMAC：leaf3的接口mac，DIP：33.1.1.1）
	检查DMAC，发现是自己，收，解封装发现IP（自己）和UDP
	然后检查UDP的DPort是否是4789，如果是，则交给vxlan处理，检查VNI，发现VNI是10，VNI和BD是1：1，就知道给哪个BD域了
#### 配置
![[{72583752-7F0D-4D66-BEF1-1980785435C3}.png]]
配置思路：
1. Core IGP：ospf，打通leaf1和leaf3的环回口通信
   正常配置，CE交换机支持切换3层口配置IP，打完命令需要commit，否则配置不生效
2. overlay网络，用户侧
   1. 配置BD：BD本地有效，可以不一样，但是vni全局有效，必须配置成一样的才能进行二层通信
	   ```
		[CE1] bridge-domain 100
				vxlan vni 10
				commit
		[CE3] bridge-domain 200
				vxlan vni 10
				commit
		```
	2. 配置VAP
	   ```
		//CE1
		int g1/0/1
			undo shutdown
		int g1/0/1.10 mode l2
			encapsulation dot1q vid 10
			bridge-domain 100
			commit
		//CE3配置untag
		int g1/0/1.10 mode l2
			encapsulation untag
			bridge-domain 200
		```
	3. 配置NVE
		```
		int nve 1
			source 11.1.1.1
			//可以同时配置多条隧道
			vni 10 head-end peer-list 33.1.1.1 [22.1.1.1]
		//CE3上反向写一条
		interface nve1
			source 33.1.1.1
			vni 10 head-end peer-list 11.1.1.1
		```

![[{950F7532-9141-4926-AEE5-46396026040C}.png]]
<font style="fontsize:10">MAC(VTEP) | SIP,DIP(VTEP地址) | UDP4789 | VXLAN(vni) | SMAC.DMAC(PC) | SIP,DIP(PC) | ICMP</font>
封装模式也称为MAC in UDP
vxlan包头：vni 24bit，00000a （0000 0000 0000 0000 0000 1010）=10
新UDP包头：Dport 4789，Sport hash值，不一定是4789
新IP包头：SIP和DIP是vxlan的vtep地址
新MAC包头：SMAC，DMAC是L3网络的每段的两端地址的MAC

445566
112233
4455666
555888822