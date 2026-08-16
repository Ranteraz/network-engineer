# План адресации Underlay и Overlay сетей (CLOS Топология)

**Инфраструктурный пул (Underlay):** `10.0.0.0/16`  
**Клиентский пул (Overlay):** `192.168.0.0/16`

## 1. Loopback интерфейсы (Underlay)

| Коммутатор | Интерфейс | IP-адрес / Маска | Описание |
| :--- | :--- | :--- | :--- |
| **spain1** | Loopback0 | `10.0.254.1/32` | Spine 1 Router ID |
| **spain2** | Loopback0 | `10.0.254.2/32` | Spine 2 Router ID |
| **Leaf1** | Loopback0 | `10.0.254.11/32` | Leaf 1 VTEP |
| **Leaf2** | Loopback0 | `10.0.254.12/32` | Leaf 2 VTEP |
| **Leaf3** | Loopback0 | `10.0.254.13/32` | Leaf 3 VTEP |

## 2. Физические стыки Spine-Leaf (Underlay)

| Линк (Spine <---> Leaf) | IP на стороне Spine | IP на стороне Leaf | Маска подсети |
| :--- | :--- | :--- | :--- |
| **spain1** (Eth1) <---> **Leaf1** (Eth1) | `10.0.1.0` | `10.0.1.1` | `/31` |
| **spain1** (Eth2) <---> **Leaf2** (Eth1) | `10.0.1.2` | `10.0.1.3` | `/31` |
| **spain1** (Eth3) <---> **Leaf3** (Eth1) | `10.0.1.4` | `10.0.1.5` | `/31` |
| **spain2** (Eth1) <---> **Leaf1** (Eth2) | `10.0.1.6` | `10.0.1.7` | `/31` |
| **spain2** (Eth2) <---> **Leaf2** (Eth2) | `10.0.1.8` | `10.0.1.9` | `/31` |
| **spain2** (Eth3) <---> **Leaf3** (Eth2) | `10.0.1.10` | `10.0.1.11` | `/31` |

## 3. Клиентские сети (Слой Overlay)

*Сети для подключения конечных компьютеров (VPC).*

| Шлюз (Leaf порт) | IP шлюза | Подсеть клиента | Компьютер | IP компьютера |
| :--- | :--- | :--- | :--- | :--- |
| **Leaf1** (Eth3) | `192.168.0.254/24` | `192.168.0.0/24` | **VPC1** | `192.168.0.1` |
| **Leaf2** (Eth3) | `192.168.2.254/24` | `192.168.2.0/24` | **VPC2** | `192.168.2.1` |
| **Leaf3** (Eth3) | `192.168.3.254/24` | `192.168.3.0/24` | **VPC3** | `192.168.3.1` |
| **Leaf3** (Eth4) | `192.168.4.254/24` | `192.168.4.0/24` | **VPC4** | `192.168.4.1` |

### Этап 2:  на IPv6 Link-Local
*Интерфейсы переведены в IPv6-Only режим.*
*Все физические линки используют автоматически сгенерированные Link-Local адреса вида `fe80::/64`.
*   Адреса уникальны исключительно в пределах одного кабеля. Адресация на портах одного коммутатора может дублироваться (например, на `spain1` интерфейсы `Eth1`, `Eth2` и `Eth3` используют одинаковый адрес `fe80::5200:ff:fed7:ee0b`).



**spain1**
```
Interface       IP Address         Status      Protocol          MTU    Owner
--------------- ------------------ ----------- ------------- ---------- -------
Ethernet1       10.0.1.0/31        up          up               1500
Ethernet2       10.0.1.2/31        up          up               1500
Ethernet3       10.0.1.4/31        up          up               1500
Loopback0       10.0.254.1/32      up          up              65535
Management1     unassigned         up          up               1500

localhost#ping 10.0.1.1 repeat 2
PING 10.0.1.1 (10.0.1.1) 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=17.9 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=14.3 ms

--- 10.0.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 22ms
rtt min/avg/max/mdev = 14.252/16.099/17.947/1.847 ms, ipg/ewma 21.947/17.485 ms
localhost#ping 10.0.1.3 repeat 2
PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
80 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=42.0 ms
80 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=24.1 ms

--- 10.0.1.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 18ms
rtt min/avg/max/mdev = 24.132/33.068/42.005/8.936 ms, pipe 2, ipg/ewma 18.112/39.770 ms
localhost#ping 10.0.1.5 repeat 2
PING 10.0.1.5 (10.0.1.5) 72(100) bytes of data.
80 bytes from 10.0.1.5: icmp_seq=1 ttl=64 time=32.1 ms
80 bytes from 10.0.1.5: icmp_seq=2 ttl=64 time=16.3 ms

--- 10.0.1.5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 16ms
rtt min/avg/max/mdev = 16.296/24.181/32.066/7.885 ms, pipe 2, ipg/ewma 16.095/30.094 ms
```

