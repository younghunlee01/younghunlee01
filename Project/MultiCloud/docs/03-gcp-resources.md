# 03. GCP 생성 리소스
[← 목차로 돌아가기](../README.md)

GCP는 **콘솔로 단계별로 구성**했습니다. 네트워크 → HA VPN → BGP → Cloud SQL → 피어링 순으로 진행했습니다.

---

## 네트워크
- VPC `historynet`
- 서브넷 `historynet-seoul` (`10.200.1.0/24`)
- 방화벽 규칙

## VPN · 라우팅
- Cloud Router (ASN 65000)
- HA VPN Gateway (`aws-vpn`)
- Peer Gateway (`aws-gw`)
- VPN 터널 ×4
- BGP 세션 ×4

## Instance
- gcp-bastion (Rocky Linux)

## DB
- Cloud SQL MySQL 8.0
- 비공개 IP 전용
- DB / user: `history`

## 피어링
- `servicenetworking` 비공개 서비스 액세스
- 커스텀 경로 가져오기 / 내보내기

---

## Cloud SQL 상세 구성

| 항목 | 값 |
|------|-----|
| DB 버전 | MySQL 8.0 |
| 인스턴스 ID | history-db |
| 리전 | asia-northeast3 (서울) |
| 가용성 | 단일 영역 |
| 머신 | 공용 - 전용 코어 (vCPU 1, 3.75GB) |
| 저장소 | SSD 10GB |
| 연결 | 비공개 IP + PSA(비공개 서비스 액세스) |

---

## 정리

> GCP는 콘솔로 단계별 구성 — 네트워크 · HA VPN · BGP · Cloud SQL · 피어링

➡️ 다음: [04. 핵심 개념](04-concepts.md)
