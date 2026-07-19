# AWS 환경에서의 3-Tier Web Service 아키텍처 구축




## 프로젝트 개요

AWS 클라우드 환경에서 3-Tier 웹 서비스 아키텍처를 직접 설계·구축한 프로젝트로, 각 계층을 독립적으로 분리하여 확장성·가용성·보안을 확보하는 것을 목표로 했습니다.

- **3-Tier 아키텍처 구현** — Nginx(Web), Tomcat(WAS), RDS(DB)를 독립된 계층으로 구성하고 계층 상호 간 데이터를 연동
- **서비스 부하 분산** — ALB(Application Load Balancer)로 트래픽을 분산하여 특정 서버 장애가 서비스 전체로 확산되지 않도록 구성
- **네트워크 영역 분리** — Public / Private 서브넷을 용도별로 분리하고, 보안 그룹(SG)과 Bastion Host를 활용해 외부 노출을 최소화

---

## 목차


1. **[인프라 구성도](./docs/01-diagram.md)** — 전체 아키텍처 다이어그램 및 네트워크 흐름
2. **[프로젝트 개요](./docs/02-overview.md)** — 목표, 3-Tier 개념, 온프레미스 → AWS 인프라 전환
3. **[핵심 기술 및 아키텍처 이론](./docs/03-architecture.md)** — AWS 주요 컴포넌트 개념, 3-Tier Architecture 이론
4. **[리소스 관리 대장](./docs/04-resources.md)** — 생성한 모든 AWS 리소스 목록
5. **[구축 내용](./docs/05-build.md)** — 네트워크 → EC2 → ELB → 서비스 연동 → RDS 단계별 구축
6. **[아키텍처 설계 고안점](./docs/06-design-decisions.md)** — 설계 결정 사항과 트레이드오프

---

## 기술 스택

`AWS VPC` `EC2` `ALB` `RDS(MySQL)` `Route 53` `NAT Gateway` `Internet Gateway`
`Nginx` `Apache Tomcat` `MySQL` `Ubuntu 24.04` `Bastion Host`

---

## 최종 결과

- `history-cloud.store` 접속 시 ALB → Nginx → Tomcat → RDS 로 이어지는 3-Tier 요청 흐름 정상 동작
- 게시판(3Tier Board) 애플리케이션에서 DB 연동(게시글 목록/작성) 확인 완료
