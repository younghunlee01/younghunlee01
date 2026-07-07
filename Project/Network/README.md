# HISTORY — Network Project

엔터프라이즈 환경을 가정한 **Switching + Multi-Protocol Routing 통합 구축 프로젝트**입니다.
라우터 6대 · L3 스위치 4대 · 백본 3대로 구성된 토폴로지 위에 Frame Relay WAN, VLAN/STP/EtherChannel 기반 스위칭, RIP·OSPF·EIGRP 재분배, 그리고 DHCP·OSPF 인증 등 IOS 보안 기능을 종합적으로 설계·구현했습니다.

> 📄 **구성도(논리적 IP / IGP · 물리적)** 는 [`docs/구성도.md`](docs/구성도.md) 에서 확인할 수 있습니다.
>
> 📑 **상세 설정 문서** : [Frame Relay](docs/01-frame-relay.md) · [Switching](docs/02-switching.md) · [Routing](docs/03-routing.md) · [Services & Security](docs/04-services-security.md)

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | HISTORY Network Project |
| 팀 구성 | 김동진 · 이영훈 · 장규혁 · 정성현 (4인) |
| 장비 규모 | Router × 6, L3 Switch × 4, Backbone × 3 |
| 플랫폼 | Cisco IOS 12.4 |
| 핵심 영역 | Frame Relay · VLAN/STP/EtherChannel · OSPF/EIGRP/RIP · Redistribution · IOS Security |

**주소 체계** : Class B `YY.YY.X.0/16` (YY = Rack 번호, X = 라우터 번호) — 본 문서는 Rack `14` 기준 `14.14.X.X`.

---

## 아키텍처 요약

- **WAN** : R1~R6를 Frame Relay 클라우드로 연결. Point-to-Point / Multipoint 서브인터페이스와 DLCI 매핑으로 PVC 구성.
- **LAN / Switching** : SW1~SW4를 EtherChannel·MST로 묶고, VTP로 VLAN 정보를 일괄 배포.
- **Routing** : 구간별로 RIPv2 / OSPF(멀티 에어리어) / EIGRP를 나누어 운용하고 경계에서 재분배로 연동.
- **Services / Security** : OSPF MD5 인증, DHCP 서버, UDP 브로드캐스트 릴레이(ip helper-address) 구성.

전체 토폴로지는 [구성도 문서](docs/구성도.md)를 참고하세요.

---

## 1. Frame Relay (WAN)

`inverse-arp` 비활성화 후 서브인터페이스 방식으로 IP를 구성하고, 자기 자신을 포함한 모든 이웃으로 ping이 되도록 설정.

| 구간 | 방식 | 네트워크 |
|------|------|----------|
| R1 ↔ R3 | Point-to-Point (DLCI 103/301) | `14.14.13.0/24` |
| R2 ↔ R6 | Point-to-Point (DLCI 206/602) | `14.14.26.0/24` |
| R2 ↔ R4 ↔ R5 | Multipoint (`frame-relay map`) | `14.14.245.0/24` |

Multipoint 구간은 OSPF `network broadcast` + `priority` 조정으로 DR/BDR을 제어.

---

## 2. Switching (Bridging & Switching)

| 기능 | 구현 내용 |
|------|-----------|
| **VTP** | SW1 = Server, SW2~SW4 = Client / Domain `history.com`, VTPv2 |
| **Trunk** | dot1q 트렁크, `allowed vlan`으로 필요 VLAN만 전달 (R6–SW2 등) |
| **EtherChannel** | `channel-group ... mode on` (Po12/13/24/34), load-balance `src-dst-ip` |
| **VLAN** | 11·12·13(BB), 21·22·23(A/B/C), 50·79(Customer), 100(Switches) |
| **STP (MST)** | Name `HISTORY` / Rev 1 — Inst1(11,21)·Inst2(100)·Inst3(12), 스위치별 root primary 분산 |
| **UDLD** | SW3–SW4 구간 `udld port aggressive` |
| **기타** | VLAN 100 SVI(`14.14.90.0/24`), MAC aging-time 조정 |

---

## 3. Routing (IP IGP Protocols)

### 라우팅 도메인
- **RIPv2** : R1 ↔ R3 (BB1 연동), `no auto-summary` + `passive-interface` + `neighbor`
- **OSPF (Process 14)** : 멀티 에어리어
  - Area 0(백본), Area 3, Area 4(Totally Stubby), Area 5(Stub)
  - **Area 26** : Virtual-Link로 백본과 논리적 연결
- **EIGRP YY (AS 14)** : SW1~SW4 + R6 스위치 백본
- **EIGRP 100** : R6–BB3, `eigrp stub redistribute`

### 재분배 (Redistribution)
| 경계 | 구간 |
|------|------|
| R3 | RIP ↔ OSPF |
| SW1 | OSPF ↔ EIGRP YY |
| R6 | OSPF ↔ EIGRP 100 |

> Loopback이 라우팅 테이블에 `/32`로 노출되지 않도록 `ip ospf network point-to-point` 적용.

---

## 4. Services & Security (IOS Features)

- **OSPF 인증** : Area 0 / Virtual-Link 구간 MD5(`message-digest`) 인증 (key `history`)
- **DHCP** : R6에 `14.14.62.0/24` 풀 구성 (excluded-address, DNS, default-router 이중화, lease infinite)
- **UDP Broadcast 관리** : `ip forward-protocol` + `ip helper-address`로 브로드캐스트 릴레이

---

## 5. 검증 결과

- `show ip route` 로 R1·R3·R5·R6·SW1의 라우팅 테이블 및 재분배 경로 확인 (RIP `R` / OSPF `O·O IA·O E2` / EIGRP `D` 코드 정상 학습)
- `show spanning-tree mst configuration` / `show spanning-tree vlan` 으로 MST 인스턴스 매핑 및 root 스위치 분산 확인
- `show etherchannel summary` 로 Port-channel(Po12/Po13 등) `SU` 상태 확인
- `show vtp status`, `show vlan brief`, `show ip interface brief` 로 VLAN 배포 및 SVI 정상 동작 확인

각 항목의 전체 설정 커맨드는 [`docs/`](docs/) 하위 문서에 정리되어 있습니다.

---

## 디렉토리 구조

```
Project/Network/
├── README.md                   # 프로젝트 개요 (현재 문서)
└── docs/
    ├── 구성도.md                # 논리적(IP/IGP) · 물리적 구성도
    ├── 01-frame-relay.md        # WAN (Frame Relay)
    ├── 02-switching.md          # VTP·Trunk·EtherChannel·VLAN·MST·UDLD·SVI
    ├── 03-routing.md            # RIP·OSPF·EIGRP·Redistribution
    ├── 04-services-security.md  # OSPF 인증·DHCP·UDP 릴레이
    └── images/
        ├── logical-ip.jpg
        ├── logical-igp.jpg
        └── physical.jpg
```
