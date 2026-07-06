# 4. CI/CD & GitOps
[← 목차로 돌아가기](../README.md)

> 지속적 통합/배포 개념과 HCP Terraform 기반 GitOps

## CI/CD란?

| 구분 | 의미 |
|------|------|
| **CI** (Continuous Integration, 지속적 통합) | 코드 변경 사항을 지속적으로 통합하고 검증하는 과정 |
| **CD** (Continuous Delivery / Deployment, 지속적 제공/배포) | 코드 변경의 통합·테스트·제공을 나타내는 프로세스 |

```
[ CI ]                          [ CD ]
BUILD → TEST → MERGE   →   RELEASE → DEPLOY
                          (Delivery)  (Deployment)
```

> **Continuous Delivery vs Deployment**
> - 지속적 **제공(Delivery)** — 자동 프로덕션 배포 기능이 **없다**. 릴리스까지는 자동, 배포는 수동 승인.
> - 지속적 **배포(Deployment)** — 업데이트를 프로덕션 환경에 **자동으로 릴리스**한다.

## CI/CD 효과

- **개발 생산성 향상** — 수동으로 코드를 통합하고 서버에 배포하는 반복 작업이 사라진다.
- **빠른 문제 해결** — 문제가 발생했을 때 어떤 코드에서 에러가 났는지 빠르게 파악하고 대응할 수 있다.
- **높은 신뢰성** — 자동화된 테스트를 거쳐 휴먼 에러를 줄이고 안정적인 소프트웨어 버전을 유지할 수 있다.

## 주요 CI/CD 도구

- GitHub Actions
- GitLab CI/CD
- Jenkins
- CircleCI
- ArgoCD

---

## HCP Terraform 기반 GitOps CD

이 프로젝트에서는 **HCP Terraform의 VCS(Version Control System) 연동** 기능을 활용해
인프라의 선언적 관리와 자동 배포(GitOps) 체계를 구축했습니다.

### GitOps란?

Git을 **단일 진실 공급원(Single Source of Truth)** 으로 삼아,
**코드 변경이 곧 인프라 변경**이 되는 방식입니다.
GitHub에 `.tf`를 push하면 HCP Terraform이 이를 감지해 자동으로 `plan` · `apply`를 실행합니다.

```
[개발자] → GitHub push → HCP Terraform (VCS 연동)
                              │ 자동 plan · apply
                              ▼
                        AWS 인프라 반영
```

### 도입 효과

- **배포 자동화** — 인프라가 자동 업데이트되어 운영 효율성 증대
- **인프라 가시성 및 협업** — 현재 인프라 상태를 확인하고 변경 이력 추적 가능
- **안전성** — State 파일 유실 방지 및 충돌 방지

> 실제 배포 파이프라인을 구성한 상세 과정은 [5. 인프라 구축 상세](./05-project.md)의
> *HCP Terraform 배포 흐름* 섹션을 참고하세요.
