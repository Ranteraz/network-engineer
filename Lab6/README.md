Лабораторная работа: Настроить маршрутизацию в рамках Overlay между клиентами.
# IP-план для топологии Spine-Leaf (IS-IS / BGP EVPN / VXLAN L3)
![Топология сети для BGP VxLAN EVPN](../images/lab7.png)
## 1. Loopback-интерфейсы (Управляющие адреса /32)
* **Loopback0:** Маршрутизация (IS-IS, BGP Router ID, Peering)
* **Loopback1:** VXLAN VTEP IP (только на Leaf-коммутаторах)

| Устройство | Loopback0 (Router ID) | Loopback1 (VXLAN VTEP) |
| :--- | :--- | :--- |
| **spain1** | `10.0.0.11/32` | — |
| **spain2** | `10.0.0.12/32` | — |
| **Leaf1** | `10.0.0.1/32` | `10.0.1.1/32` |
| **Leaf2** | `10.0.0.2/32` | `10.0.1.2/32` |
| **Leaf3** | `10.0.0.3/32` | `10.0.1.3/32` |

---

## 2. Фабричные стыки (Underlay-линки /31)

### Стыки со Spine 1 (`spain1`)
* **Линк Leaf1 <-> spain1** (Подсеть: `10.1.0.0/31`)
  * `spain1` (Ethernet1): `10.1.0.0/31`
  * `Leaf1` (Ethernet1): `10.1.0.1/31`
* **Линк Leaf2 <-> spain1** (Подсеть: `10.1.0.2/31`)
  * `spain1` (Ethernet2): `10.1.0.2/31`
  * `Leaf2` (Ethernet1): `10.1.0.3/31`
* **Линк Leaf3 <-> spain1** (Подсеть: `10.1.0.4/31`)
  * `spain1` (Ethernet3): `10.1.0.4/31`
  * `Leaf3` (Ethernet1): `10.1.0.5/31`

### Стыки со Spine 2 (`spain2`)
* **Линк Leaf1 <-> spain2** (Подсеть: `10.1.0.6/31`)
  * `spain2` (Ethernet1): `10.1.0.6/31`
  * `Leaf1` (Ethernet2): `10.1.0.7/31`
* **Линк Leaf2 <-> spain2** (Подсеть: `10.1.0.8/31`)
  * `spain2` (Ethernet2): `10.1.0.8/31`
  * `Leaf2` (Ethernet2): `10.1.0.9/31`
* **Линк Leaf3 <-> spain2** (Подсеть: `10.1.0.10/31`)
  * `spain2` (Ethernet3): `10.1.0.10/31`
  * `Leaf3` (Ethernet2): `10.1.0.11/31`

---

## 3. Сегменты клиентов (Overlay / Подсети VPC)

### Сегмент VLAN 10
* **Подсеть:** `192.168.10.0/24`
* **Anycast GW:** `192.168.10.254`
* **Узлы:**
  * `Leaf1` (Ethernet3) & `Leaf2` (Ethernet3): `192.168.10.254`
  * `VPC1` (eth0): `192.168.10.10/24` (GW: `192.168.10.254`)
  * `VPC2` (eth0): `192.168.10.20/24` (GW: `192.168.10.254`)

### Сегмент VLAN 20
* **Подсеть:** `192.168.20.0/24`
* **Anycast GW:** `192.168.20.254`
* **Узлы:**
  * `Leaf3` (Ethernet3 и Ethernet4): `192.168.20.254`
  * `VPC3` (eth0): `192.168.20.30/24` (GW: `192.168.20.254`)
  * `VPC4` (eth0): `192.168.20.40/24` (GW: `192.168.20.254`)

---

## 4. Системные параметры фабрики
* **IS-IS Area ID:** `49.0001`
* **iBGP EVPN ASN:** `65000`
* **Transit L3 VNI:** `50001` (VRF `Tenant_A`)

## 5. IP Настройки фабрики и подтверждение связанности.
* **Для сокращения объема листинга, приведу настроки только с Spine1, Spine2, Leaf1
* **На данном этапе Loopback-интерфейсы пока не могут достучаться друг-друга
* **Причина ошибки ping: connect: Network is unreachable при обращении к 10.0.0.1 и 10.0.0.2 заключается в том, 
    что еще не запущен протокол динамической маршрутизации (IS-IS). 
	Устройство знает только о своих напрямую подключенных сетях, но не имеет маршрутов к удаленным Loopback-интерфейсам.
```
	spaine1#sh ip interface brief
																			Address
	Interface        IP Address        Status      Protocol          MTU    Owner
	---------------- ----------------- ----------- ------------- ---------- -------
	Ethernet1        10.1.0.0/31       up          up               1500
	Ethernet2        10.1.0.2/31       up          up               1500
	Ethernet3        10.1.0.4/31       up          up               1500
	Loopback0        10.0.0.11/32      up          up              65535
	Management1      unassigned        up          up               1500

	spaine1#ping 10.1.0.1 repeat 2
	PING 10.1.0.1 (10.1.0.1) 72(100) bytes of data.
	80 bytes from 10.1.0.1: icmp_seq=1 ttl=64 time=32.6 ms
	80 bytes from 10.1.0.1: icmp_seq=2 ttl=64 time=20.2 ms

	--- 10.1.0.1 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 17ms
	rtt min/avg/max/mdev = 20.193/26.374/32.555/6.181 ms, pipe 2, ipg/ewma 16.792/31.009 ms
	spaine1#ping 10.0.0.1 repeat 2
```

