# 2. Terraform 구성요소

> Provider · Resource · State · Module · Variable · Backend

## 핵심 구성요소

### HCL (HashiCorp Configuration Language)
테라폼 구성 파일을 작성하는 데 사용되는 언어입니다.
**사람이 읽기 쉬우면서도 기계가 해석하기 용이하도록** 설계되었습니다.

### State (`.tfstate`)
Terraform이 **실제 인프라와 코드의 현재 상태를 매핑해 저장**하는 파일입니다.
Terraform은 이 파일을 기준으로 무엇을 바꿀지 판단합니다.
> ⚠️ **분실 시 매우 위험** — State가 없으면 기존 리소스를 인식하지 못해 중복 생성하거나 관리 불능 상태가 됩니다.

### Execution Plan
`terraform plan` 명령어를 통해 현재 인프라 상태와 구성 파일의 목표 상태를 비교하여,
어떤 리소스를 **생성 · 수정 · 삭제**할지 미리 확인합니다.

### Resource
실제로 만들어지는 인프라 단위입니다.
EC2, VPC, S3 같은 객체를 코드로 선언합니다.

### Module
여러 리소스를 묶어 **재사용 가능한 단위**로 만든 것입니다.
폴더 구조로 관리합니다.

### Backend
State 파일을 **어디에 저장할지 정의**합니다 (local, S3, Terraform Cloud 등).
> 협업 시 원격(remote) backend가 필수입니다. — State 중앙 관리, lock, 이력 추적을 제공합니다.

## 요약 표

| 요소 | 한 줄 설명 |
|------|-----------|
| HCL | 테라폼 설정 파일 작성 언어 |
| State | 실제 인프라 ↔ 코드 상태 매핑 파일 (분실 주의) |
| Execution Plan | 변경 예정 내역(생성/수정/삭제) 미리보기 |
| Resource | 실제로 만들어지는 인프라 단위 (EC2, VPC, S3…) |
| Module | 재사용 가능한 리소스 묶음 |
| Backend | State 저장 위치 정의 (협업 시 원격 필수) |

## State 파일을 왜 원격에 두는가

- **중앙 관리** — 여러 명이 같은 State를 공유해야 인프라 상태가 일관된다.
- **Lock (잠금)** — 동시에 `apply`하면 State가 깨질 수 있어, 한 번에 하나만 작업하도록 잠근다.
- **이력 추적 / 백업** — State 버전을 남겨 문제 시 복원할 수 있다.
- **민감 정보 보호** — 원격 backend는 민감 값을 암호화해 저장한다.
