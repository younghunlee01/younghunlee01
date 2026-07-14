# 07. 리소스 관리

[← 목차로 돌아가기](../README.md)

emptyDir, hostPath, PersistentVolume, PVC 등 파드에 스토리지를 연결하는 여러 볼륨 유형을 다룹니다.

---

## 1. emptyDir Volume

nginx 웹서버가 만든 로그를 busybox 컨테이너가 받아 STDOUT으로 출력하는, `emptyDir`을 통한 컨테이너 간 데이터 공유 예제입니다.

- podName: `weblog`
- Web container: `nginx:1.17`, volumeMount `/var/log/nginx` (readwrite)
- Log container: `busybox`, args `/bin/sh -c "tail -n+1 -f /data/access.log"`, volumeMount `/data` (readonly)

```yaml
# weblog.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: weblog
  name: weblog
spec:
  containers:
  - image: ghcr.io/mjc-t/nginx:1.17
    name: web
    volumeMounts:
    - mountPath: /var/log/nginx
      name: weblog
  - image: ghcr.io/mjc-t/busybox
    name: log
    args: ["/bin/sh", "-c", "tail -n+1 -f /data/access.log"]
    volumeMounts:
    - mountPath: /data
      name: weblog
      readOnly: true
  volumes:
  - name: weblog
    emptyDir: {}
```

```bash
kubectl apply -f weblog.yaml
kubectl get pods
# weblog  2/2  Running
```

> **emptyDir**은 파드가 노드에 스케줄될 때 생성되어 파드가 살아있는 동안만 존재하는 임시 볼륨입니다. 같은 파드 내 컨테이너들이 데이터를 공유할 때 유용하며, 한쪽은 쓰기(web) 다른 쪽은 읽기 전용(log)으로 마운트했습니다.

---

## 2. HostPath Volume

`fluentd` 파드를 만들어 워커 노드의 특정 디렉토리를 파드에 마운트합니다.

- podName: `fluentd`, image: `fluentd`, namespace: `default`
1. 워커 노드의 `/var/lib/docker/containers`를 동일 경로로 마운트
2. 워커 노드의 `/var/log`를 동일 이름으로 마운트

```yaml
# fluentd.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: fluentd
  name: fluentd
spec:
  containers:
  - image: ghcr.io/mjc-t/fluentd
    name: fluentd
    ports:
    - containerPort: 80
    volumeMounts:
    - name: containersdir
      mountPath: /var/lib/docker/containers
    - name: logdir
      mountPath: /var/log
  volumes:
  - name: containersdir
    hostPath:
      path: /var/lib/docker/containers
  - name: logdir
    hostPath:
      path: /var/log
```

```bash
kubectl apply -f fluentd.yaml
kubectl get pods
# fluentd  1/1  Running
```

> **hostPath**는 노드의 실제 파일시스템 경로를 파드에 연결합니다. 로그 수집기(fluentd)처럼 노드 로그에 접근해야 하는 워크로드에서 사용하지만, 파드가 노드에 종속되므로 일반 애플리케이션 데이터 저장용으로는 지양합니다.

---

## 3. Persistent Volume

`pv001`이라는 이름의 PersistentVolume을 생성합니다.

- size: `1Gi`, accessMode: `ReadWriteMany`
- type: `hostPath`, path: `/tmp/app-config`

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: "/tmp/app-config"
```

```bash
kubectl apply -f pv.yaml
kubectl get pv
# pv001  1Gi  RWX  Retain  Available
```

---

## 4. PVC를 사용하는 애플리케이션 Pod 운영

PersistentVolumeClaim을 만들어 위 PV와 바인딩하고, 이를 마운트하는 파드를 운영합니다.

**PVC**
- name: `pv-volume`, capacity: `10Mi`, accessMode: `ReadWriteMany`

**Pod**
- name: `web-server-pod`, image: `nginx`, mountPath: `/usr/share/nginx/html`

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
# pv-volume  Bound  pv001  1Gi  RWX
```

PVC가 조건(accessMode, 용량)에 맞는 PV(`pv001`)와 자동으로 바인딩됩니다.

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
  - name: nginx
    image: ghcr.io/mjc-t/nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: mypd
  volumes:
  - name: mypd
    persistentVolumeClaim:
      claimName: pv-volume
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
# web-server-pod  1/1  Running
```

> **PV / PVC 분리**: PV는 관리자가 제공하는 실제 스토리지 자원, PVC는 사용자가 요청하는 스토리지 명세입니다. 파드는 PVC만 참조하므로 스토리지 구현(hostPath, NFS, 클라우드 디스크 등)과 애플리케이션이 느슨하게 결합됩니다.
