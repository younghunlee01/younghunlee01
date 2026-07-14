# 02. 사용자 및 그룹 등록
[← 목차로 돌아가기](../README.md)

## 목표
- 사용자 8명 생성
- `eusoccer` / `krsoccer` 두 그룹 생성
- 각 그룹에 사용자를 포함

### 그룹 구성

| 그룹 | 멤버 |
|------|------|
| **eusoccer** | sonhm(손흥민), leegi(이강인), kimmj(김민재), hwanghc(황희찬) |
| **krsoccer** | kdj, lyh, jsh, jgh (조원 1~4) |

---

## 1. 사용자 생성

### 현재 등록된 사용자 확인
```bash
tail -5 /etc/passwd     # 파일의 마지막 5행 출력
```

> **`/etc/passwd` 구조** — 사용자 계정 정보 저장 파일
> `계정명 : 암호 : UID : GID : 설명 : 홈디렉터리 : 로그인 쉘`

### 사용자 등록
```bash
adduser sonhm
adduser leegi
adduser kimmj
adduser hwanghc
adduser kdj
adduser lyh
adduser jsh
adduser jgh
```

### 추가된 사용자 확인
```bash
tail /etc/passwd        # 기본값 10행 출력
```
```
sonhm:x:1001:1001::/home/sonhm:/bin/bash
leegi:x:1002:1002::/home/leegi:/bin/bash
...
```
> 💡 일반 사용자의 **UID는 1000번부터** 지정된다.

---

## 2. 사용자 패스워드 설정

```bash
passwd sonhm
```
패스워드는 `/etc/shadow`에 저장되며, 암호를 지정한 사용자만 암호화된 해시가 표시된다.

```bash
tail /etc/shadow
```
> **`/etc/shadow` 구조**
> `계정명 : 암호 : 최종변경일 : MIN : MAX : WARNING : INACTIVE : EXPIRE : Flag`

---

## 3. 그룹 생성

### 그룹 목록 확인
```bash
tail /etc/group
```

### 그룹 생성
```bash
groupadd eusoccer
groupadd krsoccer
```

### 추가된 그룹 확인
```bash
tail /etc/group
```
```
eusoccer:x:1009:
krsoccer:x:1010:
```
> **`/etc/group` 구조**
> `그룹명 : 그룹암호 : GID : 그룹 멤버`

---

## 4. 사용자를 그룹에 포함

### 기본 그룹(Primary Group) 변경 — `usermod -g`
```bash
usermod -g eusoccer sonhm
usermod -g eusoccer leegi
usermod -g eusoccer kimmj
usermod -g eusoccer hwanghc
usermod -g krsoccer kdj
usermod -g krsoccer lyh
usermod -g krsoccer jsh
usermod -g krsoccer jgh
```
`/etc/passwd`의 GID가 변경된 그룹 번호(1009/1010)로 바뀐다.

### 그룹 멤버(Secondary)로 등록 — `gpasswd -a`
```bash
gpasswd -a sonhm eusoccer
gpasswd -a leegi eusoccer
gpasswd -a kimmj eusoccer
gpasswd -a hwanghc eusoccer
gpasswd -a kdj krsoccer
gpasswd -a lyh krsoccer
gpasswd -a jsh krsoccer
gpasswd -a jgh krsoccer
```

### 그룹 멤버 등록 확인
```bash
tail /etc/group
```
```
eusoccer:x:1009:sonhm,leegi,kimmj,hwanghc
krsoccer:x:1010:kdj,lyh,jsh,jgh
```

---

## 정리

- `adduser` → `passwd` → `groupadd` → `usermod -g` / `gpasswd -a` 순으로 계정과 그룹을 구성
- 계정/그룹 정보가 각각 `/etc/passwd`, `/etc/shadow`, `/etc/group` 파일에 저장되는 구조를 직접 확인
- `usermod -g`(기본 그룹 변경)와 `gpasswd -a`(보조 그룹 추가)의 차이를 이해

➡️ 다음: [03. 디스크 추가 후 LVM 구성](03-lvm.md)
