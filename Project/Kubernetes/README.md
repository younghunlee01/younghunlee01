# Kubernetes Project

## 프로젝트 개요

Kubernetes 클러스터를 직접 구축·운영하며 CKA(Certified Kubernetes Administrator) 시험 범위에 해당하는 핵심 운영 작업을 실습·정리한 프로젝트입니다. control-plane 1대와 worker 2대로 구성된 멀티 노드 환경에서, 클러스터의 안정적인 운영을 위한 백업·복구부터 애플리케이션 배포와 네트워킹, 스토리지 관리까지 전 주기를 다뤘습니다.

- **클러스터 운영**: etcd 스냅샷 백업/복구, `kubeadm`을 이용한 클러스터 버전 업그레이드 (drain → upgrade → uncordon)
- **접근 제어**: ServiceAccount 기반 RBAC 구성 (Role/RoleBinding, ClusterRole/ClusterRoleBinding)
- **워크로드 배치**: `nodeSelector`·라벨을 활용한 스케줄링, Static Pod, Multi-Container Pod, 환경변수/커맨드 주입
- **컨트롤러 & 네트워킹**: Deployment 롤링 업데이트/롤백, ClusterIP·NodePort 서비스, Pod Scale-out
- **네트워크 정책**: 네임스페이스 라벨 기반 트래픽 격리(NetworkPolicy), Ingress 경로 라우팅, CoreDNS 이름 조회
- **스토리지**: emptyDir·hostPath 볼륨, PersistentVolume/PVC 연동


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

## 기술 스택

`Kubernetes` · `kubeadm` · `kubectl` · `containerd` · `etcd` `Calico(CNI)` · `NGINX Ingress` · `NetworkPolicy` · `CoreDNS` `PersistentVolume` · `PVC` · `hostPath` · `emptyDir` `RBAC` · `ghcr.io` · `Ubuntu(Linux)`

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