**spain2**
```
Interface       IP Address         Status      Protocol          MTU    Owner
--------------- ------------------ ----------- ------------- ---------- -------
Ethernet1       10.0.1.6/31        up          up               1500
Ethernet2       10.0.1.8/31        up          up               1500
Ethernet3       10.0.1.10/31       up          up               1500
Loopback0       10.0.254.2/32      up          up              65535
Management1     unassigned         up          up               1500

localhost#ping 10.0.1.7 repeat 2
PING 10.0.1.7 (10.0.1.7) 72(100) bytes of data.
80 bytes from 10.0.1.7: icmp_seq=1 ttl=64 time=18.4 ms
80 bytes from 10.0.1.7: icmp_seq=2 ttl=64 time=2.56 ms

--- 10.0.1.7 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 21ms
rtt min/avg/max/mdev = 2.557/10.476/18.396/7.919 ms, ipg/ewma 20.640/16.416 ms
localhost#ping 10.0.1.9 repeat 2
PING 10.0.1.9 (10.0.1.9) 72(100) bytes of data.
80 bytes from 10.0.1.9: icmp_seq=1 ttl=64 time=23.5 ms
80 bytes from 10.0.1.9: icmp_seq=2 ttl=64 time=10.8 ms

--- 10.0.1.9 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 33ms
rtt min/avg/max/mdev = 10.770/17.152/23.535/6.382 ms, ipg/ewma 32.597/21.939 ms
localhost#ping 10.0.1.11 repeat 2
PING 10.0.1.11 (10.0.1.11) 72(100) bytes of data.
80 bytes from 10.0.1.11: icmp_seq=1 ttl=64 time=39.7 ms
80 bytes from 10.0.1.11: icmp_seq=2 ttl=64 time=9.24 ms

--- 10.0.1.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 9.241/24.469/39.698/15.228 ms, ipg/ewma 51.736/35.890 ms
```

**Leaf1**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       10.0.1.1/31          up         up              1500
Ethernet2       10.0.1.7/31          up         up              1500
Ethernet3       192.168.0.254/24     up         up              1500
Loopback0       10.0.254.11/32       up         up             65535
Management1     unassigned           up         up              1500

localhost#ping 192.168.0.1
PING 192.168.0.1 (192.168.0.1) 72(100) bytes of data.
80 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=13.3 ms
80 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=16.9 ms
80 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=0.841 ms
80 bytes from 192.168.0.1: icmp_seq=4 ttl=64 time=10.1 ms
80 bytes from 192.168.0.1: icmp_seq=5 ttl=64 time=17.6 ms

--- 192.168.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
```

**Leaf2**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       10.0.1.3/31          up         up              1500
Ethernet2       10.0.1.9/31          up         up              1500
Ethernet3       192.168.2.254/24     up         up              1500
Loopback0       10.0.254.12/32       up         up             65535
Management1     unassigned           up         up              1500

localhost#ping 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 72(100) bytes of data.
80 bytes from 192.168.2.1: icmp_seq=1 ttl=64 time=12.6 ms
80 bytes from 192.168.2.1: icmp_seq=2 ttl=64 time=1.39 ms
80 bytes from 192.168.2.1: icmp_seq=3 ttl=64 time=10.4 ms
80 bytes from 192.168.2.1: icmp_seq=4 ttl=64 time=4.92 ms
80 bytes from 192.168.2.1: icmp_seq=5 ttl=64 time=7.56 ms

--- 192.168.2.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 56ms
```

**Leaf3**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       10.0.1.5/31          up         up              1500
Ethernet2       10.0.1.11/31         up         up              1500
Ethernet3       192.168.3.254/24     up         up              1500
Ethernet4       192.168.4.254/24     up         up              1500
Loopback0       10.0.254.13/32       up         up             65535
Management1     unassigned           up         up              1500

localhost#ping 192.168.3.1
PING 192.168.3.1 (192.168.3.1) 72(100) bytes of data.
80 bytes from 192.168.3.1: icmp_seq=1 ttl=64 time=10.3 ms
80 bytes from 192.168.3.1: icmp_seq=2 ttl=64 time=0.778 ms
80 bytes from 192.168.3.1: icmp_seq=3 ttl=64 time=1.13 ms
80 bytes from 192.168.3.1: icmp_seq=4 ttl=64 time=1.42 ms
80 bytes from 192.168.3.1: icmp_seq=5 ttl=64 time=8.00 ms

