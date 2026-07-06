# 3. 워크플로우

> Write → Init → Plan → Apply → Destroy

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

### 3. Plan
코드와 실제 인프라의 차이를 계산해 무엇이 **추가 / 변경 / 삭제**될지 출력합니다.
**실제 적용은 하지 않습니다.**

### 4. Apply
`plan`에서 확인한 변경사항을 실제로 적용합니다.
`yes` 입력 또는 `-auto-approve`로 진행됩니다.

### 5. Destroy
Terraform이 관리 중인 모든 리소스를 제거합니다.

## 💡 TIP

> **`apply` 전에는 반드시 `plan`으로 변경 내역을 확인할 것.**
>
> 안정성을 위해 `plan`을 통해 변경 내역을 먼저 확인하는 것이 권장됩니다.
> 특히 운영 환경에서는 의도치 않은 리소스 삭제/변경을 미리 잡아내는 안전장치가 됩니다.
