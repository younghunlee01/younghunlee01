# AWS × GCP 멀티클라우드 아키텍처

> AWS에서 동작하는 웹 서비스가 GCP의 Cloud SQL을 **사설망(VPN)으로 안전하게** 사용하도록 두 클라우드를 연결한 프로젝트입니다.

![Architecture](docs/images/architecture.png)

---

## 한 줄 요약

단일 클라우드를 넘어, AWS의 3-Tier 웹 서비스와 GCP의 Cloud SQL을 **HA VPN + BGP** 로 사설 연결했습니다.
AWS 인프라는 **Terraform(IaC)** 으로 코드화하고, 두 클라우드를 하나의 내부망처럼 통신하도록 구성했습니다.

**핵심 흐름**
`애플리케이션 → db.cloud.local (사설 DNS) → 비공개 IP 해석 → VPN 터널 → GCP Cloud SQL`

---

## 목차

| # | 문서 | 내용 |
|---|------|------|
| 01 | [아키텍처 구조](docs/01-architecture.md) | 전체 구조 다이어그램과 트래픽 흐름 |
| 02 | [AWS 생성 리소스](docs/02-aws-resources.md) | Terraform으로 구축한 AWS 리소스 |
| 03 | [GCP 생성 리소스](docs/03-gcp-resources.md) | 콘솔로 구축한 GCP 리소스 |
| 04 | [핵심 개념](docs/04-concepts.md) | 멀티클라우드 · VPN · BGP · DNS |
| 05 | [구현 과정](docs/05-implementation.md) | 단계별 구축 과정과 트러블슈팅 |

---

## 기술 스택

| 구분 | 사용 기술 |
|------|-----------|
| Cloud | AWS, GCP |
| IaC | Terraform (HCP Terraform) |
| Network | Site-to-Site VPN, HA VPN, BGP, IPsec, VPC Peering |
| Compute | EC2, GCE, ALB / NLB |
| Database | GCP Cloud SQL (MySQL 8.0) |
| DNS | AWS Route 53 (Private Hosted Zone) |
| App | Nginx, Flask |

---

