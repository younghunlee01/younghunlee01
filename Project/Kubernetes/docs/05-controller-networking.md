# 05. Kubernetes Controller & Networking

[← 목차로 돌아가기](../README.md)

> ⭐ **담당 파트** — Deployment 컨트롤러의 롤링 업데이트/롤백과 Service(ClusterIP·NodePort), 스케일 아웃까지 애플리케이션 배포·노출·확장의 전 과정을 다룹니다.

---

## 1. Rolling Update & Roll Back

Deployment로 nginx 파드 3개를 배포한 뒤 컨테이너 이미지 버전을 롤링 업데이트하고, 다시 이전 버전으로 롤백합니다.

- name: `eshop-payment`
- image: `nginx`, version `1.16` → update `1.17`
- label: `app=payment`, `environment=production`

### Deployment 매니페스트 생성

```bash
kubectl create deploy eshop-payment \
  --image=ghcr.io/mjc-t/nginx:1.16 \
  --replicas=3 \
  --dry-run=client -o yaml > eshop-payment.yaml
```

```yaml
# eshop-payment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: payment
    environment: production
  name: eshop-payment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
      environment: production
  template:
    metadata:
      labels:
        app: payment
        environment: production
    spec:
      containers:
      - image: ghcr.io/mjc-t/nginx:1.16
        name: nginx
```

### 생성

```bash
kubectl apply -f eshop-payment.yaml --record
kubectl get deployment,pod
# eshop-payment  3/3  Running
```

> `--record`는 `kubectl` 명령을 리비전 히스토리의 CHANGE-CAUSE로 남깁니다. (현재 deprecated 되었으나 롤아웃 이력 확인 실습을 위해 사용)

### 롤링 업데이트

```bash
# 이미지 버전 1.16 → 1.17
kubectl set image deployments eshop-payment nginx=ghcr.io/mjc-t/nginx:1.17 --record

# 리비전 이력 확인
kubectl rollout history deployment eshop-payment
# REVISION 1: apply --record=true
# REVISION 2: set image ... 1.17 --record=true
```

### 롤백

```bash
kubectl rollout undo deployment eshop-payment

# 확인 → 이미지가 다시 1.16 으로 돌아옴
kubectl get pods
kubectl describe pod | grep -i image:
kubectl rollout history deployment eshop-payment
```

롤백 후 파드의 이미지가 `ghcr.io/mjc-t/nginx:1.16`으로 복원되고, 리비전 번호가 새로 부여됩니다.

---

## 2. Service — ClusterIP

`devops` 네임스페이스에 Deployment `eshop-order`를 만들고 ClusterIP 서비스로 노출합니다.

**Deployment**
- name: `eshop-order`, namespace: `devops`, image: `nginx`, replicas: `2`, label: `name=order`

**Service**
- serviceName: `eshop-order-svc`, type: `ClusterIP`, port: `80`

```bash
kubectl create namespace devops

kubectl create deployment eshop-order \
  --image=ghcr.io/mjc-t/nginx \
  --replicas=2 \
  --namespace=devops \
  --dry-run=client -o yaml > eshop-order.yaml
```

```yaml
# eshop-order.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: order
  name: eshop-order
  namespace: devops
spec:
  replicas: 2
  selector:
    matchLabels:
      name: order
  template:
    metadata:
      labels:
        name: order
    spec:
      containers:
      - image: ghcr.io/mjc-t/nginx
        name: nginx
```

```bash
kubectl apply -f eshop-order.yaml
kubectl get deploy,pod -n devops

# ClusterIP 서비스 노출
kubectl expose deployment -n devops eshop-order \
  --name=eshop-order-svc \
  --port=80 \
  --target-port=80

kubectl get svc -n devops
# eshop-order-svc  ClusterIP  10.101.82.59  <none>  80/TCP

# 클러스터 내부에서 접근 확인
curl 10.101.82.59   # → Welcome to nginx!
```

> **ClusterIP**는 클러스터 내부에서만 접근 가능한 가상 IP입니다. 외부 노출은 필요 없고 파드 간 통신만 필요할 때 사용합니다.

---

## 3. Service — NodePort

`front-end` Deployment를 만들고, 각 노드의 `30200` 포트로 접근 가능한 NodePort 서비스를 만듭니다.

**Deployment**
- image: `nginx`, replicas: `2`, label: `run=nginx`

**Service**
- name: `front-end-nodesvc`, nodePort: `30200`

```bash
kubectl create deployment front-end \
  --image ghcr.io/mjc-t/nginx \
  --replicas 2 \
  --dry-run=client -o yaml > front-end.yaml
kubectl apply -f front-end.yaml

# NodePort 서비스 매니페스트 생성
kubectl expose deployment front-end --name front-end-nodesvc \
  --port 80 --target-port 80 --type NodePort \
  --dry-run=client -o yaml > front-end-nodesvc.yaml
```

```yaml
# front-end-nodesvc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx
  name: front-end-nodesvc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30200
  selector:
    run: nginx
  type: NodePort
```

```bash
kubectl apply -f front-end-nodesvc.yaml
kubectl get svc
# front-end-nodesvc  NodePort  10.100.62.38  80:30200/TCP

# 노드 IP:NodePort 로 접근 확인
curl 192.168.111.20:30200   # → Welcome to nginx!
```

> **NodePort**는 모든 노드의 지정 포트(30000–32767)를 열어 외부에서 `노드IP:포트`로 접근할 수 있게 합니다. ClusterIP 위에 얹혀 동작합니다.

---

## 4. Pod Scale-out

실행 중인 `eshop-order` Deployment의 파드 수를 2 → 5로 늘립니다.

```bash
kubectl get deployment -n devops
# eshop-order  2/2

kubectl scale deploy eshop-order -n devops --replicas=5

kubectl get deploy,pod -n devops
# eshop-order  5/5  → 파드 5개 Running
```

> `kubectl scale`은 Deployment의 `replicas` 필드를 즉시 변경합니다. 트래픽 증가에 대응하는 수평 확장(Scale-out)의 기본 방식이며, 부하 기반 자동 확장이 필요하면 HPA(HorizontalPodAutoscaler)를 사용합니다.
