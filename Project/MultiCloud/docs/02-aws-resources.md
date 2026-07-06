# 02. AWS 생성 리소스

AWS 인프라는 대부분 **Terraform으로 코드화**하여 재현성과 버전 관리를 확보했으며, VPN 부분만 콘솔로 보완했습니다.

[← 목차로 돌아가기](../README.md)
---

## Terraform (IaC)

### 네트워크
- VPC `historynet` (`10.0.0.0/16`)
- IGW (인터넷 게이트웨이)
- 서브넷 8개 (public · web · was · db × 2 AZ)
- NAT Gateway 2개
- EIP 3개
- 라우팅 테이블 3개

### Instance
- Bastion (`10.0.0.10`)
- Web a / b (Nginx 리버스 프록시)
- WAS a / b (Flask)
- EC2 총 5대

### DB · 로드밸런서
- RDS MySQL 8.0 (+ Subnet Group)
- ALB (web-alb, 공인)
- NLB (was-nlb, 내부)

### 보안 · 키 · DNS
- 보안 그룹 7종
- Key Pair (`historykey`)
- Route 53 `cloud.local` + db / was 레코드

---

## 콘솔 (VPN)

### VPN
- Customer Gateway ×2 (gcp-gw1 / gcp-gw2)
- Virtual Private Gateway
- Site-to-Site VPN ×2 (터널 4)

### 라우팅
- 라우팅 테이블 전파(propagation) 활성화

### DNS
- `db.cloud.local` CNAME → A 레코드 변경 (검증 단계)

---

## Terraform 파일 구조

```
multi-cloud/
├── default_sg.tf
├── ec2.tf
├── gw.tf
├── key.tf
├── output.tf
├── providers.tf
├── rds.tf
├── router53.tf
├── sg.tf
├── subnet.tf
├── variables.tf
├── vpc.tf
├── was-lb.tf
└── web-lb.tf
```

리소스별로 `.tf` 파일을 분리하여 관리했습니다.

---

## 핵심

> AWS 인프라 대부분을 Terraform으로 코드화해 재현성·버전관리를 확보하고, VPN만 콘솔로 보완했습니다.