--- 192.168.3.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 35ms
rtt min/avg/max/mdev = 0.778/4.330/10.332/4.020 ms, pipe 2, ipg/ewma 8.645/7.384 ms
localhost#ping 192.168.4.1
PING 192.168.4.1 (192.168.4.1) 72(100) bytes of data.
80 bytes from 192.168.4.1: icmp_seq=1 ttl=64 time=12.1 ms
80 bytes from 192.168.4.1: icmp_seq=2 ttl=64 time=19.2 ms
80 bytes from 192.168.4.1: icmp_seq=3 ttl=64 time=2.60 ms
80 bytes from 192.168.4.1: icmp_seq=4 ttl=64 time=3.90 ms
80 bytes from 192.168.4.1: icmp_seq=5 ttl=64 time=5.33 ms

--- 192.168.4.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
```

**Part2 ipv6 Link-Local**

**spain1**
```
localhost#show ipv interface brief
Interface  Status    MTU   IPv6 Address                 Addr State  Addr Source
---------- ------- ------ ---------------------------- ------------ -----------
Et1        up       1500   fe80::5200:ff:fed7:ee0b/64   up          link local
Et2        up       1500   fe80::5200:ff:fed7:ee0b/64   up          link local
Et3        up       1500   fe80::5200:ff:fed7:ee0b/64   up          link local

localhost#show ip interface brief
                                                                        Address
Interface       IP Address         Status      Protocol          MTU    Owner
--------------- ------------------ ----------- ------------- ---------- -------
Ethernet1       unassigned         up          up               1500
Ethernet2       unassigned         up          up               1500
Ethernet3       unassigned         up          up               1500
Loopback0       10.0.254.1/32      up          up              65535
Management1     unassigned         up          up               1500

localhost#ping fe80::5200:ff:fe03:3766 interface et1 repeat 2
PING fe80::5200:ff:fe03:3766%et1(fe80::5200:ff:fe03:3766%et1) 52 data bytes
60 bytes from fe80::5200:ff:fe03:3766%et1: icmp_seq=1 ttl=64 time=11.3 ms
60 bytes from fe80::5200:ff:fe03:3766%et1: icmp_seq=2 ttl=64 time=10.2 ms

--- fe80::5200:ff:fe03:3766%et1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 13ms
rtt min/avg/max/mdev = 10.247/10.778/11.309/0.531 ms, ipg/ewma 13.384/11.176 ms
localhost#ping fe80::5200:ff:fe15:f4e8 interface et2 repeat 2
PING fe80::5200:ff:fe15:f4e8%et2(fe80::5200:ff:fe15:f4e8%et2) 52 data bytes
60 bytes from fe80::5200:ff:fe15:f4e8%et2: icmp_seq=1 ttl=64 time=42.8 ms
60 bytes from fe80::5200:ff:fe15:f4e8%et2: icmp_seq=2 ttl=64 time=9.98 ms

--- fe80::5200:ff:fe15:f4e8%et2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 63ms
rtt min/avg/max/mdev = 9.982/26.393/42.804/16.411 ms, ipg/ewma 62.777/38.701 ms
localhost#ping fe80::5200:ff:fe72:8b31 interface et3 repeat 2
PING fe80::5200:ff:fe72:8b31%et3(fe80::5200:ff:fe72:8b31%et3) 52 data bytes
60 bytes from fe80::5200:ff:fe72:8b31%et3: icmp_seq=1 ttl=64 time=25.6 ms
60 bytes from fe80::5200:ff:fe72:8b31%et3: icmp_seq=2 ttl=64 time=9.63 ms

--- fe80::5200:ff:fe72:8b31%et3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 9.625/17.595/25.566/7.970 ms, ipg/ewma 45.505/23.573 ms
localhost#
```

**spain2**
```
Interface       IP Address         Status      Protocol          M
--------------- ------------------ ----------- ------------- ---------- -------
Ethernet1       unassigned         up          up               1500
Ethernet2       unassigned         up          up               1500
Ethernet3       unassigned         up          up               1500
Loopback0       10.0.254.2/32      up          up              65535
Management1     unassigned         up          up               1500

