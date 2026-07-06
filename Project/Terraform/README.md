# Terraform 프로젝트

**제공된 Terraform(HCL) 코드를 분석**하고, **HCP Terraform + GitHub 연동**을 통해
GitOps 자동 배포(CI/CD) 파이프라인을 직접 구성한 프로젝트입니다.

코드 push → 자동 `plan` / `apply` → VPC부터 ALB · Auto Scaling까지 코드로 배포하고,
최종적으로 실제 도메인(`history-cloud.store`) 서비스까지 검증했습니다.

> 📌 인프라 정의 코드(`.tf`)는 프로젝트에서 제공되었으며,
> 본 문서는 **그 코드를 리소스별로 분석·정리**하고 **배포 체계 구성 과정을 직접 수행**한 기록입니다.

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 목표 | 제공된 IaC 코드 분석 및 GitOps 기반 AWS 3-Tier 인프라 배포 |
| 직접 수행 | 코드 분석 · HCP Terraform ↔ GitHub 연동 · 변수 설정 · 배포(apply) · 도메인 연결 |
| 사용 기술 | Terraform (HCL), HCP Terraform, AWS, GitHub |
| 대상 리소스 | VPC, Subnet, IGW, NAT GW, RDS, ALB, Auto Scaling, Route53, ACM |
| 배포 방식 | GitHub push → HCP Terraform 자동 plan · apply |
| 결과 | `history-cloud.store` 도메인 서비스 정상 동작 확인 |

---

## 목차

1. [IaC 개념](./docs/01-concept.md) — Infrastructure as Code란 무엇인가
2. [Terraform 구성요소](./docs/02-components.md) — HCL · State · Resource · Module · Backend
3. [워크플로우 & 주요 명령어](./docs/03-workflow.md) — Write → Init → Plan → Apply → Destroy 와 핵심 명령어
4. [CI/CD & GitOps](./docs/04-cicd.md) — 지속적 통합/배포와 HCP Terraform 기반 GitOps
5. [인프라 구축 상세](./docs/05-project.md) — 디렉토리 구조 · 리소스별 코드 · HCP 배포 흐름

---

## 아키텍처 한눈에 보기

```
[사용자] → Route53 (DNS) → ALB (트래픽 분산)
                              │
                    ┌─────────┴─────────┐
              Public Subnet 2a     Public Subnet 2c
              (Bastion, ALB, NAT)  (ALB)
                    │                   │
              Private Subnet 2a   Private Subnet 2c
              (App, RDS)          (App, RDS)
```

- **네트워크**: VPC(`10.250.0.0/16`) 내 멀티 AZ(2a · 2c) 퍼블릭/프라이빗 서브넷 구성
- **보안**: 용도별 보안 그룹 분리, Bastion 경유 접근으로 외부 노출 최소화
- **확장성**: Launch Template + Auto Scaling으로 인스턴스 자동 증감
- **자동화**: HCP Terraform ↔ GitHub 연동으로 GitOps 배포
