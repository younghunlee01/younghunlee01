# 03. 노드 관리

[← 목차로 돌아가기](../README.md)

노드를 스케줄링 대상에서 제외하거나, 라벨 기반으로 특정 노드에 파드를 배치하는 작업입니다.

---

## 1. 노드 비우기 (Drain / Cordon / Uncordon)

`worker-2` 노드를 스케줄링 불가능하게 설정하고, 실행 중이던 모든 Pod를 다른 노드로 재배치(reschedule)합니다.

```bash
# 현재 노드 확인
kubectl get node

# worker-2를 cordon 처리하고 Pod를 다른 노드로 축출
kubectl drain worker-2 --ignore-daemonsets

# 상태 확인 → worker-2가 SchedulingDisabled 로 표시됨
kubectl get node

# 다시 스케줄링 가능하게 복구
kubectl uncordon worker-2
```

- `drain`은 내부적으로 `cordon`(신규 스케줄링 차단) + 기존 Pod 축출(eviction)을 수행합니다.
- `--ignore-daemonsets`는 DaemonSet이 관리하는 Pod(예: calico-node)를 축출 대상에서 제외합니다.

---

## 2. Pod Scheduling (nodeSelector)

라벨을 이용해 특정 노드에만 Pod를 배치합니다.

- name: `eshop-store`
- image: `nginx`
- nodeSelector: `disktype=ssd`

### 노드 라벨 부여

```bash
kubectl label node worker-1 disktype=ssd
kubectl label node worker-2 disktype=hdd
kubectl get no -L disktype
```

### Pod 매니페스트 생성 및 수정

```bash
kubectl run eshop-store \
  --image=ghcr.io/mjc-t/nginx \
  --dry-run=client -o yaml > eshop-store.yaml
```

```yaml
# eshop-store.yaml
apiVersion: v1
kind: Pod
metadata:
  name: eshop-store
spec:
  containers:
  - image: ghcr.io/mjc-t/nginx
    name: eshop-store
  nodeSelector:
    disktype: ssd
```

### 생성 및 확인

```bash
kubectl apply -f eshop-store.yaml
kubectl get po eshop-store -o wide
# NODE → worker-1 (disktype=ssd 라벨을 가진 노드에 배치됨)
```
