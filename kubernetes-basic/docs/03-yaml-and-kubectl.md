# YAML과 kubectl 기본 명령어

Kubernetes 리소스를 YAML로 선언하고 kubectl로 상태를 확인하는 기본 흐름을 정리한다.

[← README](../README.md)

---

## 7. Kubernetes YAML
Kubernetes에서 Pod, Deployment, Service 같은 리소스를 YAML 파일로 정의할 수 있다.

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment  # Deployment를 만든다.
metadata:
  name: nginx-deployment
spec:
  replicas: 2   # Pod 2개 유지
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx  # Pod 안에서 nginx 컨테이너를 실행하라는 뜻
          ports:
            - containerPort: 80
```
- `replicas: 2` : Pod 2개 유지
  ```
  Deployment
   ↓
  Pod 2개
   ↓
  nginx Container
  ```

### Service
Service는 Pod 앞단의 대표 진입점 역할을 한다.
```yaml
apiVersion: v1
kind: Service   # service 설정
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx  # app: nginx 라벨을 가진 Pod를 찾아 연결
  ports:  # 외부 접근 포트 30080 -> Service 80 -> Pod 내부 80
    - port: 80
      targetPort: 80
      nodePort: 30080
```
- Service는 selector를 통해 연결할 Pod를 찾는다.
- 이 경우 app: nginx 라벨을 가진 Pod에 요청을 전달한다.
```
브라우저
 ↓
Service
 ↓
Pod
 ↓
Container
```
- Deployment는 Pod 개수를 유지하고, Service는 Pod로 요청을 전달한다.

### 실행 명령어
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 확인 명령어
```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

### 삭제 명령어
```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

### 정리
- Kubernetes 리소스는 YAML 파일로 정의할 수 있다.
- Deployment는 Pod를 생성하고 개수를 유지한다.
- Service는 Pod 앞단의 고정 진입점 역할을 한다.
- selector와 label을 통해 Service가 Pod를 찾는다.
---

## 8. Kubernetes 상태 확인 명령어
Kubernetes에서는 `kubectl get` 명령어로 리소스 상태를 확인할 수 있다.

### Node 확인
```bash
kubectl get nodes
```
```bash
NAME             STATUS   ROLES           AGE    VERSION
docker-desktop   Ready    control-plane   3d5h   v1.34.1
```
- STATUS가 Ready이면 로컬 Kubernetes 클러스터가 정상 동작 중이다.

### Deployment 확인
```bash
kubectl get deployments
```
```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           30s
```
- `READY 2/2` : 원하는 Pod 2개 중 2개가 정상 준비되었다.

### Pod 확인
```bash
kubectl get pods
```
```bash
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7ccccd94f7-9zdp8   1/1     Running   0          3m8s
nginx-deployment-7ccccd94f7-p5dts   1/1     Running   0          3m8s
```
- `READY` : Pod 안의 컨테이너 준비 상태
- `STATUS` : Pod 실행 상태
- `RESTARTS` : 재시작 횟수 (RESTARTS가 계속 증가하면 애플리케이션이 반복적으로 죽고 다시 뜨는 상황일 수 있다.)

### Service 확인
```bash
kubectl get services
```
```bash
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        3d6h
nginx-service   NodePort    10.99.201.218   <none>        80:30080/TCP   6m15s
```
- `80:30080/TCP` : Service의 80번 포트가 외부 30080번포트와 연결됨

### 전체 리소스 확인
```bash
kubectl get all
```
```bash
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7ccccd94f7-9zdp8   1/1     Running   0          7m13s
pod/nginx-deployment-7ccccd94f7-p5dts   1/1     Running   0          7m13s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        3d6h
service/nginx-service   NodePort    10.99.201.218   <none>        80:30080/TCP   6m56s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           7m13s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7ccccd94f7   2         2         2       7m13s
```
- Pod, Service, Deployment, ReplicaSet 등을 한 번에 확인할 수 있다.

### 로그 확인
```bash
kubectl logs <pod-name>
```
- 실무에서는 애플리케이션 실행 오류, DB 연결 실패, 환경변수 문제 등을 확인할 때 사용한다.

### 삭제
```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

### 정리
- Kubernetes에서는 `kubectl get`으로 리소스 상태를 확인한다.
- Deployment는 원하는 Pod 개수가 준비되었는지 확인한다.
- Pod는 STATUS, READY, RESTARTS를 중요하게 본다.
- Service는 Pod로 접근하기 위한 진입점과 포트 정보를 제공한다.
- `kubectl logs`는 컨테이너 로그를 확인하는 데 사용한다.
---

## 9. kubectl describe
- `kubectl describe`는 Kubernetes 리소스의 상세 정보를 확인하는 명령어이다.
- Pod 상태를 한 줄로 확인할 때는 `kubectl get pods`를 사용하고, Pod의 상세 원인이나 이벤트를 확인할 때는 `kubectl describe pod`를 사용한다.

### Pod 상세 확인
```bash
kubectl describe pod <pod-name>
```

### get / logs / describe 차이
- `kubectl get pods` : Pod 상태 요약 확인
- `kubectl logs <pod-name>` : 애플리케이션 로그 확인
- `kubectl describe pod <pod-name>` : Pod 상세 정보와 Kubernetes 이벤트 확인

### describe에서 볼 부분
- `Status`
  - Pod 현재 상태
- `Containers`
  - 컨테이너 이름, 이미지, 포트 정보
- `Environment`
  - 환경변수 설정 정보
- `Events`
  - Kubernetes가 기록한 이벤트

### 자주 보는 상태
- `Pending`
  - Pod가 아직 실행될 준비가 되지 않은 상태
  - 가능한 원인 : 리소스 부족, 노드 문제, 이미지 다운로드 대기
- `ImagePullBackOff`
  - 이미지를 가져오지 못한 상태
  - 가능한 원인 : 이미지 이름 오타, 이미지 태그 없음, private registry 인증 실패
- `CrashLoopBackOff`
  - 컨테이너가 실행됐다가 계속 죽고 다시 시작되는 상태
  - 가능한 원인 : 애플리케이션 실행 실패, 환경변수 누락, DB 연결 실패, 포트 설정 오류

### 장애 확인 흐름
```text
1. kubectl get pods
   → Pod 상태 요약 확인

2. kubectl logs <pod-name>
   → 애플리케이션 로그 확인

3. kubectl describe pod <pod-name>
   → Kubernetes 이벤트와 상세 설정 확인
```

### 정리
- `kubectl get` 상태 요약을 확인
- `kubectl logs` 애플리케이션 로그 확인
- `kubectl describe` Kubernetes 리소스의 상세 정보와 이벤트 확인
- 백엔드 애플리케이션 장애 확인 시 `get -> logs -> describe` 흐름이 중요
---
