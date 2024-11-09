## MPLS-VPN配置
![[Pasted image 20240531195437.png]]
用VPN实例区分不同的企业
在运营商边界路由器上配置实例
```
ip vpn instance A
ip vpn instance B
```

在实例中配置RD值,必须配置成不同的值。RD值本地有效
```
ip vpn instance A
	route-Distinguisher 100:1
ip vpn instance B
	route-Distinguisher 200:1
```
将实例绑定到接口上(**接口配置会全部清除，需要重新配置**)
```
int g0/0/0
	ip binding vpn-instance A
```
MP-BGP（VPNv4）
bgp基础配置
```
bgp 100
	peer 44.1.1.1 as 100
	peer 44.1.1.1 connect-interface lo0
```
开启VPNv4功能
```
bgp 100
	ipv4 family vpnv4
		peer 44.1.1.1 enable
```
只用vpnv4的话可以关闭unicast下的peer
```
bgp 100
	ipv4 family unicast
		undo peer 44.1.1.1 enable
```
将bgp与vpn实例A绑定，将属于实例A的路由引入bgp
```
bgp 100
	ipv4 family vpn-instance A
		import-router ospf 2
```
配置RT值，用于vpnv4的通信
both表示出去和进入都使用100:100这个值
通信需要对端出去的值和本地进入的值一样，对端进入的值和本地出去的值一样
```
ip vpn-instance A
	vpn-target 100:100 both
```
配置lsr id（这里用的环回口地址）
```
[R3]mpls lsr-id 33.1.1.1
[R5]mpls lsr-id 55.1.1.1
[R4]mpls lsr-id 44.1.1.1
```
配置完lsr id后，才能继续配置mpls ldp，用来分发公网标签
全局和接口下都要配置
```
[R3]mpls
[R3]mpls ldp
int g0/0/1
	mpls
	mpls ldp
```
最后引入路由，两端就可以通信了
