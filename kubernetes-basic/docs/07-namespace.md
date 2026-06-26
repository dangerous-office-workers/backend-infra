# Namespace

하나의 클러스터 안에서 리소스를 논리적으로 구분하는 Namespace 사용 흐름을 정리한다.

[← README](../README.md)

---

## 19. Kubernetes Namespace

Namespace는 하나의 Kubernetes 클러스터 안에서 리소스를 논리적으로 구분하는 공간이다.

환경이나 팀별로 Deployment, Pod, Service 등의 리소스를 나누어 관리할 수 있다.

```text
Kubernetes Cluster
 ├─ dev Namespace
 ├─ test Namespace
 └─ prod Namespace
```

### Namespace 조회
```bash
kubectl get namespaces
kubectl get ns
```

Namespace를 별도로 지정하지 않으면 리소스는 기본적으로 `default` Namespace에 생성된다.

```bash
kubectl get pods -n default
```

### Namespace 생성
```bash
kubectl create namespace study

# 확인
kubectl get ns
```

### 특정 Namespace에 배포
```bash
kubectl apply -f deployment.yaml -n study
kubectl apply -f service.yaml -n study

# 확인
kubectl get all -n study
```

### Namespace별 리소스 비교
```bash
kubectl get all -n default
kubectl get all -n study

# 서로 다른 Namespace에서는 동일한 이름의 리소스를 사용할 수 있다.
# default/nginx-deployment
# study/nginx-deployment
```

### 모든 Namespace 조회
```bash
kubectl get pods --all-namespaces
kubectl get pods -A
```

리소스가 보이지 않을 때 다른 Namespace에 생성된 것은 아닌지 확인할 수 있다.

### Namespace를 지정한 명령어
```bash
kubectl get pods -n study
kubectl logs -n study <pod-name>
kubectl describe pod -n study <pod-name>
```

### port-forward
```bash
kubectl port-forward -n study service/nginx-service 8081:80
```
- 브라우저 http://localhost:8081/ 에서 확인

### YAML에 Namespace 명시
```yaml
metadata:
  name: nginx-deployment
  namespace: study
```

YAML에 Namespace를 지정하면 `kubectl apply` 실행 시 `-n` 옵션을 생략할 수 있다.

### Service 이름과 Namespace
- 같은 Namespace 안에서는 Service 이름만으로 호출할 수 있다. `nginx-service`
- 다른 Namespace의 Service는 Namespace를 포함해 호출할 수 있다. `nginx-service.study`

### Namespace 삭제
```bash
kubectl delete namespace study
```

Namespace를 삭제하면 해당 Namespace 안의 Deployment, Pod, Service 등의 리소스도 함께 삭제된다.

### 주의
- Namespace는 리소스를 논리적으로 구분하지만, 네트워크와 권한을 자동으로 완전히 격리하지는 않는다.
- 더 강한 제어에는 다음 설정이 사용될 수 있다.
```text
RBAC
NetworkPolicy
ResourceQuota
LimitRange
```
