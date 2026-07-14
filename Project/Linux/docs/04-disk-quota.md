# 04. 디스크 쿼터 설정
[← 목차로 돌아가기](../README.md)

## 목표
- 10GB 하드 디스크 추가
- 사용자별 디스크 사용량 제한(Quota) 설정

### 쿼터 대상 사용자

| 사용자 | Soft Limit | Hard Limit |
|--------|-----------|-----------|
| aespa | 700M | 1G |
| IVE | 700M | 1G |
| NewJeans | 700M | 1G |

---

## 1. 디스크 장착 & 마운트

```bash
lsblk                        # sdb(10G) 확인
fdisk /dev/sdb               # → sdb1 (10G, Linux)
mkfs.ext4 /dev/sdb1
mkdir /userHome
mount /dev/sdb1 /userHome
```

### `/etc/fstab` 등록 (쿼터 옵션 포함)
```bash
vi /etc/fstab
```
```
/dev/sdb1  /userHome  ext4  defaults,usrjquota=aquota.user,jqfmt=vfsv0  0 0
```

---

## 2. 사용자 생성 (홈 디렉터리를 /userHome 하위로)

```bash
useradd -d /userHome/aespa aespa
useradd -d /userHome/IVE IVE
useradd -d /userHome/NewJeans NewJeans

passwd aespa
passwd IVE
passwd NewJeans
```

### 재마운트로 fstab 옵션 적용
```bash
mount --options remount /userHome
```

---

## 3. 쿼터 DB 생성

```bash
cd /userHome
quotaoff -avug
quotacheck -augmn
rm -rf aquota.*
quotacheck -augmn
touch aquota.user aquota.group
chmod 600 aquota.*
quotacheck -augmn
quotaon -avug
```

---

## 4. 사용자별 사용 공간 할당

```bash
edquota -u aespa
edquota -u IVE
edquota -u NewJeans
```
각 사용자에 `soft 700 / hard 1000` (블록 단위)을 입력.

> 💡 한 사용자 설정을 다른 사용자에게 복사: `edquota -p aespa IVE`

---

## 5. 동작 검증

### 초과 테스트 (aespa 계정)
```bash
su - aespa
cp /boot/vmlinuz-5.* test1
# → sdb1: warning, user block quota exceeded.
cp test1 test2
# → cp: error copying 'test1' to 'test2': 디스크 할당량이 초과됨
```
soft limit를 넘으면 경고, hard limit에 도달하면 쓰기가 차단된다.

### 쿼터 확인
```bash
quota                        # 개별 사용자 사용량
repquota /userHome           # 전체 사용자 사용량 리포트
```

---

## 정리

- `usrjquota` 옵션으로 마운트한 파일 시스템에서 사용자별 쿼터를 활성화
- `quotacheck` → `edquota` → `quotaon` 순서로 쿼터를 구성
- soft/hard limit의 실제 동작(경고 vs 차단)을 파일 복사 테스트로 검증

➡️ 다음: [05. 서버 구성](05-servers.md)
