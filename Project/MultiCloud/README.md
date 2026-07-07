# AWS × GCP 멀티클라우드 아키텍처

> AWS에서 동작하는 웹 서비스가 GCP의 Cloud SQL을 **사설망(VPN)으로 안전하게** 사용하도록 두 클라우드를 연결한 프로젝트입니다.

![Architecture](docs/images/architecture.png)

---


## 프로젝트 개요

기존 AWS 3-Tier 웹 서비스를 단일 클라우드에 두지 않고, GCP의 Cloud SQL과 연동하는 멀티클라우드 환경으로 확장한 프로젝트입니다.
서로 다른 두 클라우드를 **HA VPN + BGP** 로 사설 연결해 하나의 내부망처럼 통신하도록 구성했으며, AWS 인프라는 **Terraform(IaC)** 으로 코드화해 재현성을 확보했습니다.

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
`AWS` · `GCP` · `Terraform`
`Site-to-Site VPN` · `HA VPN` · `IPsec` · `BGP` · `VPC Peering`
`EC2` · `ALB` · `RDS` · `Cloud SQL(MySQL)`
`Route 53` · `Private DNS` · `Bastion` · `Nginx` · `Flask`

---

