# 04. Services & Security (IOS Features)

OSPF 인증 · DHCP · UDP 브로드캐스트 관리 설정을 정리합니다.

---

## 4.1 OSPF Authentication (MD5)

Area 0 및 Virtual-Link 구간에 MD5(`message-digest`) 인증을 적용합니다. (key `history`)

```
! R2
router ospf 14
 area 0 authentication message-digest
 area 26 virtual-link 14.14.6.6 message-digest-key 1 md5 history
interface s1/0
 ip ospf message-digest-key 1 md5 history

! R6
router ospf 14
 area 0 authentication message-digest
 area 26 virtual-link 14.14.2.2 message-digest-key 1 md5 history
interface e0/0
 ip ospf message-digest-key 1 md5 history

! R4
router ospf 14
 area 0 authentication message-digest
interface s1/0
 ip ospf message-digest-key 1 md5 history
interface lo0
 ip ospf message-digest-key 1 md5 history

! R5
router ospf 14
 area 0 authentication message-digest
interface s1/0
 ip ospf message-digest-key 1 md5 history

! SW1
router ospf 14
 area 0 authentication message-digest
interface e1/2
 ip ospf message-digest-key 1 md5 history
```

---

## 4.2 DHCP (R6)

R6를 `14.14.62.0/24`(VLAN_B) DHCP 서버로 구성. 게이트웨이/DNS 이중화, lease 무제한.

```
! R6
ip dhcp excluded-address 14.14.62.6
ip dhcp excluded-address 14.14.62.2
ip dhcp pool DHCP
 network 14.14.62.0 255.255.255.0
 dns-server 150.100.1.50 150.100.1.51
 domain-name history.com
 lease infinite
 default-router 14.14.62.6 14.14.62.2
```

---

## 4.3 UDP Broadcast Management

브로드캐스트 기반 서비스(BOOTP/DHCP 등)를 지정 서버로 릴레이. `ip forward-protocol` + `ip helper-address`.

```
! R6
ip forward-protocol udp bootpc
interface e0/1.13
 ip helper-address 150.2.14.244
```
