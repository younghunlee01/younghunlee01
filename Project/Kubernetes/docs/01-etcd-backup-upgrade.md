# 01. ETCD Backup & Upgrade

[← 목차로 돌아가기](../README.md)

클러스터의 상태 저장소인 etcd를 스냅샷으로 백업/복구하고, `kubeadm`으로 control plane과 노드 구성 요소를 업그레이드하는 작업입니다.

---

## 1. ETCD Backup

`https://127.0.0.1:2379`에서 실행 중인 etcd의 스냅샷을 생성해 `etcd-snapshot.db`로 저장하고, 다시 복원합니다. etcdctl로 서버에 접속하려면 아래 TLS 인증서/키가 필요합니다.

- CA certificate: `/etc/kubernetes/pki/etcd/ca.crt`
- Client certificate: `/etc/kubernetes/pki/etcd/server.crt`
- Client key: `/etc/kubernetes/pki/etcd/server.key`

### 사전 작업

```bash
sudo -i                      # root 전환
apt install etcd-client      # etcdctl 설치
```

### 스냅샷 생성

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save etcd-snapshot.db
```

### 스냅샷 복구

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot restore etcd-snapshot.db
```

복구 시 `default.etcd/` 디렉토리가 생성되며, 내부에 `member/snap`, `member/wal`이 포함됩니다.

---

## 2. Cluster Upgrade

master 노드의 모든 control plane 및 노드 구성 요소를 업그레이드합니다. 업그레이드 전 반드시 `drain`, 완료 후 `uncordon` 하며, **Master Node에서 root 권한으로 실행**해야 합니다.

> 실습 환경이 이미 `1.33.11-1.1`이라 본 프로젝트에서는 `1.34.7-1.1`로 업그레이드했습니다.

### 현재 버전 확인

```bash
kubectl get nodes
# master / worker-1 / worker-2 → v1.33.11
```

### 업그레이드 가능 버전 확인

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

### kubeadm 업그레이드

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y '1.34.7-1.1' && \
sudo apt-mark hold kubeadm

kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.34.7-1.1
```

### 노드 Drain

```bash
kubectl drain master --ignore-daemonsets
```

### kubelet / kubectl 업그레이드 & 재시작

```bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet='1.34.7-1.1' kubectl='1.34.7-1.1'
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Uncordon & 확인

```bash
kubectl uncordon master
kubectl get node
# master → v1.34.7 (worker 노드는 순차 진행)
```

### 참고

1. 워커 노드의 kubelet은 마스터보다 최대 2단계 낮아도 호환됩니다.
2. 다만 안정성과 신규 기능 활용을 위해 버전을 동일하게 맞추는 것을 권장합니다.
3. 워커 노드도 동일한 방식(`unhold → install → hold`)으로 순차 업데이트합니다.

➡️ 다음: [02. 역할 기반 제어(RBAC)](02-rbac.md)