```
	Interface        IP Address        Status      Protocol          MTU    Owner
	---------------- ----------------- ----------- ------------- ---------- -------
	Ethernet1        10.1.0.6/31       up          up               1500
	Ethernet2        10.1.0.8/31       up          up               1500
	Ethernet3        10.1.0.10/31      up          up               1500
	Loopback0        10.0.0.12/32      up          up              65535
	Management1      unassigned        up          up               1500

	spine2(config)#ping 10.1.0.7 repeat 2
	PING 10.1.0.7 (10.1.0.7) 72(100) bytes of data.
	80 bytes from 10.1.0.7: icmp_seq=1 ttl=64 time=71.6 ms
	80 bytes from 10.1.0.7: icmp_seq=2 ttl=64 time=60.7 ms

	--- 10.1.0.7 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 12ms
	rtt min/avg/max/mdev = 60.700/66.144/71.589/5.444 ms, pipe 2, ipg/ewma 12.164/70.227 ms
	spine2(config)#ping 10.0.0.1 repeat 2
	ping: connect: Network is unreachable
```
```
	leaf1#sh ip interface brief
																			Address
	Interface        IP Address        Status      Protocol          MTU    Owner
	---------------- ----------------- ----------- ------------- ---------- -------
	Ethernet1        10.1.0.1/31       up          up               1500
	Ethernet2        10.1.0.7/31       up          up               1500
	Loopback0        10.0.0.11/32      up          up              65535
	Loopback1        unassigned        up          up              65535
	Management1      unassigned        up          up               1500

	leaf1#ping 10.1.0.0 repeat 2
	PING 10.1.0.0 (10.1.0.0) 72(100) bytes of data.
	80 bytes from 10.1.0.0: icmp_seq=1 ttl=64 time=59.4 ms
	80 bytes from 10.1.0.0: icmp_seq=2 ttl=64 time=41.9 ms

	--- 10.1.0.0 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 18ms
	rtt min/avg/max/mdev = 41.934/50.642/59.351/8.708 ms, pipe 2, ipg/ewma 18.471/57.173 ms
	leaf1#ping 10.0.0.1 repeat 2
	ping: connect: Network is unreachable
```

## 6. Underlay. ISIS секция. Опорная сеть.
* **Зачем нужен:
* **Его единственная задача — обеспечить IP-связность внутри фабрики (между Spine и Leaf коммутаторами).
* **IS-IS строит маршруты между физическими интерфейсами и Loopback-адресами коммутаторов.
* **Он ничего не знает про клиентские VTEP, VLAN или VRF.
* **Главная цель IS-IS: максимально быстро и стабильно «прокинуть» IP-пакеты от одного Leaf (VTEP) к другому Leaf (VTEP) через Spine-коммутаторы

* **Обоснование(почему все не на BGP, BGP тоже может все соеденить): 
* **1.Разделение отвественности. IS-IS строит инфраструктурные маршруты фабрики (Loopback и физические стыки), 
* **а BGP занимается исключительно клиентскими сервисами (VTEP, VRF, EVPN). Ошибка в сервисах не «уронит» опорную сеть.

* **2. Скорость сходимости (Convergence Time): IS-IS (Link-State протокол) мгновенно узнает об упавшем линке между Spine и Leaf и перестраивает топологию за миллисекунды.
* **BGP (Path-Vector) по своей природе медленнее реагирует на физические изменения.

```
	spaine1#show isis neighbors

	Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
	FABRIC    default  leaf1            L2   Ethernet1          P2P               UP    23          17
	FABRIC    default  leaf2            L2   Ethernet2          P2P               UP    21          1E
	FABRIC    default  leaf3            L2   Ethernet3          P2P               UP    25          12
	spaine1#show isis interface brief

	IS-IS Instance: FABRIC VRF: default

	Interface Level IPv4 Metric IPv6 Metric Type           Adjacency
	--------- ----- ----------- ----------- -------------- ---------
	Ethernet1 L2             10          10 point-to-point         1
	Ethernet3 L2             10          10 point-to-point         1
	Ethernet2 L2             10          10 point-to-point         1
	Loopback0 L2             10          10 loopback       (passive)

	spaine1#show ip route isis

	VRF: default
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	 I L2     10.0.0.1/32 [115/20]
			   via 10.1.0.1, Ethernet1
	 I L2     10.0.0.2/32 [115/20]
			   via 10.1.0.3, Ethernet2
	 I L2     10.0.0.3/32 [115/20]
			   via 10.1.0.5, Ethernet3
	 I L2     10.0.0.12/32 [115/30]
			   via 10.1.0.1, Ethernet1
			   via 10.1.0.3, Ethernet2
			   via 10.1.0.5, Ethernet3
	 I L2     10.1.0.6/31 [115/20]
			   via 10.1.0.1, Ethernet1
	 I L2     10.1.0.8/31 [115/20]
			   via 10.1.0.3, Ethernet2
	 I L2     10.1.0.10/31 [115/20]
			   via 10.1.0.5, Ethernet3

	spaine1#show running-config section isis
	interface Ethernet1
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet2
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet3
	   isis enable FABRIC
	   isis network point-to-point
	interface Loopback0
	   isis enable FABRIC
	router isis FABRIC
	   net 49.0001.0000.0000.0011.00
	   is-type level-2
	   !
	   address-family ipv4 unicast
```
```
	spine2#show isis interface brief

	IS-IS Instance: FABRIC VRF: default

	Interface Level IPv4 Metric IPv6 Metric Type           Adjacency
	--------- ----- ----------- ----------- -------------- ---------
	Ethernet3 L2             10          10 point-to-point         1
	Ethernet2 L2             10          10 point-to-point         1
	Ethernet1 L2             10          10 point-to-point         1
	Loopback0 L2             10          10 loopback       (passive)

	spine2#show ip route isis

	VRF: default
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	 I L2     10.0.0.1/32 [115/20]
			   via 10.1.0.7, Ethernet1
	 I L2     10.0.0.2/32 [115/20]
			   via 10.1.0.9, Ethernet2
	 I L2     10.0.0.3/32 [115/20]
			   via 10.1.0.11, Ethernet3
	 I L2     10.0.0.11/32 [115/30]
			   via 10.1.0.7, Ethernet1
			   via 10.1.0.9, Ethernet2
			   via 10.1.0.11, Ethernet3
	 I L2     10.1.0.0/31 [115/20]
			   via 10.1.0.7, Ethernet1
	 I L2     10.1.0.2/31 [115/20]
			   via 10.1.0.9, Ethernet2
	 I L2     10.1.0.4/31 [115/20]
			   via 10.1.0.11, Ethernet3

	spine2#show running-config section isis
	interface Ethernet1
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet2
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet3
	   isis enable FABRIC
	   isis network point-to-point
	interface Loopback0
	   isis enable FABRIC
	router isis FABRIC
	   net 49.0001.0000.0000.0012.00
	   is-type level-2
	   !
	   address-family ipv4 unicast
	spine2#show running-config section interface
	no service interface inactive port-id allocation disabled
	interface Ethernet1
	   no switchport
	   ip address 10.1.0.6/31
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet2
	   no switchport
	   ip address 10.1.0.8/31
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet3
	   no switchport
	   ip address 10.1.0.10/31
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet4
	interface Ethernet5
	interface Ethernet6
	interface Ethernet7
	interface Ethernet8
	interface Loopback0
	   ip address 10.0.0.12/32
	   isis enable FABRIC
	interface Management1
```
```
	leaf1#show isis neighbors

	Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
	FABRIC    default  spaine1          L2   Ethernet1          P2P               UP    21          10
	FABRIC    default  spine2           L2   Ethernet2          P2P               UP    21          17
	leaf1#show isis interface brief

	IS-IS Instance: FABRIC VRF: default

	Interface Level IPv4 Metric IPv6 Metric Type           Adjacency
	--------- ----- ----------- ----------- -------------- ---------
	Ethernet2 L2             10          10 point-to-point         1
	Ethernet1 L2             10          10 point-to-point         1
	Loopback0 L2             10          10 loopback       (passive)

	leaf1#show ip route isis

	VRF: default
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	 I L2     10.0.0.2/32 [115/30]
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.0.3/32 [115/30]
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.0.11/32 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.0.0.12/32 [115/20]
			   via 10.1.0.6, Ethernet2
	 I L2     10.1.0.2/31 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.1.0.4/31 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.1.0.8/31 [115/20]
			   via 10.1.0.6, Ethernet2
	 I L2     10.1.0.10/31 [115/20]
			   via 10.1.0.6, Ethernet2

	leaf1#show running-config section isis
	interface Ethernet1
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet2
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet3
	   isis enable FABRIC
	interface Loopback0
	   isis enable FABRIC
	router isis FABRIC
	   net 49.0001.0000.0000.0001.00
	   is-type level-2
	   !
	   address-family ipv4 unicast
	leaf1#show running-config section interface
	no service interface inactive port-id allocation disabled
	interface Ethernet1
	   no switchport
	   ip address 10.1.0.1/31
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet2
	   no switchport
	   ip address 10.1.0.7/31
	   isis enable FABRIC
	   isis network point-to-point
	interface Ethernet3
	   isis enable FABRIC
	interface Ethernet4
	interface Ethernet5
	interface Ethernet6
	interface Ethernet7
	interface Ethernet8
	interface Loopback0
	   ip address 10.0.0.1/32
	   isis enable FABRIC
	interface Loopback1
	interface Management1
```
```
spaine1#ping 10.0.0.12 repeat 2
PING 10.0.0.12 (10.0.0.12) 72(100) bytes of data.
80 bytes from 10.0.0.12: icmp_seq=1 ttl=63 time=59.7 ms
80 bytes from 10.0.0.12: icmp_seq=2 ttl=63 time=41.2 ms

--- 10.0.0.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 19ms
rtt min/avg/max/mdev = 41.222/50.443/59.664/9.221 ms, pipe 2, ipg/ewma 18.743/57.358 ms
spaine1#ping 10.0.0.3 repeat 2
PING 10.0.0.3 (10.0.0.3) 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=74.3 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=54.2 ms

--- 10.0.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 21ms
rtt min/avg/max/mdev = 54.209/64.254/74.300/10.045 ms, pipe 2, ipg/ewma 21.220/71.788 ms
```

