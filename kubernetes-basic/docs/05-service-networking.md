# Service, Label, Selector, 네트워킹

Pod 앞단의 고정 진입점인 Service와 Label/Selector 기반 연결 방식을 정리한다.

[← README](../README.md)

---

## 13. Service
- Service는 Pod의 고정 진입점 역할을 한다.
- Pod는 재생성되면 IP가 변경될 수 있다.
- Service는 사용자가 Pod의 IP를 직접 알지 않아도 되도록 대표 주소를 제공한다.
```text
사용자
    ↓
Service
    ↓
Pod
```
### Service의 역할
- 고정 주소 제공
- 로드밸런싱
- Pod 연결 관리

### 정리
- Pod IP는 변경될 수 있다.
- Service는 Pod 앞단의 대표 진입점이다.
- 사용자는 Service를 통해 애플리케이션에 접근한다.
---

## 14. Kubernetes 포트 연결
- Docker에서는 `-p` 옵션으로 호스트 포트와 컨테이너 포트를 연결했다.
  ```bash
  docker run -d -p 8080:80 nginx
  ```
  - `localhost:8080` -> `Container:80`
- Kubernetes에서는 Service를 통해 Pod의 컨테이너 포트로 요청을 전달한다.
  - 사용자 -> Service -> Pod -> Container

### Service 포트 예시
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
- `targetPort` : Pod 안의 컨테이너가 실제로 요청을 받는 포트
- `port` : Service가 클러스터 내부에서 열어두는 포트. 다른 Pod가 이 Service를 호출할 때 사용할 수 있음
- `nodePort` : 클러스터 외부에서 접근할 때 사용하는 포트. Docker Desktop 로컬 Kubernetes에서는 `http://localhost:30080`으로 접근

### 전체 흐름
```text
브라우저 localhost:30080
 ↓
NodePort 30080
 ↓
Service port 80
 ↓
Pod targetPort 80
 ↓
nginx Container
```
- Docker
  ```text
  localhost:8080
   ↓
  Container:80
  ```
- Kubernetes
  ```text
  localhost:30080
   ↓
  Service:80
   ↓
  Pod/Container:80
  ```
---

## 15. Label과 Selector
- Kubernetes에서는 Label과 Selector를 사용해 Service와 Pod를 연결한다.
- Pod는 재생성될 때 이름과 IP가 변경될 수 있으므로, Service는 특정 Pod 이름이나 IP를 직접 기억하지 않는다.
- 대신 Pod에 Label을 붙이고, Service는 Selector를 통해 해당 Label을 가진 Pod를 찾는다.

### Pod Label
Deployment의 Pod 템플릿에 다음과 같이 Label을 지정한다.
```yaml
template:
  metadata:
    labels:
      app: nginx
```
이 설정으로 Deployment가 생성하는 Pod에는 `app: nginx` Label이 붙는다.

#### 확인
```bash
kubectl get pods --show-labels  # Pod의 Label 확인
kubectl get pods -l app=nginx   # 특정 Label을 가진 Pod만 조회
```

### Service Selector
Service에서는 다음과 같이 Selector를 지정한다.
```yaml
selector:
  app: nginx
```
이 설정으로 Service는 `app: nginx` Label을 가진 Pod를 찾는다.

#### 연결 구조
```text
Service selector: app=nginx 
    ↓ 
Pod label: app=nginx 
    ↓ 
nginx Container
```

#### Service Endpoint 확인
```bash
kubectl get endpoints nginx-service
```
- 정상적으로 연결된 경우 Pod IP와 포트가 표시된다.
- Endpoint에 Pod IP가 있다는 것은 Service가 Label이 일치하는 Pod를 정상적으로 찾았다는 의미

### port-forward로 Service 연결 확인
```bash
kubectl port-forward service/nginx-service 8080:80
```
- 내 pc 8080 -> Kubernetes Service 80 -> Pod 80 -> nginx Container
- `Ctrl+C`로 종료

### Selector 불일치
- Service Selector와 Pod Label이 일치하지 않을 경우, Service는 해당 Label을 가진 Pod를 찾지 못한다.
- 이 경우 Pod는 `Running` 상태여도 Service를 통한 요청은 실패할 수 있다.
---

## 16. Kubernetes Service 타입

Kubernetes Service는 Pod 집합을 네트워크에 노출하는 역할을 한다.

Pod는 재생성될 때 IP가 변경될 수 있지만, Service를 사용하면 클라이언트는 일정한 주소로 Pod에 접근할 수 있다.

### ClusterIP

ClusterIP는 Kubernetes 클러스터 내부에서만 접근할 수 있는 Service 타입이다.

`type`을 생략하면 기본적으로 ClusterIP가 사용된다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
- 클러스터 내부 Pod → nginx-service:80 → nginx Pod:80

### NodePort

NodePort는 각 Node의 특정 포트를 통해 Service를 외부에 공개한다.

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
- Node IP:30080 → Service:80 → Pod:80

### LoadBalancer
- LoadBalancer는 클라우드 제공자의 외부 로드밸런서를 사용해 Service를 공개한다.
- 인터넷 → Cloud Load Balancer → Kubernetes Service → Pod
- 실제 외부 로드밸런서 생성은 클라우드 환경의 지원 필요

### Ingress
- Ingress는 Service 타입이 아니라 별도의 Kubernetes 리소스이다.
- HTTP 요청의 도메인이나 경로에 따라 여러 Service로 요청을 전달할 수 있다.
- `/api/users` → `user-service`  `/api/orders` → `order-service`
- Ingress가 실제로 동작하려면 Ingress Controller가 필요하다.

### Service 타입 비교
- ClusterIP: 클러스터 내부에서만 접근 가능
- NodePort: Node IP를 통해 접근 가능
- LoadBalancer: 외부 로드밸런서를 통해 접근 가능
- Ingress: HTTP 요청의 도메인이나 경로에 따라 여러 Service로 요청을 전달할 수 있다.

### ClusterIP 변경 실습
- 기존 NodePort Service에서 `type`을 `ClusterIP`로 변경, `nodePort` 항목 삭제
- 적용 : `kubectl apply -f service.yaml`
- 확인 : `kubectl get services` `kubectl get endpoints nginx-service`

### port-forward
- ClusterIP Service는 외부 브라우저에서 직접 접근할 수 없으므로 개발 및 점검 목적으로 port-forward를 사용할 수 있다.
- 적용 : `kubectl port-forward service/nginx-service 8080:80`
- 내 PC:8080 → port-forward → Service:80 → Pod:80
- port-forward는 명령어가 실행되는 동안만 유지되는 임시 연결이다.

### 내부 접근
- Kubernetes 내부에서는 Service 이름을 사용해 접근할 수 있다.
- `kubectl run curl-test --image=curlimages/curl --rm -it -- sh`
- 임시 Pod 내부에서 확인 : `curl http://nginx-service`
- Kubernetes 내부 DNS가 `nginx-service`라는 이름을 Service 주소로 해석한다.
---
