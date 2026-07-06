# 04. 리소스 관리 대장

[← 목차로 돌아가기](./README.md)

프로젝트에서 생성한 전체 AWS 리소스 목록.

---

## VPC
- `HIS-PRD-VPC` — CIDR `10.250.0.0/16`, DNS 호스트 이름 활성화

## Subnet
| 이름 | 유형 |
|------|------|
| HIS-PRD-VPC-BASTION-PUB-2A | Public |
| HIS-PRD-VPC-NGINX-PUB-2A | Public |
| HIS-PRD-VPC-NGINX-PUB-2C | Public |
| HIS-PRD-VPC-TOMCAT-PRI-2A | Private |
| HIS-PRD-VPC-TOMCAT-PRI-2C | Private |
| HIS-PRD-VPC-DB-PRI-2A | Private |
| HIS-PRD-VPC-DB-PRI-2C | Private |

## Security Group
- HIS-PRD-VPC-Bastion-PUB-SG-2A
- HIS-PRD-VPC-NGINX-PUB-SG-2A / 2C
- HIS-PRD-VPC-TOMCAT-PRI-SG-2A / 2C
- HIS-PRD-VPC-DB-PRI-SG-2A / 2C
- HIS-PRD-ALB-SG (HIS-NGINX-ALB-SG)

## Internet Gateway
- `HIS-PRD-IGW`

## NAT Gateway
- `HIS-PRD-NAT-2A` (HIS-PRD-NGW-2A)

## Routing Table
- `HIS-PRD-RT-PUB` — 라우팅: IGW / 연결: NGINX-PUB-2A, 2C
- `HIS-PRD-RT-PRI` — 라우팅: NAT / 연결: TOMCAT-PRI-2A·2C, DB-PRI-2A·2C

## EC2
- HIS-PRD-VPC-Bastion-PUB-2A
- HIS-PRD-VPC-NGINX-PUB-2A / 2C
- HIS-PRD-VPC-TOMCAT-PRI-2A / 2C

## ALB
- `HIS-PRD-ALB` (HIS-NGINX-ALB)
- Target Group: `HIS-PRD-ALB-TG`

## Route 53
- 도메인: `history-cloud.store` (가비아)
- 레코드: A (`@`, `www`) → ALB 별칭

## RDS
- `HistoryDB` — MySQL 8.0.40, 단일 AZ
- Subnet Group: `HIS-PRD-VPC-DB-GROUP`
- 초기 데이터베이스: `HIS_PRD_DB`
