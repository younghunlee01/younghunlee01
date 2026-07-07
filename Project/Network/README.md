# Multi-Protocol 엔터프라이즈 네트워크 구축

> Frame Relay WAN 위에서 RIP · OSPF · EIGRP를 구간별로 운용하고 **재분배로 하나의 라우팅 도메인처럼** 연동한 프로젝트입니다. 스위치 구간은 VLAN · EtherChannel · MST로 이중화했습니다.

![Topology](docs/images/logical-ip.jpg)

---

## 프로젝트 개요

라우터 6대 · L3 스위치 4대 · 백본 3대로 구성된 엔터프라이즈 토폴로지를 4인 팀으로 설계·구축한 프로젝트입니다.

WAN은 **Frame Relay**(P2P · Multipoint)로, 라우팅은 **RIPv2 / OSPF 멀티 에어리어 / EIGRP**를 구간별로 나눈 뒤 경계에서 **재분배**해 전 구간이 서로 통신하도록 구성했습니다. 스위치 구간은 **VTP · 802.1Q · EtherChannel · MST**로 이중화하고, **OSPF MD5 인증 · DHCP · UDP 릴레이** 등 IOS 기능으로 마무리했습니다.

**핵심 흐름**

`BB1 → RIPv2 → (R3 재분배) → OSPF 백본(Area 0·Virtual-Link) → (SW1·R6 재분배) → EIGRP → 스위치 / BB3`

---

## 목차

| # | 문서 | 내용 |
|---|------|------|
| 01 | [구성도](docs/구성도.md) | 논리적(IP·IGP) · 물리적 구성도 |
| 02 | [Frame Relay (WAN)](docs/01-frame-relay.md) | P2P · Multipoint · DLCI 매핑 |
| 03 | [Switching](docs/23-switching.md) | VTP · Trunk · EtherChannel · VLAN · MST · SVI |
| 04 | [Routing](docs/03-routing.md) | RIP · OSPF · EIGRP · Redistribution |
| 05 | [Services & Security](docs/04-services-security.md) | OSPF MD5 인증 · DHCP · UDP 릴레이 |

---

## 기술 스택

`Cisco IOS 12.4` · `Router × 6` · `L3 Switch × 4` · `Backbone × 3`
`Frame Relay` · `P2P` · `Multipoint` · `DLCI`
`VLAN` · `VTP` · `802.1Q Trunk` · `EtherChannel` · `MST(STP)` · `UDLD`
`RIPv2` · `OSPF` · `EIGRP` · `Redistribution` · `Virtual-Link` · `Stub Area`
`OSPF MD5` · `DHCP` · `IP Helper(UDP Relay)`
