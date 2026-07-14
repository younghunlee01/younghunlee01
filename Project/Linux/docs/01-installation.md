# 01. 설치 & 네트워크 설정
[← 목차로 돌아가기](../README.md)

## 목표
- VMware Workstation Pro 17에 Rocky Linux 9.7을 3대 설치
- 각 VM에 고정 IP 주소 부여

## VM 사양

| VM 이름 | RAM | DISK | IP 주소 |
|---------|-----|------|---------|
| Project | 4 GB | 80 GB | 192.168.111.100/24 |
| Project(B) | 2 GB | 80 GB | 192.168.111.150/24 |
| Project(C) | 2 GB | 80 GB | 192.168.111.200/24 |

- Gateway: `192.168.111.2`
- DNS: `192.168.111.2`

세 VM 모두 하나의 Virtual Network(VMnet)에 연결되어 있으며, 이 네트워크가 Host PC / 인터넷과 통신한다.

---

## 방법 1. CLI 환경에서 네트워크 변경

### 1) 변경 전 IP 확인
```bash
ip address
```

### 2) 네트워크 환경 설정 파일 수정
```bash
vi /etc/NetworkManager/system-connections/ens160.nmconnection
```

`[ipv4]` 섹션을 아래와 같이 수정한다.
```ini
[ipv4]
address1=192.168.111.100/24
dns=192.168.111.2;
gateway=192.168.111.2
method=manual
```

### 3) 네트워크 연결 적용 및 재연결
```bash
nmcli connection reload
nmcli connection up ens160
```

### 4) 변경된 IP 확인
```bash
ip a
```

---

## 방법 2. GUI / TUI 환경에서 네트워크 변경

### TUI (nmtui)
```bash
nmtui
```
`연결 편집` → IPv4 설정을 `<수동>`으로 변경 후 주소/게이트웨이/DNS 입력 → 연결 비활성화 후 재활성화.

### GUI (설정)
`유선 네트워크 설정` → **IPv4** 탭 → `수동` 선택
- 주소: `192.168.111.100`
- 네트마스크: `255.255.255.0`
- 게이트웨이: `192.168.111.2`
- DNS: `192.168.111.2`

적용 후 연결을 껐다 켜면 변경된 IP가 적용된다.

---

## 결과

| VM | 적용된 IP |
|----|-----------|
| Project | 192.168.111.100/24 |
| Project-B | 192.168.111.150/24 |
| Project-C | 192.168.111.200/24 |

➡️ 다음: [02. 사용자 및 그룹 등록](02-users-and-groups.md)