## 7. Overlay. Control Plane. BGP секция. Клиентская сеть. 
* **Зачем нужен:
* **По протоколу BGP коммутаторы Leaf обмениваются информацией о MAC-адресах, 
* **IP-адресах клиентов и маршрутах внутри конкретных VRF. Вся эта сигнализация идет поверх построенной IS-IS сети.
* **Обоснование:
* **1. Разделение отвественности. Если падает физический линк, IS-IS мгновенно перестраивает маршрут в Underlay (за миллисекунды). 
* **При этом BGP EVPN в оверлее даже не замечает сбоя и продолжает работать без потери сессий.

* **2.  IS-IS быстро сходится на физическом уровне, а BGP справляется с клиентскими маршрутами и мак-адресами.
```
	spaine1(config-router-bgp)#show run section bgp
	router bgp 65000
	   router-id 10.0.0.11
	   no bgp default ipv4-unicast
	   bgp cluster-id 10.0.0.11
	   neighbor LEAFS-EVPN peer group
	   neighbor LEAFS-EVPN remote-as 65000
	   neighbor LEAFS-EVPN update-source Loopback0
	   neighbor LEAFS-EVPN route-reflector-client
	   neighbor LEAFS-EVPN send-community extended
	   neighbor SPINES-EVPN peer group
	   neighbor SPINES-EVPN remote-as 65000
	   neighbor SPINES-EVPN update-source Loopback0
	   neighbor SPINES-EVPN send-community extended
	   neighbor 10.0.0.1 peer group LEAFS-EVPN
	   neighbor 10.0.0.2 peer group LEAFS-EVPN
	   neighbor 10.0.0.3 peer group LEAFS-EVPN
	   neighbor 10.0.0.12 peer group SPINES-EVPN
	   !
	   address-family evpn
		  neighbor LEAFS-EVPN activate
		  neighbor SPINES-EVPN activate
	spaine1(config-router-bgp)#sh bgp summary
	BGP summary information for VRF default
	Router identifier 10.0.0.11, local AS number 65000
	Neighbor           AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc   NLRI Adv
	--------- ----------- ------------- ----------------------- -------------- ---------- ---------- ----------
	10.0.0.1        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.2        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.3        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.12       65000 Connect       L2VPN EVPN              Configured              0          0          0
```
```
	spine2#show run section bgp
	router bgp 65000
	   router-id 10.0.0.12
	   no bgp default ipv4-unicast
	   bgp cluster-id 10.0.0.12
	   neighbor LEAFS-EVPN peer group
	   neighbor LEAFS-EVPN remote-as 65000
	   neighbor LEAFS-EVPN update-source Loopback0
	   neighbor LEAFS-EVPN route-reflector-client
	   neighbor LEAFS-EVPN send-community extended
	   neighbor SPINES-EVPN peer group
	   neighbor SPINES-EVPN remote-as 65000
	   neighbor SPINES-EVPN update-source Loopback0
	   neighbor SPINES-EVPN send-community extended
	   neighbor 10.0.0.1 peer group LEAFS-EVPN
	   neighbor 10.0.0.2 peer group LEAFS-EVPN
	   neighbor 10.0.0.3 peer group LEAFS-EVPN
	   neighbor 10.0.0.11 peer group SPINES-EVPN
	   !
	   address-family evpn
		  neighbor LEAFS-EVPN activate
		  neighbor SPINES-EVPN activate
	spine2#sh bgp summary
	BGP summary information for VRF default
	Router identifier 10.0.0.12, local AS number 65000
	Neighbor           AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc   NLRI Adv
	--------- ----------- ------------- ----------------------- -------------- ---------- ---------- ----------
	10.0.0.1        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.2        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.3        65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.11       65000 Established   L2VPN EVPN              Negotiated              0          0          0
```
```
	leaf1#show run section bgp
	router bgp 65000
	   router-id 10.0.0.1
	   no bgp default ipv4-unicast
	   neighbor SPINES-EVPN peer group
	   neighbor SPINES-EVPN remote-as 65000
	   neighbor SPINES-EVPN update-source Loopback0
	   neighbor SPINES-EVPN send-community extended
	   neighbor 10.0.0.11 peer group SPINES-EVPN
	   neighbor 10.0.0.12 peer group SPINES-EVPN
	   !
	   address-family evpn
		  neighbor SPINES-EVPN activate
	leaf1#sh bgp summary
	BGP summary information for VRF default
	Router identifier 10.0.0.1, local AS number 65000
	Neighbor           AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc   NLRI Adv
	--------- ----------- ------------- ----------------------- -------------- ---------- ---------- ----------
	10.0.0.11       65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.12       65000 Established   L2VPN EVPN              Negotiated              0          0          0
```
```
	leaf2(config-router-bgp)#sh run section bgp
	router bgp 65000
	   router-id 10.0.0.2
	   no bgp default ipv4-unicast
	   neighbor SPINES-EVPN peer group
	   neighbor SPINES-EVPN remote-as 65000
	   neighbor SPINES-EVPN update-source Loopback0
	   neighbor SPINES-EVPN send-community extended
	   neighbor 10.0.0.11 peer group SPINES-EVPN
	   neighbor 10.0.0.12 peer group SPINES-EVPN
	   !
	   address-family evpn
		  neighbor SPINES-EVPN activate
	leaf2(config-router-bgp)#sh bgp summary
	BGP summary information for VRF default
	Router identifier 10.0.0.2, local AS number 65000
	Neighbor           AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc   NLRI Adv
	--------- ----------- ------------- ----------------------- -------------- ---------- ---------- ----------
	10.0.0.11       65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.12       65000 Established   L2VPN EVPN              Negotiated              0          0          0
```
```
	leaf3#sh run section bgp
	router bgp 65000
	   router-id 10.0.0.3
	   no bgp default ipv4-unicast
	   neighbor SPINES-EVPN peer group
	   neighbor SPINES-EVPN remote-as 65000
	   neighbor SPINES-EVPN update-source Loopback0
	   neighbor SPINES-EVPN send-community extended
	   neighbor 10.0.0.11 peer group SPINES-EVPN
	   neighbor 10.0.0.12 peer group SPINES-EVPN
	   !
	   address-family evpn
		  neighbor SPINES-EVPN activate
	leaf3#sh bgp summary
	BGP summary information for VRF default
	Router identifier 10.0.0.3, local AS number 65000
	Neighbor           AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc   NLRI Adv
	--------- ----------- ------------- ----------------------- -------------- ---------- ---------- ----------
	10.0.0.11       65000 Established   L2VPN EVPN              Negotiated              0          0          0
	10.0.0.12       65000 Established   L2VPN EVPN              Negotiated              0          0          0
```

