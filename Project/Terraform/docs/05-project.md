# 5. 인프라 구축 상세
[← 목차로 돌아가기](../README.md)

> 디렉토리 구조 · 리소스별 코드 리뷰 · HCP Terraform 배포 흐름

> 📌 아래 `.tf` 코드는 프로젝트에서 제공된 것으로,
> 각 리소스의 역할과 설계 의도를 **직접 분석·정리**한 내용입니다.
> HCP Terraform 배포 흐름 및 도메인 연결은 **직접 수행**했습니다.

## 프로젝트 디렉토리 구조

리소스가 역할별로 `.tf` 파일로 분리되어 있습니다.

```
project/
├── provider.tf        # terraform & provider
├── vpc.tf             # VPC
├── subnet.tf          # Subnet
├── igw.tf             # Internet Gateway
├── ngw.tf             # NAT Gateway & EIP
├── rt.tf              # Routing Table & Routing
├── security_group.tf  # Security Group
├── key_pair.tf        # Key Pair
├── ec2.tf             # Bastion Instance
├── rds.tf             # Subnet Group & Parameter Group & RDS
├── s3.tf              # S3 Bucket
├── acm.tf             # ACM
├── route53.tf         # Route53 Zone
├── alb.tf             # Target Group & Listener & ALB
└── auto_scaling.tf    # Launch Template & Auto Scaling
```

## 리소스별 요약

### provider.tf
전체 프로젝트의 실행 환경을 정의합니다.
- Terraform 버전(`>= 1.0.0, < 2.0.0`)과 AWS provider(`~> 5.0`) 명시
- 배포 리전: `ap-northeast-2` (서울)

### vpc.tf
- CIDR 대역 할당: `10.250.0.0/16`
- DNS 기능 활성화(`enable_dns_support`, `enable_dns_hostnames`)로 도메인 주소 기반 통신 지원

### subnet.tf
- **퍼블릭 서브넷** 2개 — `10.250.1.0/24` (2a), `10.250.2.0/24` (2c). 공인 IP 자동 할당
- **프라이빗 서브넷** 2개 — `10.250.11.0/24` (2a), `10.250.12.0/24` (2c). 외부 직접 접근 차단
- Kubernetes(EKS) 연동용 태그 지정 (`kubernetes.io/role/elb` 등)

### igw.tf
VPC와 인터넷 간 양방향 통신 관문. 퍼블릭 자원의 인터넷 접속을 활성화합니다.

### ngw.tf
- **EIP** — NAT 게이트웨이용 고정 공인 IP 확보
- **NAT Gateway** — 프라이빗 자원의 **단방향** 인터넷 접속 지원. 내부 보안을 유지하며 외부 패치/업데이트만 허용

### rt.tf
- **퍼블릭 라우팅 테이블** — `0.0.0.0/0` → Internet Gateway
- **프라이빗 라우팅 테이블** — `0.0.0.0/0` → NAT Gateway
- 각 라우팅 테이블을 해당 서브넷(2a · 2c)에 매핑

### security_group.tf
용도별로 보안 그룹을 분리했습니다.

| 보안 그룹 | 허용 인바운드 |
|-----------|--------------|
| Bastion | ICMP, 22(SSH), 80, 443, 3306 |
| ALB | 80, 443 |
| EKS Node | ICMP, 22, 80, 443 |
| RDS | 3306 |

> 아웃바운드는 모든 포트 허용(`egress 0.0.0.0/0`).

### key_pair.tf
- RSA 2048비트 알고리즘으로 키 생성
- 공개키를 AWS에 등록(`aws_key_pair`)
- 개인키를 `.pem` 파일로 로컬 저장(`local_file`)

### ec2.tf
Bastion Host 인스턴스 구성.
- AMI 지정, 인스턴스 타입 `t3.micro`
- 퍼블릭 서브넷 배치, 공인 IP 자동 할당
- 루트 볼륨 8GB(`gp2`)

### rds.tf
- **DB 서브넷 그룹** — 프라이빗 서브넷(2a · 2c)을 그룹화
- **파라미터 그룹(MariaDB 10.11)** — `time_zone = Asia/Seoul`, 문자셋 `utf8mb4`(이모지·다국어 지원), `binlog_format = ROW`(복제/복구용)
- **RDS 인스턴스** — `db.t3.micro`, MariaDB 10.11, RDS 전용 보안 그룹 연결

### s3.tf
전 세계 유일한 이름의 S3 버킷 생성 (`history-2026-terraformproject`).

### acm.tf
- 도메인 보안을 위한 ACM 인증서 생성(`*.history-cloud.store`, DNS 검증)
- 무중단 갱신 설정(`create_before_destroy`)
- 소유권 증명을 위한 Route53 DNS 레코드 자동 등록

### route53.tf
도메인(`history-cloud.store`)의 네임서버 역할을 하고 DNS 레코드를 관리할 호스팅 영역 생성.

### alb.tf
- **ALB** — 외부 트래픽을 받아 분산하는 퍼블릭 애플리케이션 로드밸런서
- **Target Group** — 트래픽이 도달할 대상 그룹 정의 및 상태 검사(`health_check`) 설정
- **Listener** — 80포트 요청을 감지해 Target Group으로 전달

### auto_scaling.tf
- **Launch Template** — EC2 사양·AMI·키페어 지정, User Data로 부팅 시 Nginx 자동 설치 및 기본 페이지 배포
- **Auto Scaling Group** — min 2 / max 4 / desired 3, ELB 헬스체크 기반 자동 교체, Target Group 자동 등록
- 템플릿 최신 버전 연결 + `create_before_destroy`로 무중단 교체

## HCP Terraform 배포 흐름 (GitOps) — 직접 수행

제공된 `.tf` 코드를 실제로 배포하기 위해 아래 파이프라인을 직접 구성했습니다.

1. `app.terraform.io`에서 **Organization** 생성 (`history-hcp`)
2. **Workspace** 생성 — workflow는 **VCS(Version Control Workflow)** 선택
3. **GitHub 저장소 연동** (`HCP-Terraform-HISTORY`) 후 Install
4. **변수 설정** — `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`를 **Sensitive** 변수로 등록
5. **Settings 변경** — Auto-apply 활성화, Remote State Sharing 설정
6. **Run** — GitHub에 `.tf` push → HCP Terraform이 감지해 자동 `plan` · `apply`
7. **연동 확인** — GitHub push(`triggered via GitHub`)로 run이 자동 실행되고 `Applied` 확인

## 도메인 연결 (Connect)

1. Route53에서 네임서버(NS) 확인
2. 도메인 등록처(가비아)에서 네임서버를 Route53 NS로 설정
3. Route53에서 A레코드(`@`, `www`) 생성 → ALB에 별칭(Alias) 연결
4. `history-cloud.store` 접속 확인 — Nginx on EC2 정상 서비스

---

## 프로젝트 한 줄 요약

**IaC 개념을 코드로 구현(Terraform) → GitHub + HCP Terraform 연동으로 자동 배포(GitOps) → VPC부터 ALB · Auto Scaling까지 AWS 3-Tier 인프라를 코드로 배포하고 실제 도메인 서비스까지 검증한 프로젝트.**
