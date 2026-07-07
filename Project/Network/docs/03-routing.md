# 03. Routing (IP IGP Protocols)
[← 목차로 돌아가기](../README.md)

RIPv2 / OSPF(멀티 에어리어) / EIGRP를 구간별로 운용하고, 경계에서 재분배로 연동합니다. 설정 커맨드는 원문 그대로이며, 라우터/스위치별로 모두 기재했습니다.

---

## 3.1 RIPv2

R1 ↔ R3 구간(BB1 연동). `passive-interface default` 후 필요한 이웃만 지정.

```
! R1
router rip
 version 2
 no auto-summary
 network 150.1.0.0
 network 14.0.0.0
 passive-interface default
 neighbor 150.1.14.254
 neighbor 14.14.13.3

! R3
router rip
 version 2
 no auto-summary
 network 14.0.0.0
 passive-interface default
 neighbor 14.14.13.1
```

---

## 3.2 EIGRP

### EIGRP YY (AS 14) — 스위치 백본

```
! SW1
ip routing
router eigrp 14
 no auto-summary
 network 14.14.90.1 0.0.0.0

! SW2
ip routing
router eigrp 14
 no auto-summary
 network 14.14.8.8 0.0.0.0
 network 14.14.90.2 0.0.0.0

! SW3
ip routing
router eigrp 14
 no auto-summary
 network 14.14.9.9 0.0.0.0
 network 14.14.90.3 0.0.0.0

! SW4
ip routing
router eigrp 14
 no auto-summary
 network 14.14.10.10 0.0.0.0
 network 14.14.90.4 0.0.0.0
```

### EIGRP 100 — R6 (BB3)

```
! R6
router eigrp 100
 no auto-summary
 network 150.3.14.1 0.0.0.0
 eigrp stub redistribute
```

---

## 3.3 OSPF (Process 14)

### Area 0 (백본)

R2·R4·R5(FR 245망) + SW1·R6(36망). 245 멀티포인트 구간은 priority로 DR/BDR 제어.

```
! R2
router ospf 14
 network 14.14.245.2 0.0.0.0 area 0
interface s1/0.245
 ip ospf priority 0

! R4
router ospf 14
 network 14.14.245.4 0.0.0.0 area 0
 network 14.14.4.4 0.0.0.0 area 0
interface s1/0.245
 ip ospf priority 0

! R5
router ospf 14
 network 14.14.245.5 0.0.0.0 area 0
 neighbor 14.14.245.2
 neighbor 14.14.245.4

! SW1
router ospf 14
 network 14.14.36.7 0.0.0.0 area 0

! R6
router ospf 14
 network 14.14.36.6 0.0.0.0 area 0
```

### Area 26 (Virtual-Link)

R2 ↔ R6 구간을 Area 26으로 두고, 백본과 직접 붙지 않으므로 **Virtual-Link**로 Area 0에 연결.

```
! R2
router ospf 14
 network 14.14.62.2 0.0.0.0 area 26
 network 14.14.26.2 0.0.0.0 area 26
 area 26 virtual-link 14.14.6.6
interface s1/0.206
 ip ospf network point-to-point
interface e0/0
 ip ospf network point-to-point

! R6
router ospf 14
 network 14.14.62.6 0.0.0.0 area 26
 network 14.14.26.6 0.0.0.0 area 26
 network 14.14.6.6 0.0.0.0 area 26
 area 26 virtual-link 14.14.2.2
interface s1/0.602
 ip ospf network point-to-point
interface e0/1.22
 ip ospf network point-to-point
```

### Area 3 (R3 ↔ SW1)

```
! R3
router ospf 14
 network 14.14.3.3 0.0.0.0 area 3
 network 14.14.33.3 0.0.0.0 area 3

! SW1
router ospf 14
 network 14.14.33.7 0.0.0.0 area 3
 network 14.14.7.7 0.0.0.0 area 3
```

### Area 4 (R2, Totally Stubby)

```
! R2
router ospf 14
 network 14.14.2.2 0.0.0.0 area 4
 network 14.14.20.2 0.0.0.0 area 4
 area 4 stub no-summary
```

### Area 5 (R5, Stub)

```
! R5
router ospf 14
 network 14.14.55.5 0.0.0.0 area 5
 network 14.14.5.5 0.0.0.0 area 5
 area 5 stub
```

### Loopback 처리

Loopback이 `/32`로 광고되지 않도록 P2P 네트워크 타입 적용.

```
! R1~R6
interface lo0
 ip ospf network point-to-point

! SW1~SW4
interface lo0
 ip ospf network point-to-point
```

---

## 3.4 재분배 (Redistribution)

| 경계 | 구간 |
|------|------|
| R3 | RIP ↔ OSPF |
| SW1 | OSPF ↔ EIGRP YY |
| R6 | OSPF ↔ EIGRP 100 |

```
! R3 (RIP - OSPF)
router rip
 redistribute ospf 14 metric 3
router ospf 14
 redistribute rip subnets

! SW1 (OSPF - EIGRP YY)
router eigrp 14
 redistribute ospf 14 metric 1 1 1 1 1
router ospf 14
 redistribute eigrp 14 subnets

! R6 (OSPF - EIGRP 100)
router ospf 14
 redistribute eigrp 100 subnets
router eigrp 100
 redistribute ospf 14 metric 1 1 1 1 1
 eigrp stub redistribute
```

---

## 3.5 검증 결과 (요약)

`show ip route` 출력 기준.

- **SW1** : EIGRP(`D`)로 다른 스위치 SVI(`14.14.8/9/10.0`) 학습, OSPF(`O`, `O IA`)로 백본/에어리어 간 경로 학습.
- **R3** : RIP(`R`)로 R1·BB1(`150.1.14.0`), OSPF(`O`, `O IA`)로 나머지 학습.
- **R6** : OSPF(`O IA`) + 직결(`C`), BB3망(`150.3.14.0/24`) 직결 확인.
- **R1** : RIP 재분배 경로(`R`)로 전체 `14.x`망 학습.
- **R5** : 외부 경로 `O E2` / Inter-Area `O IA`로 재분배 결과 확인.
- Loopback이 라우팅 테이블에 `/24`로 표기됨 (= P2P 적용 정상).