## 8. Секция VRF.

### Понятие VRF и его назначение
* **VRF (Virtual Routing and Forwarding)** — технология виртуализации сетевого уровня (L3), разделяющая один физический коммутатор на несколько независимых виртуальных маршрутизаторов. 

* **Зачем нужен:** Обеспечивает строгую изоляцию трафика разных клиентов (мультитенантность) в рамках единой физической инфраструктуры ЦОД и позволяет использовать пересекающиеся IP-адреса без конфликтов. 
* **В данной работе контекст `Tenant_A` полностью изолирован от глобальной таблицы маршрутизации (`default`).

### Анализ текущей ситуации связности (Тестирование VPC)
* **На текущем этапе (настроен только VRF, без VXLAN) проверка связности показывает следующие результаты:

* **Доступность шлюза (VPC 1-4 -> Anycast GW):** Успешно. Все хосты имеют доступ к локальным распределенным шлюзам (`192.168.10.254` и `192.168.20.254`). Это подтверждает корректность настройки Anycast Gateway.
* **Локальная L2-связность (VPC3 <-> VPC4):** Успешно. Хосты видят друг друга, так как подключены к одному физическому коммутатору (`leaf3`) в рамках общего VLAN 20. Трафик коммутируется локально.
* **Удаленная связность (VPC1 <-> VPC2):** Недоступно (`not reachable`). Хосты находятся в одном VLAN 10, но на разных коммутаторах (`leaf1` и `leaf2`).
* **Передача L2-трафика между ними через Spine-коммутаторы невозможна до момента настройки VXLAN-инкапсуляции.
 
