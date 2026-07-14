# 02. 역할 기반 제어 (RBAC)

[← 목차로 돌아가기](../README.md)

ServiceAccount에 네임스페이스 단위(Role) 또는 클러스터 단위(ClusterRole) 권한을 부여하는 RBAC 구성 작업입니다.

---

## 1. ServiceAccount + Role + RoleBinding

`api-access` 네임스페이스의 모든 Pod를 조회(view)할 수 있는 서비스 계정을 만듭니다.

- `api-access` 네임스페이스에 `pod-viewer` ServiceAccount 생성
- `podreader-role`(Role), `podreader-rolebinding`(RoleBinding) 생성
- Pod 리소스에 대해 `watch, list, get` 허용

```bash
# 네임스페이스
kubectl create namespace api-access

# ServiceAccount
kubectl create serviceaccount pod-viewer -n api-access

# Role (Pod에 대한 watch/list/get)
kubectl create role podreader-role \
  --resource=pod \
  --verb=watch,list,get \
  --namespace=api-access

# RoleBinding (Role ↔ ServiceAccount 매핑)
kubectl create rolebinding podreader-rolebinding \
  --role=podreader-role \
  --serviceaccount=api-access:pod-viewer \
  --namespace=api-access
```

확인:

```bash
kubectl get role,rolebinding -n api-access
```

---

## 2. ServiceAccount + ClusterRole + ClusterRoleBinding

애플리케이션 배포용 ClusterRole을 만들고 특정 네임스페이스의 ServiceAccount에 바인딩합니다.

- `Deployment`, `StatefulSet`, `DaemonSet`에 대해서만 `create`가 허용된 ClusterRole `deployment-clusterrole` 생성
- `api-access` 네임스페이스에 `cicd-token` ServiceAccount 생성
- ClusterRole을 `cicd-token`에 바인딩

```bash
# ServiceAccount
kubectl create serviceaccount cicd-token --namespace=api-access

# ClusterRole (지정한 3개 리소스에 대해서만 create)
kubectl create clusterrole deployment-clusterrole \
  --resource=deployment,statefulset,daemonset \
  --verb=create

# ClusterRoleBinding
kubectl create clusterrolebinding deployment-clusterrolebinding \
  --clusterrole=deployment-clusterrole \
  --serviceaccount=api-access:cicd-token
```

확인:

```bash
kubectl get clusterrole | grep deployment-clusterrole
kubectl get clusterrolebinding | grep deployment-clusterrolebinding
```

> **Role vs ClusterRole**: Role/RoleBinding은 특정 네임스페이스 범위에서만 유효하고, ClusterRole은 클러스터 전역 리소스에 대해 정의할 수 있습니다. ClusterRole을 ClusterRoleBinding 대신 RoleBinding으로 묶으면 특정 네임스페이스로 권한 범위를 제한할 수도 있습니다.

➡️ 다음: [03. 노드 관리](03-node-management.md)
