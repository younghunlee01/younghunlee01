# 05. 구축 내용

[← 목차로 돌아가기](./README.md)

네트워크 → EC2 → ELB → 서비스 연동 → RDS 순서로 진행한 단계별 구축 내용.

> 📌 각 단계의 콘솔 스크린샷은 `./images/` 폴더에 업로드한 뒤 해당 위치에 삽입하세요.

---

## 1) 네트워크 환경 구성

### VPC 생성
- 이름: `HIS-PRD-VPC` / CIDR: `10.250.0.0/16` / DNS 호스트 이름 활성화

### Subnet (Public & Private)
| 그룹 | 2A CIDR | 2C CIDR |
|------|---------|---------|
| NGINX-PUB | 10.250.1.0/24 | 10.250.11.0/24 |
| TOMCAT-PRI | 10.250.2.0/24 | 10.250.12.0/24 |

### Internet Gateway & NAT Gateway
- **IGW:** `HIS-PRD-IGW` → VPC 연결
- **NAT:** `HIS-PRD-NGW-2A` / 서브넷 NGINX-PUB-2A / 퍼블릭 / 탄력적 IP 할당

### Routing Table (Public & Private)
- `HIS-PRD-RT-PUB` — `0.0.0.0/0` → IGW / 연결: NGINX-PUB-2A, 2C
- `HIS-PRD-RT-PRI` — `0.0.0.0/0` → NAT / 연결: TOMCAT-PRI-2A, 2C

### Route 53
- 가비아 도메인 `history-cloud.store`의 네임서버를 AWS 호스팅 영역 네임서버로 변경
- 퍼블릭 호스팅 영역 생성

---

## 2) EC2 및 Bastion Host 운영

### Security Group
| SG | 인바운드 규칙 |
|----|--------------|
| HIS-PRD-VPC-NGINX-PUB-SG | HTTP, HTTPS, SSH |
| HIS-PRD-VPC-TOMCAT-PRI-SG | 8080, SSH |

### EC2 Instance 공통 설정
- AMI: Ubuntu 24.04 / 인스턴스 유형: t3.micro / 키 페어: `his-keypair.pem` (RSA, .pem)

### 인스턴스별 IP
| 인스턴스 | 2A | 2C | 퍼블릭 IP |
|----------|-----|-----|----------|
| NGINX | 10.250.1.240 | 10.250.11.240 | 활성화 |
| TOMCAT | 10.250.2.240 | 10.250.12.240 | 비활성화 |
| Bastion | 10.250.4.240 | — | 활성화 |

### Bastion Host 접속
- **Xshell 연결:** 프로토콜 SSH / 호스트 Bastion Public IPv4 / 사용자 `ubuntu` / Public Key `his-keypair`
- **Bastion → 타 EC2 SSH 접속 절차**
  1. SFTP로 `.pem` 파일 이동 (또는 파일 내용 복사)
  2. `.pem` 파일 권한 변경 — `chmod 400 his-keypair.pem`
  3. SSH 접속 — `ssh -i his-keypair.pem ubuntu@<상대 EC2 Private IPv4>`

---

## 3) Load Balancer(ELB) 구축

### Security Group
- `HIS-NGINX-ALB-SG` / 인바운드: HTTP, HTTPS

### Target Group
- 이름 `HIS-PRD-ALB-TG` / 대상 유형: 인스턴스
- 대상: NGINX-PUB-2A, NGINX-PUB-2C

### ALB 생성
- 이름 `HIS-NGINX-ALB` / 유형 ALB / 체계: 인터넷 경계
- 가용 영역·서브넷: NGINX-PUB-2A, 2C
- 리스너: HTTP:80 → 대상 그룹 `HIS-PRD-ALB-TG`로 전달

### Route 53 — ALB 연결
- 레코드 A (`@`, `www`) / 별칭: ALB(서울) / 라우팅 정책: 단순 라우팅

---

## 4) 티어별 서비스 연동

### SSH 설정 (Bastion Host)
- `~/.ssh/config`에 각 서버(nginx2a/2c, tomcat2a/2c) 호스트 alias 등록 → `ssh nginx2a` 형태로 간편 접속

### Nginx 설치 (NGINX-2A, 2C)
1. 저장소 파일 생성 및 내용 작성
2. GPG 키 추가 — `curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo gpg --dearmor ...`
3. `sudo apt update` → `sudo apt install nginx`
4. 버전 확인(`nginx -V`, nginx/1.28.3) 및 `systemctl start nginx`

### index.html 수정 및 확인
- NGINX-2A(빨강), NGINX-2C(파랑)로 구분되도록 `index.html` 수정
- Public IPv4 접속(2A/2C) 및 `history-cloud.store` 접속(ALB)으로 부하 분산 동작 확인

### Tomcat 설치 (TOMCAT-2A, 2C)
1. OpenJDK 11 설치
2. 환경 변수 추가 (`/etc/profile` — JAVA_HOME 등)
3. Tomcat 9.0.108 설치
   - 3-1. systemd 서비스 등록 (`/etc/systemd/system/tomcat.service`)
   - 3-2. 서비스 등록·활성화, 권한 설정, 서비스 시작 및 확인

### Nginx ↔ Tomcat 통신
- Nginx 리버스 프록시 설정: `/etc/nginx/conf.d/default.conf`에서 `proxy_pass http://<tomcat private IP>:8080`
- `nginx -t`로 문법 확인 후 `systemctl restart nginx`
- `.jsp` 파일을 Bastion에서 받아 `scp`로 nginx2a/2c에 이동 → `index`로 대체 → `history-cloud.store`에서 3Tier 데모 페이지 확인

---

## 5) 관계형 데이터베이스(RDS) 구축

### Subnet 생성 및 Routing Table 설정
- `HIS-PRD-VPC-DB-PRI-2A` (10.250.3.0/24) / `-2C` (10.250.13.0/24)
- `HIS-PRD-RT-PRI`에 DB 서브넷 명시적 연결 추가

### Security Group & 서브넷 그룹
- SG `HIS-PRD-VPC-DB-PRI-SG` / 인바운드: MySQL/Aurora(3306)
- 서브넷 그룹 `HIS-PRD-VPC-DB-GROUP` (DB-PRI-2A, 2C)

### RDS 생성 (HistoryDB)
- 엔진 MySQL 8.0.40 / 템플릿 프리티어 → 단일 AZ
- 인스턴스 클래스 버스터블(`db.t4g.micro`) / 스토리지 마그네틱 5GB
- 자격 증명: 마스터 `admin` / 자체 관리 / 암호화 활성화 / 자동 백업 비활성화
- 초기 데이터베이스: `HIS_PRD_DB`

### DB 접속 (MySQL Workbench)
- SSH 터널: Bastion Public IPv4 / 사용자 `ubuntu` / 키 `his-keypair.pem`
- MySQL Host: RDS 엔드포인트 / Username: 마스터 이름
- 접속 후 테이블 생성 및 샘플 데이터 삽입

### Tomcat ↔ DB 연동
1. MySQL Connector/J 드라이버 설치 (`.../tomcat/lib`)
2. `server.xml` — 글로벌 네이밍 리소스(`jdbc/HistoryDB`) 설정
3. `context.xml` — `ResourceLink` 추가

### 연동 결과
- 게시글 목록: 쿼리로 삽입한 데이터 정상 조회
- 게시글 작성: 작성 후 목록에서 반영 확인 → **DB 연동 성공**