* **Команды конфигуррирования VRF, на примере leaf3
```
	vrf instance Tenant_A
	!
	ip routing vrf Tenant_A
	!
	ip virtual-router mac-address 00:1c:73:00:00:99
	!
	vlan 20
	   name Client-B
	!
	interface Ethernet3
	   switchport mode access
	   switchport access vlan 20
	   no shutdown
	!
	interface Ethernet4
	   switchport mode access
	   switchport access vlan 20
	   no shutdown
	!
	interface VLAN 20
	   vrf Tenant_A
	   ip address 192.168.20.3/24
	   ip virtual-router address 192.168.20.254
```
* **Демонстрация настроек VRF на Leaf1-3
```
	leaf1(config)#do show running-config section Tenant_A
	vrf instance Tenant_A
	interface Vlan10
	   vrf Tenant_A
	ip routing vrf Tenant_A
	leaf1(config)#do show ip interface brief
																			Address
	Interface       IP Address          Status     Protocol          MTU    Owner
	--------------- ------------------- ---------- ------------- ---------- -------
	Ethernet1       10.1.0.1/31         up         up               1500
	Ethernet2       10.1.0.7/31         up         up               1500
	Loopback0       10.0.0.1/32         up         up              65535
	Loopback1       unassigned          up         up              65535
	Management1     unassigned          up         up               1500
	Vlan10          192.168.10.1/24     up         up               1500
```
```
	leaf2(config)#do show running-config section Tenant_A
	vrf instance Tenant_A
	interface Vlan10
	   vrf Tenant_A
	ip routing vrf Tenant_A
	leaf2(config)#!
	leaf2(config)#do show ip interface brief
																			Address
	Interface       IP Address          Status     Protocol          MTU    Owner
	--------------- ------------------- ---------- ------------- ---------- -------
	Ethernet1       10.1.0.3/31         up         up               1500
	Ethernet2       10.1.0.9/31         up         up               1500
	Loopback0       10.0.0.2/32         up         up              65535
	Management1     unassigned          up         up               1500
	Vlan10          192.168.10.2/24     up         up               1500
```
```
	leaf3(config-if-Vl20)#do show running-config section Tenant_A
	vrf instance Tenant_A
	interface Vlan20
	   vrf Tenant_A
	ip routing vrf Tenant_A
	leaf3(config-if-Vl20)#!
	leaf3(config-if-Vl20)#do show ip interface brief
																			Address
	Interface       IP Address          Status     Protocol          MTU    Owner
	--------------- ------------------- ---------- ------------- ---------- -------
	Ethernet1       10.1.0.5/31         up         up               1500
	Ethernet2       10.1.0.11/31        up         up               1500
	Loopback0       10.0.0.3/32         up         up              65535
	Loopback1       10.0.1.3/32         up         up              65535
	Management1     unassigned          up         up               1500
	Vlan20          192.168.20.3/24     up         up               1500
```
* **Демонстрация доступности с клиентских VPC1-4 для текущего этапа
```
	VPCS> ip 192.168.10.10 192.168.10.254
	Checking for duplicate address...
	VPCS : 192.168.10.10 255.255.255.0 gateway 192.168.10.254

	VPCS> ping 192.168.10.254
	84 bytes from 192.168.10.254 icmp_seq=1 ttl=64 time=46.912 ms
	84 bytes from 192.168.10.254 icmp_seq=2 ttl=64 time=12.010 ms
	
	VPCS> ping 192.168.10.20
	host (192.168.10.20) not reachable
```
```
	VPCS> ip 192.168.10.20/24 192.168.10.254
	Checking for duplicate address...
	VPCS : 192.168.10.20 255.255.255.0 gateway 192.168.10.254

	VPCS> ping 192.168.10.254
	84 bytes from 192.168.10.254 icmp_seq=1 ttl=64 time=43.889 ms
	84 bytes from 192.168.10.254 icmp_seq=2 ttl=64 time=37.721 ms

	VPCS> ping 192.168.10.10
	host (192.168.10.10) not reachable
```
```
	VPCS> ip 192.168.20.30 192.168.20.254
	Checking for duplicate address...
	VPCS : 192.168.20.30 255.255.255.0 gateway 192.168.20.254

	VPCS> ping 192.168.20.254

	84 bytes from 192.168.20.254 icmp_seq=1 ttl=64 time=42.873 ms
	84 bytes from 192.168.20.254 icmp_seq=2 ttl=64 time=26.869 ms

	VPCS> ping 192.168.20.40
	84 bytes from 192.168.20.40 icmp_seq=1 ttl=64 time=3.841 ms
	84 bytes from 192.168.20.40 icmp_seq=2 ttl=64 time=12.008 ms
```
```
	VPCS> ip 192.168.20.40 192.168.20.254
	Checking for duplicate address...
	VPCS : 192.168.20.40 255.255.255.0 gateway 192.168.20.254

	VPCS> ping 192.168.20.254
	84 bytes from 192.168.20.254 icmp_seq=1 ttl=64 time=24.667 ms
	84 bytes from 192.168.20.254 icmp_seq=2 ttl=64 time=17.409 ms

	VPCS> ping 192.168.20.30
	84 bytes from 192.168.20.30 icmp_seq=1 ttl=64 time=11.832 ms
	84 bytes from 192.168.20.30 icmp_seq=2 ttl=64 time=14.383 ms
```

## 9. Data Plane. Секция VXLAN.
### Назначение VXLAN на данном этапе
**VXLAN (Virtual Extensible LAN)** применяется в качестве технологии **Data Plane (транспорта)** для инкапсуляции клиентского трафика в IP/UDP-пакеты и его передачи через физическую опорную сеть (Underlay).

* **Связующее звено:** Именно VXLAN связывает изолированные VRFы между разными коммутаторами и делает возможным прохождение трафика от одной VPC к другой через фабрику.
* **L2 VNI (10010, 10020):** Используются для «растягивания» плоских клиентских VLAN-сегментов между изолированными Leaf-коммутаторами. 
  *(Важное примечание: на данном этапе связность между VPC1 и VPC2 ещё отсутствует, так как коммутаторы не обменялись таблицами маршрутов до включения Control Plane).*
* **L3 VNI (50001):** Выступает в роли транзитной магистрали (Transit VNI) внутри контекста `Tenant_A` для маршрутизации трафика между разными клиентскими подсетями (Symmetric IRB) через фабрику.

### Обоснование использования двух Loopback-интерфейсов
* **Разделение управляющего адреса (`Loopback0`) и адреса туннеля (`Loopback1`):

* **Разделение обязанностей (Control vs Data Plane):** `Loopback0` используется строго для служебных протоколов (BGP-соседство, Router ID, IS-IS), а `Loopback1` (VTEP) — исключительно для аппаратной упаковки и передачи клиентских данных.
* **Резервирование и отказоустойчивость (Anycast VTEP):** Позволяет в будущем объединять Leaf-коммутаторы в отказоустойчивые пары (MLAG). При этом устройства сохраняют уникальные адреса для BGP и IS-IS (`Loopback0`), но используют один общий IP-адрес для VXLAN-туннеля (`Loopback1`).

