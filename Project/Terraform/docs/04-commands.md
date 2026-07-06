# 4. 주요 명령어

> 핵심 5종 + 기타 4종, 그리고 자주 쓰는 옵션 정리

## 핵심 명령어 (5종)

| 명령어 | 설명 |
|--------|------|
| `terraform init` | 프로젝트 초기화 / provider 다운로드 |
| `terraform validate` | 문법 / 설정 검증 |
| `terraform plan` | 변경 예정 사항 미리보기 |
| `terraform apply` | 실제 인프라에 변경 적용 |
| `terraform destroy` | 관리 중인 모든 리소스 제거 |

## 기타 명령어 (4종)

| 명령어 | 설명 |
|--------|------|
| `terraform fmt` | 코드 포매팅 (들여쓰기 정리) |
| `terraform show` | 인프라의 상세 정보 확인 |
| `terraform output` | output 블록의 값 출력 |
| `terraform import` | 기존 인프라 리소스를 테라폼으로 가져와 관리 대상으로 편입 |

## 가장 많이 쓰는 흐름

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

## 명령어별 주요 옵션

### `terraform init`
- `-upgrade` : provider 버전 업그레이드
- `-reconfigure` : backend 재설정

> 프로젝트를 처음 사용하거나 backend / provider 설정이 바뀌었을 때 실행합니다.
> `.terraform/` 디렉토리와 lock 파일이 생성됩니다.

### `terraform plan`
- `-out=plan.tfplan` : 결과를 파일로 저장
- `-var-file=prod.tfvars` : 변수 파일 지정

> 코드와 실제 인프라의 차이를 계산해 무엇이 추가/변경/삭제될지 출력합니다. 실제 적용은 하지 않습니다.

### `terraform apply`
- `-auto-approve` : 확인 단계 생략
- `plan.tfplan` : `plan` 결과 파일 적용

> `plan`에서 본 변경사항을 실제로 적용합니다. `yes` 입력 또는 `-auto-approve`로 진행됩니다.
