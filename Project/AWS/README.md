# AWS 환경에서의 3-Tier Web Service 아키텍처 구축

> Nginx(Web) · Tomcat(WAS) · RDS(DB)를 독립 계층으로 분리하고, ALB·Bastion·VPC 서브넷 분리를 통해 가용성과 보안을 확보한 3-Tier 웹 서비스 인프라 구축 프로젝트


---

## 프로젝트 요약

| 항목 | 내용 |
|------|------|
| 아키텍처 | 3-Tier (Presentation / Application / Data) |
| Web Tier | Nginx (Public Subnet, Reverse Proxy) |
| WAS Tier | Apache Tomcat 9.0.108 (Private Subnet) |
| DB Tier | Amazon RDS for MySQL 8.0.40 (Private Subnet) |
| 부하 분산 | Application Load Balancer (ALB) |
| 가용 영역 | ap-northeast-2a / 2c (Multi-AZ 구성) |
| 보안 | Security Group, NACL, Bastion Host |
| DNS | Route 53 (가비아 도메인 연동) |

---

## 목차 (Documentation)

각 문서는 아래 순서대로 프로젝트 진행 흐름을 담고 있습니다.

1. **[프로젝트 개요](./01-overview.md)** — 목표, 3-Tier 개념, 온프레미스 → AWS 인프라 전환
2. **[핵심 기술 및 아키텍처 이론](./02-architecture.md)** — AWS 주요 컴포넌트 개념, 3-Tier Architecture 이론
3. **[인프라 구성도](./03-diagram.md)** — 전체 아키텍처 다이어그램 및 네트워크 흐름
4. **[리소스 관리 대장](./04-resources.md)** — 생성한 모든 AWS 리소스 목록
5. **[구축 내용](./05-build.md)** — 네트워크 → EC2 → ELB → 서비스 연동 → RDS 단계별 구축
6. **[아키텍처 설계 고안점](./06-design-decisions.md)** — 설계 결정 사항과 트레이드오프

---

## 기술 스택

`AWS VPC` `EC2` `ALB` `RDS(MySQL)` `Route 53` `NAT Gateway` `Internet Gateway`
`Nginx` `Apache Tomcat` `MySQL` `Ubuntu 24.04` `Bastion Host`

---

## 최종 결과

- `history-cloud.store` 접속 시 ALB → Nginx → Tomcat → RDS 로 이어지는 3-Tier 요청 흐름 정상 동작
- 게시판(3Tier Board) 애플리케이션에서 DB 연동(게시글 목록/작성) 확인 완료
