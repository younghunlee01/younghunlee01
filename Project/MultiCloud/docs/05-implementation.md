# 05. 구현 과정
[← 목차로 돌아가기](../README.md)

전체 구축은 다음 순서로 진행했습니다.

**AWS(Terraform) → GCP 리소스 → VPN 연결 → 라우팅/피어링 → 검증**

---

## 1. AWS 인프라 (Terraform)

```
terraform init
terraform plan
terraform apply
```

`terraform apply`로 VPC부터 RDS, ALB까지 61개 리소스를 한 번에 프로비저닝했습니다.

<!-- images/terraform-apply.png -->

이후 Bastion을 통해 WAS 서버에 접속하여 Flask 애플리케이션의 DB 연결 설정을 수정했습니다.
- `MYSQL_HOST = db.cloud.local` (DB IP를 도메인으로 지정)
- 사용자·DB를 `history`로 변경 후 서비스 재시작

---

## 2. GCP 리소스

콘솔에서 다음 순서로 구성했습니다.

1. **VPC** — `history-net` 생성, 서브넷 `history-net-seoul` (`10.200.1.0/24`)
2. **방화벽** — SSH·RDP·ICMP 등 규칙 추가
3. **Instance** — `gcp-bastion` (Rocky Linux)
4. **Cloud SQL** — MySQL 8.0, 비공개 IP + PSA 연결
5. **DB/사용자** — `history` 데이터베이스와 `users` 테이블 생성

<!-- images/cloud-sql.png -->

---

## 3. VPN 연결

### GCP 측
- Cloud Router 생성 (ASN 65000)
- HA VPN Gateway `aws-vpn` 생성 (인터페이스 2개)

### AWS 측
- Customer Gateway 2개 생성 (gcp-gw1 / gcp-gw2) — IP는 GCP VPN 주소
- Virtual Private Gateway 생성 (ASN 64512) → VPC 연결
- Site-to-Site VPN 2개 생성 (gcp-vpn1 / gcp-vpn2), 각 터널의 내부 CIDR·PSK 지정

### GCP 측 마무리
- Peer VPN Gateway 생성, 인터페이스 4개에 AWS VPN 주소 지정
- VPN 터널 4개 생성 후 각 터널을 Cloud Router에 연결
- 터널마다 BGP 세션 생성 (피어 ASN = AWS의 64512, 피어 주소 = VPN 내부 IP)

<!-- images/vpn-bgp-established.png -->

BGP 세션 4개가 모두 `설정됨(established)` 상태가 되는 것을 확인했습니다.

---

## 4. 라우팅 및 피어링 (트러블슈팅)

### 문제
VPN을 모두 연결했음에도 AWS에서 Cloud SQL 접속이 실패했습니다.

### 원인
Cloud SQL은 GCP 내부에서 별도 망(`servicenetworking`)에 피어링으로 연결되어 있는데,
**VPC 피어링은 전이되지 않아** AWS가 Cloud SQL 대역(`10.255.16.0/24`)을 학습하지 못했습니다.

### 해결
1. **AWS** — 라우팅 테이블에 VPN 경로 전파(propagation) 활성화
2. **GCP** — 피어링에서 커스텀 경로 가져오기/내보내기 활성화
3. **GCP** — Cloud Router 커스텀 경로에 `10.255.16.0/24` 추가 → BGP로 AWS에 전달

<!-- images/custom-route.png -->

### 배운 점
연결이 됐다고 경로가 자동으로 도는 것은 아니며, 라우팅 전파 범위를 명시적으로 설계해야 한다는 것을 체감했습니다.

---

## 5. 검증

### DB 접속 검증
AWS Bastion에서 도메인으로 Cloud SQL에 접속했습니다.

```bash
mysql -h db.cloud.local -u history -p
# → GCP Cloud SQL(MySQL 8.0-google) 접속 성공
# → show tables; 로 users 테이블 확인
```

<!-- images/mysql-connect-success.png -->

### 웹 서비스 검증
등록 도메인으로 웹에 접속하여 회원가입 후, 해당 데이터가 GCP Cloud SQL에 저장되는 것을 확인했습니다.

- 로그인 후 사용자 정보(Client IP, Server Private IP 등) 출력
- `select * from users;` 로 저장된 계정 데이터 확인

<!-- images/dashboard.png -->

**AWS 웹 → VPN → GCP DB** 전체 흐름이 정상 동작함을 검증했습니다.

---

## 회고 및 개선점

학습 프로젝트로서 전체 흐름 검증에 집중했으며, 프로덕션 관점에서는 다음을 보완할 수 있습니다.

- VPN 사전공유키(PSK)를 테스트 값이 아닌 안전한 키로 관리
- Cloud SQL을 단일 영역이 아닌 다중 영역(고가용성) 구성으로 전환
- 클라우드 간 데이터 전송(egress) 비용을 고려한 캐싱 / DB 위치 재검토
- 모니터링·로깅 체계 추가 (CloudWatch, Cloud Monitoring)
- GCP 인프라도 Terraform으로 코드화하여 전체 재현성 확보
