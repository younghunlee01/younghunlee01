# Kubernetes Project

Kubernetes 클러스터 운영 실습 프로젝트입니다. 클러스터 백업/업그레이드부터 RBAC, 스케줄링, 파드 구성, 컨트롤러, 네트워킹, 스토리지까지 CKA(Certified Kubernetes Administrator) 시험 범위에 해당하는 주요 작업을 직접 수행하고 정리했습니다.

- **환경**: `master`(control-plane) 1대 + `worker-1`, `worker-2` 2대 구성
- **CNI**: Calico
- **컨테이너 이미지**: 사내 레지스트리 미러(`ghcr.io/mjc-t/...`) 사용


---

## 기술 스택

| 구분 | 사용 기술 |
|------|-----------|
| Container Orchestration | Kubernetes (kubeadm), kubectl |
| Container Runtime | containerd |
| Networking (CNI) | Calico |
| Cluster Store | etcd |
| Ingress | NGINX Ingress Controller |
| Container Registry | GitHub Container Registry (ghcr.io) |
| Application Images | nginx, redis, memcached, busybox, fluentd |
| OS / Infra | Ubuntu (Linux), systemd |

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![etcd](https://img.shields.io/badge/etcd-419EDA?style=flat&logo=etcd&logoColor=white)
![Calico](https://img.shields.io/badge/Calico-FF7300?style=flat&logo=projectcalico&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=flat&logo=nginx&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)

---

## 전체 목차

| # | 섹션 | 핵심 키워드 |
|---|------|-------------|
| 01 | [ETCD Backup & Upgrade](docs/01-etcd-backup-upgrade.md) | etcd 스냅샷 백업·복구, `kubeadm` 클러스터 업그레이드 |
| 02 | [역할 기반 제어 (RBAC)](docs/02-rbac.md) | ServiceAccount, Role/RoleBinding, ClusterRole/ClusterRoleBinding |
| 03 | [노드 관리](docs/03-node-management.md) | `drain`/`cordon`/`uncordon`, `nodeSelector` 스케줄링 |
| 04 | [파드 생성](docs/04-pod-creation.md) | env·command·args, Static Pod, 로그 추출, Multi-Container Pod |
| **05** | **[Kubernetes Controller & Networking](docs/05-controller-networking.md)** | **Rolling Update/Rollback, ClusterIP, NodePort, Scale-out** |
| **06** | **[NetworkPolicy & Ingress](docs/06-networkpolicy-ingress.md)** | **NetworkPolicy, Ingress, Service/Pod DNS Lookup** |
| 07 | [Resource 관리](docs/07-resource-management.md) | emptyDir, hostPath, PersistentVolume, PVC |

---

## 담당 파트

이 저장소에서 **제가 담당한 파트는 아래 두 섹션**입니다.

- **[05. Kubernetes Controller & Networking](docs/05-controller-networking.md)** — Deployment 롤링 업데이트/롤백, ClusterIP·NodePort 서비스, 파드 스케일 아웃
- **[06. NetworkPolicy & Ingress](docs/06-networkpolicy-ingress.md)** — 네임스페이스 기반 트래픽 격리(NetworkPolicy), Ingress 라우팅, 서비스/파드 DNS 조회

---

## 사용 명령어 요약

```bash
# etcd 스냅샷 백업
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save etcd-snapshot.db

# 클러스터 업그레이드
sudo kubeadm upgrade apply v1.34.7-1.1

# RBAC
kubectl create role podreader-role --resource=pod --verb=watch,list,get -n api-access

# 스케줄링
kubectl drain worker-2 --ignore-daemonsets
kubectl label node worker-1 disktype=ssd

# 컨트롤러 & 네트워킹
kubectl set image deployments eshop-payment nginx=<img>:1.17 --record
kubectl expose deployment eshop-order --type=NodePort --port=80

# 스토리지
kubectl get pv,pvc
```

각 명령어의 전체 맥락과 실행 결과는 위 목차의 개별 문서에서 확인할 수 있습니다.
