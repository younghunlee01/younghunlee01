# 03. 핵심 기술 및 아키텍처 이론


---

## AWS 글로벌 인프라

- **Regions** — AWS가 전 세계에서 데이터 센터를 클러스터링하는 물리적 위치
- **가용 영역(Availability Zone)** — AWS 리전의 중복 전력, 네트워킹 및 연결이 제공되는 하나 이상의 개별 데이터 센터

---

## AWS 주요 컴포넌트

| 컴포넌트 | 설명 |
|---------|------|
| **VPC** (Virtual Private Cloud) | 클라우드 환경에서 사용자가 생성하고 관리하는 독립적인 가상 네트워크 공간 |
| **Subnet** | 네트워크 영역을 분할하여 더 작은 크기로 쪼갠 네트워크 |
| **Routing Table** | 목적지 주소를 목적지에 도달하기 위한 네트워크 노선으로 변환시키는 정의서 |
| **Internet Gateway** | VPC와 인터넷 사이의 통신을 가능하게 하는 수평 확장·중복·고가용성 VPC 컴포넌트 |
| **NAT Gateway** | 사설 IP를 사용하면서 공인 IP와 상호 변환해 주는 VPC 컴포넌트 (IPv4 주소 부족 문제 해결) |
| **EC2** (Elastic Compute Cloud) | AWS 클라우드 환경의 가상 서버 |
| **Security Group** | 연결된 리소스에 도달하고 나가는 트래픽을 제어하는 가상 방화벽 |
| **ELB** (Elastic Load Balancer) | 트래픽을 여러 대상에 분산시켜 가용성과 확장성을 높이는 서비스 |
| **Target Group** | ELB에서 요청을 처리하는 실제 백엔드 서버를 논리적으로 묶어 관리하는 그룹 |
| **Route 53** | 안정적이고 확장 가능한 DNS, 도메인 등록, 상태 확인 기능을 제공하는 웹 서비스 |

---

## 3-Tier Architecture

애플리케이션을 3개의 논리적·물리적 컴퓨팅 계층으로 분리해서 운영하는 소프트웨어 아키텍처.

각 계층이 자체 인프라에서 실행되므로 별도의 개발 팀에 의해 동시에 개발·운영될 수 있어, 다른 계층에 영향을 주지 않고 필요에 따라 업데이트되거나 확장될 수 있다.

| 계층 | 역할 | 담당 서버 |
|------|------|----------|
| **Presentation Tier** | 사용자 인터페이스(UI) 담당. 웹 서버가 사용자의 요청을 가장 먼저 받아 처리 | Nginx |
| **Application Tier** | 비즈니스 로직 처리. 웹 서버의 요청을 받아 데이터를 처리하고 DB와 연동 | Apache Tomcat |
| **Data Tier** | 정보 저장 및 관리. DB 서버가 담당하며 데이터를 안전하게 저장하고 조회 | MySQL (RDS) |

**요청 흐름:**
`Client → HTTP Request → Presentation(Nginx) → Application(Tomcat) → Data Query → Data(MySQL)`
