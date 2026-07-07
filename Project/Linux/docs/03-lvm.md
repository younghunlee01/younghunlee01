# 03. 디스크 추가 후 LVM 구성
[← 목차로 돌아가기](../README.md)

## 목표
- 물리 디스크 3개(20GB · 30GB · 50GB) 추가
- LVM으로 묶어 논리 볼륨 구성

### 구성 계획

```
Physical Volume (PV)   →   Volume Group (VG)   →   Logical Volume (LV)
20G / 30G / 50G            DATA (~100G)             VIDEO (40G)
                                                    AUDIO (나머지)
```

---

## 1. 디스크 장착 확인

```bash
lsblk                    # sdb(20G), sdc(30G), sdd(50G) 확인
fdisk -l | grep sd
```

---

## 2. 파티션 할당

각 디스크를 `Linux LVM(8e)` 타입 파티션으로 생성한다.
```bash
fdisk /dev/sdb    # → sdb1 (20G, Linux LVM)
fdisk /dev/sdc    # → sdc1 (30G, Linux LVM)
fdisk /dev/sdd    # → sdd1 (50G, Linux LVM)
```

---

## 3. PV 구성

```bash
pvcreate /dev/sdb1
pvcreate /dev/sdc1
pvcreate /dev/sdd1
pvscan
```

---

## 4. VG 구성

```bash
vgcreate DATA /dev/sdb1 /dev/sdc1 /dev/sdd1
vgdisplay
```
`VG Name: DATA`, `VG Size: <99.99 GiB`, `Cur PV: 3` 로 세 물리 볼륨이 하나의 그룹으로 묶인 것을 확인.

---

## 5. LV 구성

```bash
lvcreate --size 40G --name VIDEO DATA          # VIDEO: 40G
lvcreate --extents 100%FREE --name AUDIO DATA  # AUDIO: 나머지 전부
lvscan
ls -l /dev/DATA
```
```
/dev/DATA/VIDEO -> ../dm-3
/dev/DATA/AUDIO -> ../dm-4
```

---

## 6. 파일 시스템 생성 & 마운트

```bash
mkfs.ext4 /dev/DATA/VIDEO
mkfs.ext4 /dev/DATA/AUDIO

mkdir /lvm1 /lvm2
mount /dev/DATA/VIDEO /lvm1
mount /dev/DATA/AUDIO /lvm2
df
```

---

## 7. 부팅 시 자동 마운트 (`/etc/fstab`)

```bash
vi /etc/fstab
```
```
/dev/DATA/VIDEO   /lvm1   ext4   defaults   0 0
/dev/DATA/AUDIO   /lvm2   ext4   defaults   0 0
```

---

## 정리

- 물리 디스크 → **PV → VG(DATA) → LV(VIDEO/AUDIO)** 계층으로 스토리지를 구성했다.
- `--extents 100%FREE` 옵션으로 VG의 남은 공간 전체를 AUDIO 볼륨에 할당했다.
- `/etc/fstab`에 등록해 재부팅 후에도 마운트가 유지되도록 했다.
