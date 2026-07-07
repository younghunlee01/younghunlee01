# 02. Frame Relay (WAN)
[← 목차로 돌아가기](../README.md)

라우터 6대의 Serial 인터페이스를 Frame Relay 클라우드로 연결합니다. 공통적으로 `inverse-arp`를 비활성화하고 서브인터페이스 방식으로 IP를 구성하며, 자기 자신을 포함한 모든 이웃으로 ping이 되도록 `frame-relay map` / `interface-dlci`를 설정합니다.

## 구간 요약

| 구간 | 방식 | 네트워크 | DLCI |
|------|------|----------|------|
| R1 ↔ R3 | Point-to-Point | `14.14.13.0/24` | 103 / 301 |
| R2 ↔ R6 | Point-to-Point | `14.14.26.0/24` | 206 / 602 |
| R2 ↔ R4 ↔ R5 | Multipoint | `14.14.245.0/24` | 205 / 405 / 502·504 |

> Multipoint 구간(245망)은 OSPF `network broadcast`로 동작하며, R4·R5는 DR/BDR 선출에서 빠지도록 `ip ospf priority 0`을 적용합니다.

---

## R1 (P2P)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.103 point-to-point
 ip address 14.14.13.1 255.255.255.0
 frame-relay interface-dlci 103
```

## R3 (P2P)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.301 point-to-point
 ip address 14.14.13.3 255.255.255.0
 frame-relay interface-dlci 301
```

## R2 (Multipoint + P2P)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.245 multipoint
 ip address 14.14.245.2 255.255.255.0
 frame-relay map ip 14.14.245.5 205 broadcast
 frame-relay map ip 14.14.245.4 205 broadcast
 frame-relay map ip 14.14.245.2 205
 ip ospf network broadcast
interface s1/0.206 point-to-point
 ip address 14.14.26.2 255.255.255.0
 frame-relay interface-dlci 206
```

## R4 (Multipoint)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.245 multipoint
 ip address 14.14.245.4 255.255.255.0
 frame-relay map ip 14.14.245.5 405 broadcast
 frame-relay map ip 14.14.245.2 405 broadcast
 frame-relay map ip 14.14.245.4 405
 ip ospf network broadcast
```

## R5 (Multipoint)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.245 multipoint
 ip address 14.14.245.5 255.255.255.0
 frame-relay map ip 14.14.245.2 502 broadcast
 frame-relay map ip 14.14.245.4 504 broadcast
 frame-relay map ip 14.14.245.5 502
 ip ospf network broadcast
```

## R6 (P2P)

```
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown
interface s1/0.602 point-to-point
 ip address 14.14.26.6 255.255.255.0
 frame-relay interface-dlci 602
```