* **Конфигурирование vxlan на примере Leaf 1
```
	leaf1(config)# interface Loopback1
	leaf1(config-if-Lo1)# ip address 10.0.1.1/32
	leaf1(config-if-Lo1)# isis enable FABRIC      ! Отдаем адрес VTEP в IS-IS, чтобы другие роутеры знали, куда слать VXLAN-пакеты
	leaf1(config-if-Lo1)# exit

	leaf1(config)# interface Vxlan1
	leaf1(config-if-Vx1)# vxlan source-interface Loopback1
	leaf1(config-if-Vx1)# vxlan udp-port 4789
	leaf1(config-if-Vx1)# vxlan vlan 10 vni 10010  ! Привязываем VLAN 10 к вашему L2 VNI
	leaf1(config-if-Vx1)# vxlan vrf Tenant_A vni 50001 ! Связываем VRF с транзитным L3 VNI
```
```
	leaf1(config-if-Vx1)#do show running-config interfaces vxlan 1
	interface Vxlan1
	   vxlan source-interface Loopback1
	   vxlan udp-port 4789
	   vxlan vlan 10 vni 10010
	   vxlan vrf Tenant_A vni 50001
	leaf1(config-if-Vx1)#do show interface vxlan 1
	Vxlan1 is up, line protocol is up (connected)
	  Hardware is Vxlan
	  Source interface is Loopback1 and is active with 10.0.1.1
	  Listening on UDP port 4789
	  Replication/Flood Mode is headend with Flood List Source: CLI
	  Remote MAC learning is disabled
	  VNI mapping to VLANs
	  Static VLAN to VNI mapping is
		[10, 10010]
	  Dynamic VLAN to VNI mapping for 'evpn' is
		[4097, 50001]
	  Note: All Dynamic VLANs used by VCS are internal VLANs.
			Use 'show vxlan vni' for details.
	  Static VRF to VNI mapping is
	   [Tenant_A, 50001]
	  Shared Router MAC is 0000.0000.0000
	leaf1(config-if-Vx1)#do show ip route vrf Tenant_A

	VRF: Tenant_A
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	Gateway of last resort is not set

	 C        192.168.10.0/24
			   directly connected, Vlan10

	leaf1(config-if-Vx1)#
```
```
	leaf2(config-if-Vx1)#do show running-config interfaces vxlan 1
	interface Vxlan1
	   vxlan source-interface Loopback1
	   vxlan udp-port 4789
	   vxlan vlan 10 vni 10010
	   vxlan vrf Tenant_A vni 50001
	leaf2(config-if-Vx1)#do show interface vxlan 1
	Vxlan1 is up, line protocol is up (connected)
	  Hardware is Vxlan
	  Source interface is Loopback1 and is active with 10.0.1.2
	  Listening on UDP port 4789
	  Replication/Flood Mode is headend with Flood List Source: CLI
	  Remote MAC learning is disabled
	  VNI mapping to VLANs
	  Static VLAN to VNI mapping is
		[10, 10010]
	  Dynamic VLAN to VNI mapping for 'evpn' is
		[4097, 50001]
	  Note: All Dynamic VLANs used by VCS are internal VLANs.
			Use 'show vxlan vni' for details.
	  Static VRF to VNI mapping is
	   [Tenant_A, 50001]
	  Shared Router MAC is 0000.0000.0000
	leaf2(config-if-Vx1)#do show ip route vrf Tenant_A

	VRF: Tenant_A
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	Gateway of last resort is not set

	 C        192.168.10.0/24
			   directly connected, Vlan10
```
```
	leaf3(config-if-Vx1)#do show running-config interfaces vxlan 1
	interface Vxlan1
	   vxlan source-interface Loopback1
	   vxlan udp-port 4789
	   vxlan vlan 20 vni 10020
	   vxlan vrf Tenant_A vni 50001
	leaf3(config-if-Vx1)#do show interface vxlan 1
	Vxlan1 is up, line protocol is up (connected)
	  Hardware is Vxlan
	  Source interface is Loopback1 and is active with 10.0.1.3
	  Listening on UDP port 4789
	  Replication/Flood Mode is headend with Flood List Source: CLI
	  Remote MAC learning is disabled
	  VNI mapping to VLANs
	  Static VLAN to VNI mapping is
		[20, 10020]
	  Dynamic VLAN to VNI mapping for 'evpn' is
		[4097, 50001]
	  Note: All Dynamic VLANs used by VCS are internal VLANs.
			Use 'show vxlan vni' for details.
	  Static VRF to VNI mapping is
	   [Tenant_A, 50001]
	  Shared Router MAC is 0000.0000.0000
	leaf3(config-if-Vx1)#do show ip route vrf Tenant_A

	VRF: Tenant_A
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	Gateway of last resort is not set

	 C        192.168.20.0/24
			   directly connected, Vlan20
```
### Проверка IP-доступности VTEP-интерфейсов в опорной сети (Underlay)
Перед запуском VXLAN-туннелирования необходимо убедиться, что физическая сеть (IS-IS Underlay) построила маршруты между интерфейсами инкапсуляции (`Loopback1`) всех коммутаторов фабрики. 
```
	leaf1(config-if-Vx1)#show ip route isis

	VRF: default
	Source Codes:
		   C - connected, S - static, K - kernel,
		   O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
		   O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
		   O N2 - OSPF NSSA external type2, O3 - OSPFv3,
		   O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
		   O3 E2 - OSPFv3 external type 2,
		   O3 N1 - OSPFv3 NSSA external type 1,
		   O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
		   B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
		   I L2 - IS-IS level 2, A B - BGP Aggregate,
		   A O - OSPF Summary, NG - Nexthop Group Static Route,
		   V - VXLAN Control Service, M - Martian,
		   DH - DHCP client installed default route,
		   DP - Dynamic Policy Route, L - VRF Leaked,
		   G  - gRIBI, RC - Route Cache Route,
		   CL - CBF Leaked Route

	 I L2     10.0.0.2/32 [115/30]
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.0.3/32 [115/30]
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.0.11/32 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.0.0.12/32 [115/20]
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.1.2/32 [115/30] - маршрут до VTEP-адреса leaf2 успешно изучен
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.0.1.3/32 [115/30] - маршрут до VTEP-адреса leaf3 успешно изучен
			   via 10.1.0.0, Ethernet1
			   via 10.1.0.6, Ethernet2
	 I L2     10.1.0.2/31 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.1.0.4/31 [115/20]
			   via 10.1.0.0, Ethernet1
	 I L2     10.1.0.8/31 [115/20]
			   via 10.1.0.6, Ethernet2
	 I L2     10.1.0.10/31 [115/20]
			   via 10.1.0.6, Ethernet2
```
Тестирование доступности VTEP-адресов соседей (`Leaf2` и `Leaf3`) с коммутатора `Leaf1`:

