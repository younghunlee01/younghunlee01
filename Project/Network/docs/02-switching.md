# 02. Switching (Bridging & Switching)
[← 목차로 돌아가기](../README.md)

SW1~SW4 스위치 구간의 VTP · Trunk · EtherChannel · VLAN · STP(MST) · UDLD · SVI 설정을 정리합니다. 설정 커맨드는 원문 그대로이며, 반복 구간도 스위치별로 모두 기재했습니다.

---

## 2.1 VTP

SW1을 Server, SW2~SW4를 Client로 구성해 SW1의 VLAN 정보를 하위 스위치가 학습하도록 합니다.

```
! SW1
vtp mode server
vtp domain history.com
vtp password history
vtp version 2

! SW2
vtp mode client
vtp domain history.com
vtp password history
vtp version 2

! SW3
vtp mode client
vtp domain history.com
vtp password history
vtp version 2

! SW4
vtp mode client
vtp domain history.com
vtp password history
vtp version 2
```

**검증** — `show vtp status` (SW1)

```
VTP Version running        : 2
VTP Domain Name            : history.com
VTP Operating Mode         : Server
Number of existing VLANs   : 14
Configuration Revision     : 31
```

---

## 2.2 Trunk (R6 ↔ SW2)

R6–SW2 사이를 트렁크로 만들고, 필요한 VLAN(13, 22)만 전달합니다. R6는 dot1q 서브인터페이스로 라우팅합니다.

```
! R6
interface e0/1
 no shutdown

interface e0/1.13
 encapsulation dot1q 13
 ip address 150.3.14.1 255.255.255.0

interface e0/1.22
 encapsulation dot1q 22
 ip address 14.14.62.6 255.255.255.0

! SW2
interface e1/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 13,22
```

---

## 2.3 EtherChannel

스위치 상호 다중 링크를 Port-channel로 묶습니다(`mode on`). 사용하지 않는 e2/2-3은 shutdown.

| Port-channel | 구간 | 포트 |
|--------------|------|------|
| Po12 | SW1 ↔ SW2 | e3/0-1 |
| Po13 | SW1 ↔ SW3 | e3/2-3 |
| Po34 | SW3 ↔ SW4 | e3/0-1 |
| Po24 | SW2 ↔ SW4 | e3/2-3 |

```
! SW1~SW4 공통 : 미사용 링크 차단
interface range e2/2-3
 shutdown

! SW1
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 12 mode on
interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 13 mode on

! SW2
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 12 mode on
interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 24 mode on

! SW3
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 34 mode on
interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 13 mode on

! SW4
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 34 mode on
interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 24 mode on
```

**로드밸런싱** (2.4)

```
! SW1-SW2
port-channel load-balance src-dst-ip
```

**검증** — `show etherchannel summary` (SW1)

```
Group  Port-channel  Protocol   Ports
12     Po12(SU)       -          Et3/0(P)  Et3/1(P)
13     Po13(SU)       -          Et3/2(P)  Et3/3(P)
```

---

## 2.4 VLAN

SW1(VTP Server)에 VLAN을 생성하고 각 스위치의 액세스 포트에 할당합니다.

| VLAN | 이름 | 용도 |
|------|------|------|
| 11 / 12 / 13 | BB1 / BB2 / BB3 | 백본 |
| 21 / 22 / 23 | VLAN_A / VLAN_B / VLAN_C | 서비스 |
| 50 / 79 | VLAN_CUSTOMER1 / CUSTOMER2 | 고객 |
| 100 | VLAN_SWITCHES | 스위치 관리(SVI) |

```
! SW1 : VLAN 생성
vlan 11
 name BB1
vlan 12
 name BB2
vlan 13
 name BB3
vlan 21
 name VLAN_A
vlan 22
 name VLAN_B
vlan 23
 name VLAN_C
vlan 50
 name VLAN_CUSTOMER1
vlan 79
 name VLAN_CUSTOMER2
vlan 100
 name VLAN_SWITCHS

! SW1 : 포트 할당 (e0/3, e1/2는 L3 라우티드 포트)
interface e2/0
 switchport mode access
 switchport access vlan 11
interface e0/0
 switchport mode access
 switchport access vlan 11
interface e0/2
 switchport mode access
 switchport access vlan 22
interface e0/3
 no switchport
 ip address 14.14.33.7 255.255.255.0
interface e1/2
 no switchport
 ip address 14.14.36.7 255.255.255.0

! SW2 : 포트 할당
interface e2/0
 switchport mode access
 switchport access vlan 12
interface e1/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 13,22
interface e0/2
 switchport mode access
 switchport access vlan 21
interface e1/1
 switchport mode access
 switchport access vlan 23

! SW3 : 포트 할당
interface e2/0
 switchport mode access
 switchport access vlan 13
```

---

## 2.5 STP (MST)

MST로 구성하고(Name `HISTORY`, Revision 1) 인스턴스별 root를 스위치에 분산합니다.

| Instance | VLAN | Root Primary |
|----------|------|--------------|
| 1 | 11, 21 | SW1 |
| 2 | 100 | SW4 |
| 3 | 12 | SW2 |

```
! SW1
spanning-tree mode mst
spanning-tree mst configuration
 name HISTORY
 revision 1
 instance 1 vlan 11,21
 instance 2 vlan 100
 instance 3 vlan 12
spanning-tree mst 1 root primary

! SW2
spanning-tree mode mst
spanning-tree mst configuration
 name HISTORY
 revision 1
 instance 1 vlan 11,21
 instance 2 vlan 100
 instance 3 vlan 12
spanning-tree mst 3 root primary

! SW3
spanning-tree mode mst
spanning-tree mst configuration
 name HISTORY
 revision 1
 instance 1 vlan 11,21
 instance 2 vlan 100
 instance 3 vlan 12

! SW4
spanning-tree mode mst
spanning-tree mst configuration
 name HISTORY
 revision 1
 instance 1 vlan 11,21
 instance 2 vlan 100
 instance 3 vlan 12
spanning-tree mst 2 root primary
```

> Instance 3(VLAN 12)은 2.11 단계에서 `instance 3 vlan 12`를 추가하고 SW2를 root primary로 지정합니다.

**검증** — `show spanning-tree mst configuration` (SW1)

```
Name      [HISTORY]
Revision  1     Instances configured 4
Instance  Vlans mapped
0         1-10,13-20,22-99,101-4094
1         11,21
2         100
3         12
```

---

## 2.6 UDLD / MAC Aging

```
! SW3-SW4 : 단방향 링크 감지
interface range e3/0-1
 udld port aggressive

! SW3 : VLAN 13 MAC aging-time 조정
mac address-table aging-time 500 vlan 13
```

---

## 2.7 VLAN 100 SVI (VLAN_SWITCHES)

각 스위치에 VLAN 100 SVI를 생성해 스위치 간 관리/EIGRP 통신에 사용합니다. (`14.14.90.0/24`)

```
! SW1
interface vlan 100
 no shutdown
 ip address 14.14.90.1 255.255.255.0

! SW2
interface vlan 100
 no shutdown
 ip address 14.14.90.2 255.255.255.0

! SW3
interface vlan 100
 no shutdown
 ip address 14.14.90.3 255.255.255.0

! SW4
interface vlan 100
 no shutdown
 ip address 14.14.90.4 255.255.255.0
```