localhost(config-if-Et3)#show ipv interface brief
Interface  Status    MTU   IPv6 Address                 Addr State  Addr Source
---------- ------- ------ ---------------------------- ------------ -----------
Et1        up       1500   fe80::5200:ff:fed5:5dc0/64   up          link local
Et2        up       1500   fe80::5200:ff:fed5:5dc0/64   up          link local
Et3        up       1500   fe80::5200:ff:fed5:5dc0/64   up          link local

localhost(config-if-Et3)#ping fe80::5200:ff:fe03:3766 interface et1 repeat 2
PING fe80::5200:ff:fe03:3766%et1(fe80::5200:ff:fe03:3766%et1) 52 data bytes
60 bytes from fe80::5200:ff:fe03:3766%et1: icmp_seq=1 ttl=64 time=24.6 ms
60 bytes from fe80::5200:ff:fe03:3766%et1: icmp_seq=2 ttl=64 time=18.1 ms

--- fe80::5200:ff:fe03:3766%et1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 34ms
rtt min/avg/max/mdev = 18.079/21.357/24.636/3.278 ms, ipg/ewma 33.916/23.816 ms
localhost(config-if-Et3)#ping fe80::5200:ff:fe15:f4e8 interface et2 repeat 2
PING fe80::5200:ff:fe15:f4e8%et2(fe80::5200:ff:fe15:f4e8%et2) 52 data bytes
60 bytes from fe80::5200:ff:fe15:f4e8%et2: icmp_seq=1 ttl=64 time=54.2 ms
60 bytes from fe80::5200:ff:fe15:f4e8%et2: icmp_seq=2 ttl=64 time=31.3 ms

--- fe80::5200:ff:fe15:f4e8%et2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 36ms
rtt min/avg/max/mdev = 31.272/42.731/54.190/11.459 ms, pipe 2, ipg/ewma 35.846/51.325 ms
localhost(config-if-Et3)#ping fe80::5200:ff:fe72:8b31 interface et3 repeat 2
PING fe80::5200:ff:fe72:8b31%et3(fe80::5200:ff:fe72:8b31%et3) 52 data bytes
60 bytes from fe80::5200:ff:fe72:8b31%et3: icmp_seq=1 ttl=64 time=30.6 ms
60 bytes from fe80::5200:ff:fe72:8b31%et3: icmp_seq=2 ttl=64 time=8.58 ms

--- fe80::5200:ff:fe72:8b31%et3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 8.584/19.588/30.593/11.004 ms, ipg/ewma 52.277/27.841 ms
```


**Leaf1**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       unassigned           up         up              1500
Ethernet2       unassigned           up         up              1500
Ethernet3       192.168.0.254/24     up         up              1500
Loopback0       10.0.254.11/32       up         up             65535
Management1     unassigned           up         up              1500

localhost(config-if-Et3)#show ipv interface brief
Interface  Status    MTU   IPv6 Address                 Addr State  Addr Source
---------- ------- ------ ---------------------------- ------------ -----------
Et1        up       1500   fe80::5200:ff:fe03:3766/64   up          link local
Et2        up       1500   fe80::5200:ff:fe03:3766/64   up          link local
```

**Leaf2**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       unassigned           up         up              1500
Ethernet2       unassigned           up         up              1500
Ethernet3       192.168.2.254/24     up         up              1500
Loopback0       10.0.254.12/32       up         up             65535
Management1     unassigned           up         up              1500

localhost(config-if-Et3)#show ipv interface brief
Interface  Status    MTU   IPv6 Address                 Addr State  Addr Source
---------- ------- ------ ---------------------------- ------------ -----------
Et1        up       1500   fe80::5200:ff:fe15:f4e8/64   up          link local
Et2        up       1500   fe80::5200:ff:fe15:f4e8/64   up          link local
Et3        up       1500   fe80::5200:ff:fe15:f4e8/64   up          link local
```

**Leaf3**
```
Interface       IP Address           Status     Protocol         MTU    Owner
--------------- -------------------- ---------- ------------ ---------- -------
Ethernet1       unassigned           up         up              1500
Ethernet2       unassigned           up         up              1500
Ethernet3       192.168.3.254/24     up         up              1500
Ethernet4       192.168.4.254/24     up         up              1500
Loopback0       10.0.254.13/32       up         up             65535
Management1     unassigned           up         up              1500

localhost(config-if-Et4)#show ipv interface brief
Interface  Status    MTU   IPv6 Address                 Addr State  Addr Source
---------- ------- ------ ---------------------------- ------------ -----------
Et1        up       1500   fe80::5200:ff:fe72:8b31/64   up          link local
Et2        up       1500   fe80::5200:ff:fe72:8b31/64   up          link local
```