```
	leaf1(config-if-Vx1)#do ping 10.0.1.2
	PING 10.0.1.2 (10.0.1.2) 72(100) bytes of data.
	80 bytes from 10.0.1.2: icmp_seq=1 ttl=63 time=36.0 ms
	80 bytes from 10.0.1.2: icmp_seq=2 ttl=63 time=20.6 ms
	80 bytes from 10.0.1.2: icmp_seq=3 ttl=63 time=4.13 ms
	80 bytes from 10.0.1.2: icmp_seq=4 ttl=63 time=52.6 ms
	80 bytes from 10.0.1.2: icmp_seq=5 ttl=63 time=23.2 ms

	--- 10.0.1.2 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 114ms
	rtt min/avg/max/mdev = 4.129/27.313/52.632/16.229 ms, pipe 2, ipg/ewma 28.507/31.898 ms

	leaf1(config-if-Vx1)#do ping 10.0.1.3
	PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
	80 bytes from 10.0.1.3: icmp_seq=1 ttl=63 time=51.1 ms
	80 bytes from 10.0.1.3: icmp_seq=2 ttl=63 time=45.5 ms
	80 bytes from 10.0.1.3: icmp_seq=3 ttl=63 time=34.5 ms
	80 bytes from 10.0.1.3: icmp_seq=4 ttl=63 time=25.5 ms
	80 bytes from 10.0.1.3: icmp_seq=5 ttl=63 time=14.5 ms

	--- 10.0.1.3 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 108ms
	rtt min/avg/max/mdev = 14.512/34.220/51.111/13.201 ms, pipe 2
```

**Вывод теста:** Потери пакетов отсутствуют (0% packet loss), связность между VTEP-интерфейсами фабрики на транспортном уровне обеспечена.

## 10. Overlay Control Plane. BGP EVPN
* **После активации процесса BGP EVPN внутри контекста `Tenant_A` маршрутизация в оверлее полностью запущена. На данном этапе Control Plane успешно собрала информацию о топологии сети и распространила её по фабрике. 
* **Коммутаторы обменялись маршрутами типа 5 (IP Prefix) через Spine-коммутаторы, выполняющие роль Route Reflector. В результате в изолированной таблице маршрутизации VRF появились удаленные клиентские подсети, 
* **привязанные к интерфейсу `Vxlan1` и транзитному `L3VNI 50001`. Это делает возможной сквозную маршрутизацию (Symmetric IRB) между фабриками. 

* **Ниже представлены итоговые конфигурации, состояние полученных EVPN-префиксов и финальный вид таблицы маршрутизации:
```
	leaf1(config)# router bgp 65000                                 # Запуск глобального процесса BGP фабрики
	leaf1(config-router-bgp)# vrf Tenant_A                          # Переход в изолированный контекст клиента
	leaf1(config-router-bgp-vrf-Tenant_A)# rd 10.0.0.1:50001        # Уникальный различитель маршрутов для EVPN
	leaf1(config-router-bgp-vrf-Tenant_A)# route-target import evpn 50001:50001  # Прием чужих маршрутов с этой меткой в VRF
	leaf1(config-router-bgp-vrf-Tenant_A)# route-target export evpn 50001:50001  # Маркировка своих маршрутов этой меткой при отправке
	leaf1(config-router-bgp-vrf-Tenant_A)# redistribute connected   # Анонс локальных подсетей (VLAN) клиента в BGP EVPN
```
* ** Далее этот блок команд на осталтных лифах, меняться будет только сторока rd 10.0.0.2,3:50001

