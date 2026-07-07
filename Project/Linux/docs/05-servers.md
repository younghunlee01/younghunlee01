# 05. 서버 구성 (Server Configuration)
[← 목차로 돌아가기](../README.md)

Rocky Linux 9.7 환경에서 구축한 9종의 네트워크 서비스입니다.
모든 서버는 **설치 → 환경설정 → 서비스 기동 → 방화벽 허가 → 동작 검증**의 일관된 흐름으로 구성했습니다.

## 목차
- 🙋 [SSH Server](#ssh-server)
- [XRDP Server](#xrdp-server)
- 🙋 [DNS Server](#dns-server)
- 🙋 [Web Server](#web-server)
- 🙋 [FTP Server](#ftp-server)
- [NFS Server](#nfs-server)
- [Samba Server](#samba-server)
- [DHCP Server](#dhcp-server)
- [Mail Server](#mail-server)
- [MariaDB Server](#mariadb-server)

> 🙋 표시가 제가 직접 구축·문서화한 담당 파트입니다.

---

# SSH Server 🙋

원격 접속 환경을 구성하고 Windows XShell에서 Project(192.168.111.100)에 접속.

```bash
# 1. 설치 여부 확인 및 설치
rpm -qa openssh-server
dnf install openssh-server          # 미설치 시

# 2. 환경설정
vi /etc/ssh/sshd_config             # 필요 시 설정 변경

# 3. 서비스 시작
systemctl restart sshd
systemctl enable sshd

# 4. 방화벽 허가
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload
firewall-cmd --list-services        # ssh 확인
```

**5. XShell 연결** — 호스트 `192.168.111.100`, 포트 `22`, 프로토콜 SSH로 접속 성공.

> 📌 이후 대부분의 서버 작업을 XShell 원격 세션에서 진행.

---

# XRDP Server

Windows 원격 데스크톱(RDP)으로 GUI 접속.

```bash
# 1. 설치
rpm -qa xrdp
dnf -y install epel-release
dnf -y install xrdp

# 2. 서비스 시작 및 활성화
systemctl start xrdp
systemctl enable xrdp
systemctl status xrdp               # active (running) 확인

# 3. 방화벽 허가 (RDP 포트 3389)
firewall-cmd --permanent --add-port=3389/tcp
firewall-cmd --reload
firewall-cmd --list-ports           # 3389/tcp 확인
```

**연결** — Windows `원격 데스크톱 연결`에서 `192.168.111.100` 입력 → 로그인 화면에서 `Session: Xorg`, `username: rocky` + 패스워드 → 데스크톱 접속 성공.

---

# DNS Server 🙋

BIND(named)로 `history.com` 도메인을 구축하고, CNAME으로 `www`→Project-B, `ftp`→Project-C 매핑.

```bash
# 1. 설치
rpm -qa bind bind-chroot
dnf -y install bind bind-chroot
```

**2. 환경설정 — `/etc/named.conf`**
```conf
options {
    listen-on port 53 { any; };
    listen-on-v6 port 53 { none; };
    allow-query { any; };
    recursion yes;
    dnssec-validation no;
};

zone "history.com" IN {
    type master;
    file "history.com.db";
    allow-update { none; };
};
```

```bash
# 3. 서비스 시작
systemctl start named
systemctl enable named

# 4. 방화벽 허가
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload
firewall-cmd --list-services        # dns 확인
```

**5. 클라이언트 네임서버 지정 — `/etc/resolv.conf`**
```
nameserver 192.168.111.100
```

**6. 존 파일 설정 — `/var/named/history.com.db`**
```dns
$TTL    3H
@       SOA     @   root. ( 2 1D 1H 1W 1H )
        IN      NS      @
        IN      A       192.168.111.100

project     IN  A       192.168.111.100
project-b   IN  A       192.168.111.150
project-c   IN  A       192.168.111.200

www         IN  CNAME   project-b
ftp         IN  CNAME   project-c
```

```bash
# 7. 오류 검사
named-checkconf
named-checkzone history.com history.com.db   # loaded serial 2, OK
```

**8. 서버 접속 확인**
```bash
nslookup
> www.history.com    # → project-b.history.com. / 192.168.111.150
> ftp.history.com    # → project-c.history.com. / 192.168.111.200
```

> **www**은 Project-B, **ftp**는 Project-C의 CNAME으로 설정 → 이후 Web·FTP 서버를 도메인 이름으로 접근하는 기반 마련.

---

# Web Server 🙋

Apache(httpd) 구성 — **Project-B**, `www.history.com`으로 서비스.

```bash
# 1. 설치
rpm -qa httpd
dnf install -y httpd

# 2. 서비스 시작
systemctl start httpd
systemctl enable httpd

# 3. 방화벽 허가
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-services        # http 확인
```

**4. index.html 수정 — `/var/www/html/index.html`**
```html
<html>
    <body>
        <h1>Team History</h1>
    </body>
</html>
```

**5. 접속 확인**
```bash
curl www.history.com                # <h1>Team History</h1> 출력
```
브라우저 `http://www.history.com` 접속 시 **Team History** 페이지 표시. (DNS CNAME → Project-B로 연결)

---

# FTP Server 🙋

vsftpd 익명 접속 구성 — **Project-C**, `ftp.history.com`으로 서비스.

```bash
# 1. 설치
rpm -qa vsftpd
dnf -y install vsftpd

# 2. 환경 설정 — 익명 접속 허용
vi /etc/vsftpd/vsftpd.conf
# anonymous_enable=YES

# 3. 공유 디렉터리에 파일 생성
cd /var/ftp/pub
cp /boot/vmlinuz-5.* file1

# 4. 서비스 시작
systemctl restart vsftpd
systemctl enable vsftpd

# 5. 방화벽 허가
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload
firewall-cmd --list-services        # ftp 확인
```

**6. 클라이언트 접속 & 다운로드**
```bash
ftp ftp.history.com
# Name: anonymous / Password: (아무 값) → 230 Login successful.
ftp> cd pub
ftp> ls
ftp> get file1                      # 다운로드
```
> 익명 접속을 허용했으므로 Name에 `anonymous`, 패스워드는 아무 값이나 입력. 파일은 접속 당시 디렉터리에 다운로드됨.

---

# NFS Server

`/share` 디렉터리를 공유하고 클라이언트(Project-B)에서 마운트.

**서버 측 (Project)**
```bash
rpm -qa nfs-utils

vi /etc/exports
# /share  *(rw,sync)

mkdir /share
chmod 707 /share

systemctl restart nfs-server
systemctl enable nfs-server

firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
firewall-cmd --list-services        # nfs 확인
```

**클라이언트 측 (Project-B)**
```bash
mkdir myShare
mount -t nfs 192.168.111.100:/share myShare

# /etc/fstab 등록
# 192.168.111.100:/share  /home/rocky/myShare  nfs  defaults  0 0
```
재부팅 후 `myShare`에서 파일 생성/공유 확인.

---

# Samba Server

Windows와 파일 공유(`/sbhistory`), 전용 그룹 기반 접근 제어.

```bash
# 1. 설치
dnf -y install samba samba-common samba-client

# 2. 공유 디렉터리 & 권한 부여
mkdir /sbhistory
groupadd sambaGroup
chgrp sambaGroup /sbhistory
chmod 770 /sbhistory
usermod -G sambaGroup rocky
smbpasswd -a rocky
```

**3. 환경설정 — `/etc/samba/smb.conf`**
```ini
[global]
    workgroup = WORKGROUP
    security = user
    unix charset = UTF-8
    map to guest = Bad User

[sbhistory]
    path = /sbhistory
    writable = yes
    guest ok = no
    create mode = 0777
    directory mode = 0777
    valid users = @sambaGroup
```

```bash
# 4. 설정파일 오류 확인
testparm                            # Loaded services file OK.

# 5. 서비스 시작
systemctl restart smb nmb
systemctl enable smb nmb

# 6. 방화벽 허가
firewall-cmd --permanent --add-service=samba
firewall-cmd --permanent --add-service=samba-client
firewall-cmd --reload

# 7. SELinux 설정
setsebool -P samba_enable_home_dirs on
chcon -R -t samba_share_t /sbhistory

# 8. 테스트 파일 생성
touch /sbhistory/f1.txt
```

**9. Windows 접속** — `실행(Win+R)` → `\\192.168.111.100\sbhistory` → 자격 증명 `rocky` 입력 → `f1.txt` 공유 파일 확인.

> Rocky Linux에서는 방화벽뿐 아니라 **SELinux 컨텍스트(samba_share_t)** 설정이 필요.

---

# DHCP Server

Project 서버를 DHCP 서버로 구성, Project-C에 IP 자동 배정.

**1. 사전 작업** — VMware DHCP 체크 해제, Project-C 네트워크를 `자동(DHCP)`으로 지정.

```bash
# 2. 설치
dnf -y install dhcp-server
```

**3. 환경설정 — `/etc/dhcp/dhcpd.conf`**
```conf
ddns-update-style interim;
subnet 192.168.111.0 netmask 255.255.255.0 {
    option routers 192.168.111.2;
    option subnet-mask 255.255.255.0;
    range dynamic-bootp 192.168.111.201 192.168.111.210;
    option domain-name-servers 8.8.8.8;
    default-lease-time 10000;
    max-lease-time 50000;
}
```

```bash
# 4. 서비스 시작
systemctl restart dhcpd
systemctl enable dhcpd

# 5. 방화벽 허가
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --reload
firewall-cmd --list-services        # dhcp 확인
```

**6. 확인 (Project-C)** — `ip a` 실행 시 지정 범위의 첫 주소 `192.168.111.201`이 자동 배정됨.

> DHCP는 브로드캐스트 기반이므로 같은 네트워크의 호스트라면 IP를 배정받을 수 있음.

---

# Mail Server

Sendmail(발신) + Dovecot(수신)로 `mail.history.com` 메일 서버 구축.

```bash
# 1. 설치
rpm -qa sendmail
dnf -y install sendmail sendmail-cf dovecot

# 2. 호스트 이름 변경
hostnamectl set-hostname mail.history.com
# /etc/hosts, /etc/mail/local-host-names, /etc/sysconfig/network 에 mail.history.com 등록
```

**3~5. DNS 존 파일에 MX 레코드 추가 — `/var/named/history.com.db`**
```dns
@     IN  MX  10  mail.history.com.
mail  IN  A       192.168.111.100
```
```bash
named-checkconf
named-checkzone history.com history.com.db    # OK
```

**6~8. 발신 설정**
```bash
vi /etc/mail/sendmail.cf
#  Cwhistory.com  (85)
#  Fw/etc/mail/local-host-names  (87)
#  O DaemonPortOptions=Port=smtp, Name=MTA  (268)

vi /etc/mail/access
#  192.168.111  RELAY / history.com  RELAY
makemap hash /etc/mail/access < /etc/mail/access
```

**9~11. 수신 설정 (Dovecot)**
```bash
vi /etc/dovecot/dovecot.conf
#  protocols = imap pop3 lmtp submission
#  listen = *, ::
#  base_dir = /var/run/dovecot/

vi /etc/dovecot/conf.d/10-ssl.conf      # ssl = yes
vi /etc/dovecot/conf.d/10-mail.conf
#  mail_location = mbox:~/mail:INBOX=/var/mail/%u
#  mail_access_groups = mail
#  lock_method = fcntl
```

```bash
# 12. 서비스 시작
systemctl restart named && systemctl enable named

# 13. 방화벽 허가
firewall-cmd --permanent --add-service=smtp
firewall-cmd --permanent --add-service=pop3
firewall-cmd --permanent --add-service=imap
firewall-cmd --reload                   # imap pop3 smtp 확인
```

**14~15. Evolution 등록 & 메일 발송** — 수신 POP `mail.history.com:995`, 발신 SMTP `mail.history.com:25`로 계정 등록 후 `rocky@mail.history.com` 발송 성공.

> 메일 서버는 **DNS(MX) → Sendmail(SMTP) → Dovecot(POP3/IMAP)** 연동이 모두 갖춰져야 동작.

---

# MariaDB Server

MariaDB 구축 + 원격 접속 허용, 클라이언트(Project-B)에서 원격 조회.

**서버 측 (Project)**
```bash
# 1. 설치
rpm -qa mariadb-server
dnf install mariadb-server

# 2. 서비스 시작
systemctl restart mariadb
systemctl enable mariadb

# 3. 방화벽 개방
firewall-cmd --permanent --add-service=mysql
firewall-cmd --reload

# 4. DB 비밀번호 설정
mysqladmin -u root password '1234'

# 5. DB 접속
mysql -h localhost -u root -p
```
```sql
-- 6. 원격 접속 권한 설정
grant all on *.* to root@'%' identified by '1234';
FLUSH PRIVILEGES;

-- 7. DB 생성
CREATE DATABASE history CHARACTER SET utf8;

-- 8. 조회
show databases;   -- history, information_schema, mysql, performance_schema
```

**클라이언트 측 (Project-B)**
```bash
dnf -y install mariadb
mysql -h 192.168.111.100 -u root -p
```
```sql
show databases;   -- history DB가 원격에서도 조회됨
```

> `root@'%'` 권한 부여 + `mysql` 방화벽 개방으로 외부 접속을 허용, 서버의 `history` DB가 클라이언트에서 정상 조회됨을 확인.

---

## 전체 정리

- 서버마다 공통적으로 **설치 → 설정 → 기동 → 방화벽 → 검증** 흐름을 적용했다.
- DNS의 CNAME/MX 레코드가 Web·FTP·Mail 서비스와 어떻게 연동되는지 end-to-end로 확인했다.
- Rocky Linux 환경에서 firewalld와 **SELinux**를 함께 고려해야 정상 동작하는 서비스(Samba 등)를 경험했다.
