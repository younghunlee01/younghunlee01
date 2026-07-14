# 3. 워크플로우 & 주요 명령어
[← 목차로 돌아가기](../README.md)

> Write → Init → Plan → Apply → Destroy, 그리고 핵심 명령어와 옵션

## 기본 흐름

```
Write  →  Init  →  Plan  →  Apply  →  Destroy
```

| 단계 | 명령어 | 설명 |
|------|--------|------|
| 1. Write | — | `.tf` 파일에 인프라 정의 |
| 2. Init | `terraform init` | Provider 다운로드, backend 설정 |
| 3. Plan | `terraform plan` | 실제 변경될 내용 미리보기 |
| 4. Apply | `terraform apply` | 변경사항을 실제 인프라에 반영 |
| 5. Destroy | `terraform destroy` | 만든 리소스를 모두 제거 |

## 단계별 상세

### 1. Write
`.tf` 파일에 원하는 인프라 상태를 코드로 작성합니다.

### 2. Init
프로젝트를 처음 사용하거나 backend / provider 설정이 바뀌었을 때 실행합니다.
`.terraform/` 디렉토리와 lock 파일이 생성됩니다.
- `-upgrade` : provider 버전 업그레이드
- `-reconfigure` : backend 재설정

### 3. Plan
코드와 실제 인프라의 차이를 계산해 무엇이 **추가 / 변경 / 삭제**될지 출력합니다.
**실제 적용은 하지 않습니다.**
- `-out=plan.tfplan` : 결과를 파일로 저장
- `-var-file=prod.tfvars` : 변수 파일 지정

### 4. Apply
`plan`에서 확인한 변경사항을 실제로 적용합니다.
`yes` 입력 또는 `-auto-approve`로 진행됩니다.
- `-auto-approve` : 확인 단계 생략
- `plan.tfplan` : `plan` 결과 파일 적용

### 5. Destroy
Terraform이 관리 중인 모든 리소스를 제거합니다.

## 💡 TIP

> **`apply` 전에는 반드시 `plan`으로 변경 내역을 확인할 것.**
>
> 안정성을 위해 `plan`을 통해 변경 내역을 먼저 확인하는 것이 권장됩니다.
> 특히 운영 환경에서는 의도치 않은 리소스 삭제/변경을 미리 잡아내는 안전장치가 됩니다.

---

## 전체 명령어 정리

### 핵심 명령어 (5종)

| 명령어 | 설명 |
|--------|------|
| `terraform init` | 프로젝트 초기화 / provider 다운로드 |
| `terraform validate` | 문법 / 설정 검증 |
| `terraform plan` | 변경 예정 사항 미리보기 |
| `terraform apply` | 실제 인프라에 변경 적용 |
| `terraform destroy` | 관리 중인 모든 리소스 제거 |

### 기타 명령어 (4종)

| 명령어 | 설명 |
|--------|------|
| `terraform fmt` | 코드 포매팅 (들여쓰기 정리) |
| `terraform show` | 인프라의 상세 정보 확인 |
| `terraform output` | output 블록의 값 출력 |
| `terraform import` | 기존 인프라 리소스를 테라폼으로 가져와 관리 대상으로 편입 |

### 가장 많이 쓰는 흐름

```bash
# 1. 작업 디렉토리 초기화
terraform init

# 2. 문법 검증
terraform validate

# 3. 변경사항 미리보기
terraform plan

# 4. 변경사항 적용
terraform apply

# 5. 리소스 제거
terraform destroy
```

➡️ 다음: [04. CI/CD & GitOps](04-cicd.md)