### 10.1 Демонстрация настроек BGP
```
leaf1(config-router-bgp-vrf-Tenant_A)#do show running-config section bgp
router bgp 65000
   router-id 10.0.0.1
   no bgp default ipv4-unicast
   neighbor SPINES-EVPN peer group
   neighbor SPINES-EVPN remote-as 65000
   neighbor SPINES-EVPN update-source Loopback0
   neighbor SPINES-EVPN send-community extended
   neighbor 10.0.0.11 peer group SPINES-EVPN
   neighbor 10.0.0.12 peer group SPINES-EVPN
   !
   address-family evpn
      neighbor SPINES-EVPN activate
   !
   vrf Tenant_A
      rd 10.0.0.1:50001
      route-target import evpn 50001:50001
      route-target export evpn 50001:50001
      redistribute connected
```
### 10.2. Демонстрация полученных EVPN-маршрутов 
```	  
leaf1(config-router-bgp-vrf-Tenant_A)#do show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending best path selection
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:50001 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.0.0.3:5001 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.0.11
 *        RD: 10.0.0.3:5001 ip-prefix 192.168.10.0/24
                                 10.0.1.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.0.12
 * >      RD: 10.0.0.3:50001 ip-prefix 192.168.20.0/24
                                 10.0.1.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.0.11
 *        RD: 10.0.0.3:50001 ip-prefix 192.168.20.0/24
                                 10.0.1.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.0.12
```
### 10.3. Финальное состояние таблицы маршрутизации VRF Tenant_A
```								 
leaf1(config-router-bgp-vrf-Tenant_A)#do show ip route vrf Tenant_A

VRF: Tenant_A
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, O IA - OSPF inter area, O E1 - OSPF external type 1,
       O E2 - OSPF external type 2, O N1 - OSPF NSSA external type 1,
       O N2 - OSPF NSSA external type2, O3 - OSPFv3,
       O3 IA - OSPFv3 inter area, O3 E1 - OSPFv3 external type 1,
       O3 E2 - OSPFv3 external type 2,
       O3 N1 - OSPFv3 NSSA external type 1,
       O3 N2 - OSPFv3 NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 C        192.168.10.0/24
           directly connected, Vlan10
 B I      192.168.20.0/24 [200/0]
           via VTEP 10.0.1.3 VNI 50001 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
```
## 10.4 Включение L2 EVPN
* **Если не включить L2 EVPN, то vpc1,2, находящие в одном vlan10, но вразных VRF/Leaf не будут видеть друг-друга.
* **Этот блок команд на осталтных лифах, меняться будет только сторока rd 10.0.0.х:50001
```
	leaf1(config)# router bgp 65000                                 # Запуск глобального процесса BGP 
	leaf1(config-router-bgp)# vlan-aware-bundle Tenant_A_L2_Services # Создание контейнера для L2-сервисов (MAC VRF)
	leaf1(config-macvrf-Tenant_A_L2_Services)# rd 10.0.0.1:10010    # Уникальный различитель маршрутов для плоского VLAN 10
	leaf1(config-macvrf-Tenant_A_L2_Services)# route-target import 10010:10010 # Прием чужих MAC-адресов этого влана в фабрику
	leaf1(config-macvrf-Tenant_A_L2_Services)# route-target export 10010:10010 # Маркировка и анонс своих локальных MAC-адресов в сеть
	leaf1(config-macvrf-Tenant_A_L2_Services)# vlan 10              # Привязка клиентского VLAN 10 к этому L2-процессу BGP
```
## 10.5 демонстрация настроек L2 EVPN
* **Вывод с остальных лифов аналогичен, меняется только строки с 10.0.0.x соответсвенно 
```
	leaf1(config-macvrf-Tenant_A_L2_Services)# do show bgp evpn instance
	EVPN instance: VLAN-aware bundle Tenant_A_L2_Services
	  Route distinguisher: 10.0.0.1:10010
	  Route target import: Route-Target-AS:10010:10010
	  Route target export: Route-Target-AS:10010:10010
	  Service interface: VLAN-aware bundle
	  Local VXLAN IP address: 10.0.1.1
	  VXLAN: enabled
	  MPLS: disabled
```

## 11. Финальная проверка связности между клиентами в Overlay (Тестирование)

После полной настройки Control Plane (BGP EVPN) и Data Plane (VXLAN) запущено итоговое тестирование прохождения трафика с хоста `VPC1` (`192.168.10.10`):

### 11.1. Проверка связности внутри одной подсети (L2 Overlay)
Трафик передается между `Leaf1` и `Leaf2` в рамках одного VLAN 10 через L2 VNI `10010`:
```
	VPCS> show ip

	NAME        : VPCS[1]
	IP/MASK     : 192.168.10.10/24
	GATEWAY     : 192.168.10.254
	DNS         :
	MAC         : 00:50:79:66:68:07
	LPORT       : 20000
	RHOST:PORT  : 127.0.0.1:30000
	MTU         : 1500

	VPCS> ping 192.168.10.20

	84 bytes from 192.168.10.20 icmp_seq=1 ttl=64 time=68.255 ms
	84 bytes from 192.168.10.20 icmp_seq=2 ttl=64 time=58.987 ms
	84 bytes from 192.168.10.20 icmp_seq=3 ttl=64 time=59.937 ms
	84 bytes from 192.168.10.20 icmp_seq=4 ttl=64 time=45.478 ms
	84 bytes from 192.168.10.20 icmp_seq=5 ttl=64 time=49.602 ms
```

### 11.2. Проверка связности между разными подсетями (L3 Overlay)
Маршрутизация трафика из VLAN 10 (`Leaf1`) во VLAN 20 (`Leaf3`) через транзитный магистральный L3 VNI `50001` (Symmetric IRB):
```
	VPCS> show ip

	NAME        : VPCS[1]
	IP/MASK     : 192.168.10.10/24
	GATEWAY     : 192.168.10.254
	DNS         :
	MAC         : 00:50:79:66:68:07
	LPORT       : 20000
	RHOST:PORT  : 127.0.0.1:30000
	MTU         : 1500

	VPCS> ping 192.168.20.30

	84 bytes from 192.168.20.30 icmp_seq=1 ttl=62 time=52.068 ms
	84 bytes from 192.168.20.30 icmp_seq=2 ttl=62 time=62.667 ms
	84 bytes from 192.168.20.30 icmp_seq=3 ttl=62 time=83.292 ms
	84 bytes from 192.168.20.30 icmp_seq=4 ttl=62 time=54.736 ms
	84 bytes from 192.168.20.30 icmp_seq=5 ttl=62 time=82.675 ms

	VPCS> ping 192.168.20.40

	84 bytes from 192.168.20.40 icmp_seq=1 ttl=62 time=84.765 ms
	84 bytes from 192.168.20.40 icmp_seq=2 ttl=62 time=69.975 ms
	84 bytes from 192.168.20.40 icmp_seq=3 ttl=62 time=72.662 ms
	84 bytes from 192.168.20.40 icmp_seq=4 ttl=62 time=40.143 ms
	84 bytes from 192.168.20.40 icmp_seq=5 ttl=62 time=71.325 ms
```

### 12. Таблица arp на Leaf1.
```
leaf1(config-macvrf-Tenant_A_L2_Services)#do show mac address-table vlan 10
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    001c.7300.0099    STATIC      Cpu
  10    0050.7966.6807    DYNAMIC     Et3        1       0:09:38 ago
Total Mac Addresses for this criterion: 2

          Multicast Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       ----        -----
Total Mac Addresses for this criterion: 0
```

```
leaf1(config-macvrf-Tenant_A_L2_Services)#do show ip arp vrf Tenant_A
Legend:
 not learned: Associated MAC address is not present in the MAC address table
 -: Static (configuration or programmed by feature)
Address         Age (sec)  Hardware Addr   Interface
192.168.10.10     2:21:14  0050.7966.6807  Vlan10, Ethernet3
192.168.10.20     1:38:54  0050.7966.6808  Vlan10, not learned
```