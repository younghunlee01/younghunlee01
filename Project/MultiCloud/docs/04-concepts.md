# 04. 핵심 개념

이 프로젝트를 이해하는 데 필요한 핵심 개념을 정리했습니다.

---

## 멀티클라우드

둘 이상의 퍼블릭 클라우드(예: AWS + GCP)를 하나의 시스템에서 함께 사용하는 구성입니다.

### 장점
- 벤더 종속(lock-in) 회피 — 이전성과 협상력 확보
- 각 클라우드의 강점만 골라 조합 (AWS 생태계 + GCP 데이터)
- 리전·가용성 분산으로 장애 위험 분산
- 규제·데이터 주권 요건에 유연하게 대응

### 단점
- 네트워크·보안 구성 복잡도 증가 (VPN / BGP)
- 운영·모니터링 도구가 플랫폼별로 분산 → 관리 부담
- 클라우드 간 데이터 전송(egress) 비용·지연
- 두 플랫폼 모두 숙련 필요 — 학습 곡선

> **본 프로젝트** — AWS(Instance·네트워크)와 GCP(Cloud SQL)를 조합하고, 복잡도는 VPN으로 연결해 해결했습니다.

---

## 클라우드 간 VPN 연결

서로 다른 클라우드의 VPC를 VPN으로 묶어 하나의 내부망처럼 통신하게 하는 구성입니다.

- 공인 IP 노출 없이 비공개 IP로 직접 통신
- IPsec으로 구간 암호화 — 인터넷에 데이터 노출 안 됨
- 실무 하이브리드/멀티클라우드의 기본 연결 패턴

```
AWS VPC (10.0.0.0/16)  ──IPsec VPN 사설 터널──  GCP VPC (historynet)
     EC2 Bastion              공인 인터넷 미경유·암호화        Cloud SQL · 비공개 IP
```

---

## HA VPN과 BGP

### 왜 터널이 4개인가
- **HA VPN** — 게이트웨이 인터페이스가 2개라 한쪽 장애에도 끊기지 않음
- **AWS Site-to-Site VPN** — 연결당 터널 2개
- GCP 인터페이스 2개 × AWS 터널 2개 = **4개 풀메시 구성**

### 왜 BGP인가
- 정적(수동) 입력 대신 BGP로 양측 경로를 **동적 교환**
- 터널 장애 시 살아있는 경로로 자동 우회
- ASN 65000(GCP) ↔ 64512(AWS)로 경로 협상

> **요약** — 이중화(HA)와 동적 라우팅(BGP)을 위해 터널을 4개로 풀메시 구성했습니다.

---

## 구성요소 매핑 (AWS ↔ GCP)

이름만 다를 뿐 역할은 동일합니다.

| 역할 | AWS | GCP |
|------|-----|-----|
| 자기측 VPN 게이트웨이 | Virtual Private GW | HA VPN Gateway |
| 상대 클라우드 표현 | Customer Gateway | Peer VPN Gateway |
| 암호화 터널 | Site-to-Site VPN | VPN Tunnel |
| 동적 라우팅 주체 | VGW + 라우팅 전파 | Cloud Router + BGP |
| BGP ASN | 64512 | 65000 |

---

## 라우팅과 VPC 피어링

### 문제 — 피어링은 전이되지 않는다

```
AWS VPC  ──VPN·BGP──>  GCP historynet  ──VPC 피어링──>  servicenetworking (Cloud SQL)
```

VPC 피어링은 **전이(transitive)되지 않기 때문에**, 기본 상태에서는 AWS가 Cloud SQL 대역을 학습하지 못합니다.
(A-B, B-C가 연결돼도 A가 C를 자동으로 알지 못함)

### 해결
- **AWS** — 라우팅 테이블에 VPN 경로 전파(propagation) 활성화
- **GCP** — 피어링에서 커스텀 경로 가져오기/내보내기 활성화
- **GCP** — Cloud Router 커스텀 경로에 `10.255.16.0/24` 추가 → BGP로 AWS에 전달

> **요약** — VPC 피어링은 전이되지 않으므로, Cloud SQL 대역을 Cloud Router 커스텀 경로에 추가해야 합니다.

---

## 사설 DNS

비공개 IP 대신 도메인으로 접속하게 만들어, 클라우드 간 서비스 디스커버리를 구현합니다.

```
애플리케이션 (mysql -h db.cloud.local)
  → Route 53 · cloud.local (CNAME → A 레코드, 값 = Cloud SQL 비공개 IP)
  → Cloud SQL (10.255.16.3)
```

### 왜 이름으로?
- IP가 바뀌어도 애플리케이션 코드/설정은 그대로 — 이름만 유지
- 사설 호스팅 영역이라 내부에서만 해석 (서비스 디스커버리)
- 검증: Bastion에서 `mysql -h db.cloud.local` 접속 성공

> **요약** — 비공개 IP 대신 도메인으로 접속하게 만들어, 클라우드 간 서비스 디스커버리를 구현했습니다.
