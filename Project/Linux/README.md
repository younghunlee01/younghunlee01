# Linux Project

> Rocky Linux 9.7 기반 인프라 구축 프로젝트
> VMware 가상화 환경에서 **3대의 서버**를 구성하고, 사용자·스토리지 관리부터 **9종의 네트워크 서비스**까지 직접 구축·검증한 팀 프로젝트입니다.

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **OS** | Rocky Linux 9.7 |
| **가상화** | VMware Workstation Pro 17 |
| **구성 대수** | 3대 (Project / Project-B / Project-C) |
| **주요 영역** | 네트워크 · 사용자/그룹 · LVM · 디스크 쿼터 · 서버 서비스 9종 |

### 서버 구성

| VM 이름 | IP 주소 | RAM | DISK | 역할 |
|---------|---------|-----|------|------|
| **Project** | 192.168.111.100/24 | 4 GB | 80 GB | 메인 서버 (DNS, Mail, DB, NFS, Samba, DHCP 등) |
| **Project-B** | 192.168.111.150/24 | 2 GB | 80 GB | Web 서버 / 클라이언트 |
| **Project-C** | 192.168.111.200/24 | 2 GB | 80 GB | FTP 서버 / 클라이언트 |

> Gateway · DNS: `192.168.111.2`

---

## 목차

### 1. 기본 인프라
- [01. 설치 & 네트워크 설정](docs/01-installation.md)
- [02. 사용자 및 그룹 등록](docs/02-users-and-groups.md)
- [03. 디스크 추가 후 LVM 구성](docs/03-lvm.md)
- [04. 디스크 쿼터 설정](docs/04-disk-quota.md)

### 2. 서버 구성
- [05. 서버 구성 (9종 서비스)](docs/05-servers.md)
  — SSH · XRDP ·  DNS ·  Web ·  FTP · NFS · Samba · DHCP · Mail · MariaDB

---

## 담당 파트 (My Contribution)


- **02. 사용자 및 그룹 등록** — 계정/그룹 생성, 기본 그룹 변경, 그룹 멤버 관리, `/etc/passwd`·`/etc/shadow`·`/etc/group` 구조 분석
- **05. 서버 구성 (일부)**
  - **SSH Server** — 원격 접속 환경 구성 및 XShell 연동
  - **DNS Server** — 존 파일·CNAME 레코드 설계 및 정·역방향 조회 검증
  - **Web Server** — Apache(httpd) 구성 및 가상 도메인 서비스
  - **FTP Server** — vsftpd 익명 접속 환경 구성 및 파일 전송 검증

---

## 🛠️ 기술 스택

`Rocky Linux 9.7` · `VMware` · `NetworkManager(nmcli/nmtui)` · `LVM` · `Quota`
`OpenSSH` · `XRDP` · `BIND(named)` · `Apache httpd` · `vsftpd` · `NFS` · `Samba` · `dhcp-server` · `Sendmail/Dovecot` · `MariaDB`
`firewalld` · `SELinux`

---

## 🔑 배운 점

- 물리 디스크 추가부터 **PV → VG → LV** 로 이어지는 LVM 스토리지 계층을 직접 구성하며 유연한 볼륨 관리 방식을 이해했습니다.
- 각 서버 서비스마다 **설치 → 환경설정 → 서비스 기동 → 방화벽 허가 → 동작 검증**이라는 일관된 구축 흐름을 체득했습니다.
- DNS의 정방향/CNAME 레코드와 Mail의 MX 레코드가 실제 서비스(Web·FTP·Mail)와 어떻게 연동되는지 end-to-end로 확인했습니다.
