# 02. 아키텍처 상세

## Multi-Cloud Cross-Region DR Architecture

AWS–GCP 하이브리드 멀티 클라우드 기반 재해복구 시스템입니다.

![DR Architecture](../images/dr-architecture.png)


---

## 계층별 설계

### 1) 트래픽 / DNS 계층 — AWS Route 53
- **Failover 라우팅 정책**으로 Primary(서울)·Secondary(도쿄) 레코드를 구성합니다.
- **Health Check**로 서울 ALB 상태를 감시하다가, 이상 감지 시 트래픽을 도쿄로 자동 전환합니다.
- 사용자 진입점 도메인: `easiha.com`

### 2) 로드밸런싱 / 컴퓨팅 계층 — AWS (양 리전 동일 구성)
| 계층 | 구성 |
|------|------|
| **ALB** | 리전별 Application Load Balancer |
| **WEB (Public Subnet)** | WEB 서버 2대 — 외부 요청 수신 |
| **WAS (Private Subnet)** | WAS 서버 2대 — 애플리케이션 처리, 외부에 직접 노출되지 않음 |

- 도쿄(DR) 리전은 서울(Main) 리전과 **동일한 WEB/WAS/ALB 구성**으로 복제합니다.
- OS: Red Hat Enterprise Linux, 검증 애플리케이션: DVWA.

### 3) 데이터 계층 — GCP Cloud SQL (MariaDB)
- 두 AWS 리전이 **하나의 공용 DB**(GCP Cloud SQL)를 바라보게 하여, 리전 전환 후에도 데이터 연속성을 유지합니다.
- 각 리전의 WAS가 DB에 접근하려면, GCP Cloud SQL **방화벽에서 해당 리전의 퍼블릭 IP를 허용**해야 합니다.

---

## 설계 근거 (Why)

| 선택 | 이유 |
|------|------|
| **크로스 리전(서울↔도쿄)** | 하나의 재난이 두 센터를 동시에 무너뜨리지 못하도록 지리적으로 격리 |
| **Active–Passive (Failover)** | 평시 비용을 낮추면서도 장애 시 자동 전환 보장 |
| **멀티 클라우드 DB(GCP)** | 컴퓨팅(AWS)과 데이터(GCP) 계층을 분리해 벤더/리전 단일 장애 영향 축소 |
| **Route 53 Health Check** | 사람의 수동 개입 없이 장애 감지 → 전환 자동화 |

---

## 주요 리소스 참고값 (실습 기준)

| 항목 | 값 |
|------|-----|
| 도메인 | `easiha.com` |
| 서울 ALB | `dualstack.dvwa-...ap-northeast-2.elb.amazonaws.com` |
| 도쿄 ALB | `dualstack.dvwa-...ap-northeast-1.elb.amazonaws.com` |
| Route 53 레코드 | `Seoul-Main-Active`(Primary) / `Tokyo-backup-DR`(Secondary) |

---

⬅️ 이전: [01. 배경 및 개념](01-background.md) · ➡️ 다음: [03. 구축 과정](03-implementation.md)
