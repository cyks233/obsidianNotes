![[Pasted image 20240213150552.png]]
使用AC控制器给AP下发配置
1. 基础配置（打通AC,AP之间的路由）
	配置vlan，网关，IP地址。（E0/0/3口只能配trunk，STA1和AP1的两个vlan都要从这个口出去，还要配置pvid，让AP的vlan1报文打上vlan20的标签）
	SW1上配置DHCP（接口和全局都可），配置DHCP option 43用来让AP知道AC
	AC上配置默认路由。配置AP认证，建立capwap隧道。
	```
	==SW1==
	dhcp enable
	int vlanif 40
		dhcp select interface
		dhcp option 43 sub-option 2 ip-address 10.1.1.1
	==SW2==
	int e0/0/3
		port link-type trunk
		port trunk allow-pass vlan all
		port trunk pvid vlan 20
	==AC==
	capwap source interface vlanif 40
	wlan
		ap auth-mode mac-auth
		ap-id 0 ap-mac 00e0-fc42-6c60
		ap-name AP1
	```
2. AP的配置下发
	1. 配置国家代码
		```
		regulatory-domain-profile name AP
			country-code CN
		```
	2. 配置SSID(无线名)模板
		```
		ssid-profile name employee
			ssid employee
		ssid-profile name guest
			ssid guest
		```
	3. 配置安全模板
		```
		security-profile name AP
			security  wpa-wpa2 psk pass-phrase huawei123 aes 
		```
	1. 配置VAP(虚拟AP)模板
		每个VAP都会产生一个SSID
		调用SSID模板，关联安全模板，配置服务vlan，配置转发模式
		```
		vap-profile name Employee
			ssid-profile employee
			security-profile AP
			service-vlan vlan-id 30
			forward-mode tunnel
			
		vap-profile name Guest
			ssid-profile guest
			service-vlan vlan-id 60
			forward-mode tunnel
		```
	1. 调用VAP模板(使用AP组)
		```
		wlan
			ap-group name AP01
				regulatory-domain-profile AP
				vap-profile Employee wlan 1 radio all
				vap-profile Guest wlan 2 radio all
			ap-id 0
				ap-group AP01
		```

转发模式
1. tunnel  隧道转发
	数据先到AC，再到网关
2. direct-forward 直接转发
	数据直接到网关