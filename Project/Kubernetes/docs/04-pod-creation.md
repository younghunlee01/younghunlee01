# 04. 파드 생성

[← 목차로 돌아가기](../README.md)

환경변수/커맨드 지정, Static Pod, 로그 추출, 멀티 컨테이너 파드 등 다양한 파드 생성 패턴을 다룹니다.

---

## 1. 환경변수 · command · args 적용

`cka-exam` 네임스페이스에 아래 조건의 Pod를 생성합니다.

- podName: `pod-01`
- image: `busybox`
- env: `CERT = "CKA-cert"`
- command: `/bin/sh`
- args: `"-c", "while true; do echo $(CERT); sleep 10; done"`

```bash
kubectl create namespace cka-exam

kubectl run pod-01 \
  --image=ghcr.io/mjc-t/busybox \
  --env=CERT="CKA-cert" \
  --namespace=cka-exam \
  --dry-run=client -o yaml > pod-01.yaml
```

```yaml
# pod-01.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-01
  namespace: cka-exam
spec:
  containers:
  - image: ghcr.io/mjc-t/busybox
    name: pod-01
    env:
    - name: CERT
      value: "CKA-cert"
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(CERT); sleep 10;done"]
```

```bash
kubectl apply -f pod-01.yaml
kubectl get pods --namespace=cka-exam
```

---

## 2. Static Pod 생성

`worker-1` 노드에 `nginx-static-pod`라는 Static Pod를 생성합니다.

- podName: `nginx-static-pod`
- image: `nginx`
- port: `80`

Static Pod는 kubelet이 노드의 매니페스트 디렉토리(`/etc/kubernetes/manifests`)를 직접 감시해 관리합니다. 따라서 매니페스트를 해당 노드에 배치해야 합니다.

```yaml
# nginx-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-pod
spec:
  containers:
  - image: ghcr.io/mjc-t/nginx
    name: nginx-static-pod
    ports:
    - containerPort: 80
```

```bash
# worker-1 노드에서 (root)
sudo -i
cd /etc/kubernetes/manifests
vi nginx-static-pod.yaml   # 위 내용 저장

# master 노드에서 확인 → 이름 뒤에 노드명이 붙음
kubectl get pods -o wide
# nginx-static-pod-worker1  Running  ...  worker1
```

---

## 3. 로그 확인

Pod의 로그를 모니터링하고 파일로 추출합니다.

```bash
kubectl get pods

# 로그 모니터링
kubectl logs nginx-static-pod-worker1

# 로그를 파일로 저장
kubectl logs nginx-static-pod-worker1 > /home/cka/pod-log
cat /home/cka/pod-log
```

---

## 4. Multi Container Pod 생성

4개의 컨테이너를 동작시키는 `eshop-frontend` Pod를 생성합니다. (nginx, redis, memcached, busybox)

```yaml
# eshop-frontend.yaml
apiVersion: v1
kind: Pod
metadata:
  name: eshop-frontend
spec:
  containers:
  - image: ghcr.io/mjc-t/nginx
    name: nginx
  - image: ghcr.io/mjc-t/redis:3.2-alpine
    name: redis
  - image: ghcr.io/mjc-t/memcached:latest
    name: memcached
  - image: ghcr.io/mjc-t/busybox:1.28
    name: busybox
    command: ["sh", "-c", "sleep 3600"]
```

```bash
kubectl apply -f eshop-frontend.yaml
kubectl get pods
# eshop-frontend  4/4  Running
```

> **busybox** 이미지는 실행 중인 foreground 프로세스가 있어야 컨테이너가 유지되므로 `sleep 3600` 같은 command를 지정합니다.

➡️ 다음: [05. Kubernetes Controller & Networking](05-controller-networking.md)
