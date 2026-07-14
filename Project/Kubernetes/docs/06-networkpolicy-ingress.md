# 06. NetworkPolicy & Ingress

[← 목차로 돌아가기](../README.md)

> ⭐ **담당 파트** — 네임스페이스 라벨 기반 트래픽 격리(NetworkPolicy), Ingress 경로 라우팅, 클러스터 DNS를 통한 Service/Pod 이름 조회를 다룹니다.

---

## 1. Network Policy

`customera`, `customerb` 네임스페이스를 만들고 각각 `partition` 라벨을 부여한 뒤, `default` 네임스페이스의 `poc` 파드에는 **`partition=customera` 네임스페이스에서만** 80포트로 접근할 수 있도록 격리합니다.

**Pod**
- name: `poc`, image: `nginx`, port: `80`, label: `app=poc`

### 네임스페이스 생성 & 라벨링

```bash
kubectl create ns customera
kubectl create ns customerb

kubectl label ns customera partition=customera
kubectl label ns customerb partition=customerb

kubectl get ns --show-labels | grep customer
```

### NetworkPolicy 작성

```yaml
# netpol.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-from-customera
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: poc            # 이 정책이 적용될 대상 파드
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          partition: customera   # 이 라벨을 가진 네임스페이스만 허용
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl apply -f netpol.yaml
kubectl get networkpolicy
# allow-web-from-customera  app=poc
```

> **동작 원리**: NetworkPolicy가 특정 파드에 처음 적용되는 순간, 명시적으로 허용(allow)하지 않은 모든 인그레스 트래픽은 차단됩니다. 위 정책은 `partition=customera` 라벨이 붙은 네임스페이스에서 오는 TCP:80만 `poc` 파드로 통과시키고, `customerb`를 포함한 나머지는 모두 차단합니다. (NetworkPolicy는 Calico 등 이를 지원하는 CNI가 있어야 강제됩니다.)

---

## 2. Ingress

nginx Ingress 리소스를 만들어 `hi` 서비스를 `/hi` 경로로 노출합니다.

- name: `ping`, namespace: `ing-internal`
- backend service: `hi`, port: `5678`, path: `/hi`

```bash
kubectl create namespace ing-internal
```

```yaml
# ing.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ping
  namespace: ing-internal
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /hi
        pathType: Prefix
        backend:
          service:
            name: hi
            port:
              number: 5678
```

```bash
kubectl apply -f ing.yaml
kubectl get ingress -n ing-internal
# ping  nginx-example  *  80
```

> **Ingress**는 L7(HTTP/HTTPS) 라우팅 규칙을 정의하는 리소스로, 호스트/경로 기준으로 트래픽을 여러 서비스로 분기합니다. 실제 라우팅은 Ingress Controller(nginx 등)가 이 규칙을 읽어 수행합니다.

---

## 3. Service and DNS Lookup

`nginx` 파드(`resolver`)와 `resolver-service`를 만들고, 클러스터 DNS로 서비스명과 파드명을 조회할 수 있는지 테스트합니다.

- DNS 조회용 이미지: `busybox:1.28`
- 조회 도구: `nslookup`
- 결과 저장: 서비스 → `~/nginx.svc`, 파드 → `~/nginx.pod`

### Pod & Service 생성

```bash
kubectl run resolver --image ghcr.io/mjc-t/nginx --port 80
kubectl get pods resolver

kubectl expose pod resolver --name resolver-service --port 80
kubectl get svc resolver-service
# resolver-service  ClusterIP  10.108.94.23  80/TCP
```

### DNS Lookup — Service

```bash
kubectl run -it test-nslookup --image=ghcr.io/mjc-t/busybox:1.28 \
  --restart=Never --rm \
  -- nslookup 10.108.94.23 > ~/nginx.svc
```

결과 예시(`~/nginx.svc`):

```
Name:  10.108.94.23
Address 1: 10.108.94.23 resolver-service.default.svc.cluster.local
```

### DNS Lookup — Pod

파드는 IP의 `.`을 `-`로 바꾼 이름으로 조회합니다. (`10.244.235.191` → `10-244-235-191`)

```bash
kubectl run -it test-nslookup --image=ghcr.io/mjc-t/busybox:1.28 \
  --restart=Never --rm \
  -- nslookup 10-244-235-191.default.pod.cluster.local > ~/nginx.pod
```

결과 예시(`~/nginx.pod`):

```
Name:  10-244-235-191.default.pod.cluster.local
Address 1: 10.244.235.191
```

> **클러스터 DNS(CoreDNS)** 명명 규칙
> - Service: `<service>.<namespace>.svc.cluster.local`
> - Pod: `<pod-ip-with-dashes>.<namespace>.pod.cluster.local`

➡️ 다음: [07. 리소스 관리](07-resource-management.md)
