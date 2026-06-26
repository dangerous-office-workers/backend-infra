# kubernetes-basic
### Kubernetes 기초 학습 디렉토리
- Kubernetes가 왜 등장했는지 이해
- Pod, Deployment, Service 개념 이해
- Docker와 Kubernetes의 차이 이해
---

## 1. 왜 Kubernetes가 필요한가?
Docker는 컨테이너를 실행하는 도구
```bash
docker run nginx
```
- Docker는 컨테이너 실행은 잘하지만, 컨테이너가 많아지면 관리가 어려워진다.
- 예를 들어 Spring Boot, Redis, MySQL을 여러 대 서버에서 운영한다고 가정하면,
    - 컨테이너가 죽었는지
    - 몇 개가 실행 중인지
    - 다시 실행해야 하는지
    등을 직접 관리해야 한다. Kubernetes는 이러한 컨테이너 관리를 자동화하기 위해 등장했다.
---

## 2. Docker와 Kubernetes
### Docker
```text
Container
```
- Docker는 컨테이너를 실행한다.

### Kubernetes
```text
Pod
 └─ Container
```
- Kubernetes는 Pod를 실행한다.
---

## 3. Pod
Pod는 Kubernetes에서 가장 작은 실행 단위 (컨테이너를 담는 상자)
```text
Pod
 └─ FastAPI Container
```
```text
Pod
 └─ Spring Boot Container
```
- 대부분의 경우 Pod 1개, Container 1개 형태로 사용한다.
---

## 4. Deployment
Pod를 관리하는 객체(Pod 관리자)
```text
Deployment
 ↓
Pod 3개
```
Deployment는 다음 역할을 수행한다.
- Pod 생성
- Pod 개수 유지
- Pod 재생성
- 배포 관리
```text
Pod 1개 종료
 ↓
Deployment 감지
 ↓
새 Pod 생성
```
---

## 5. Kubernetes 구조
```text
Deployment
    ↓
Pod
    ↓
Container
```
Docker에서는 Container가 실행 단위였지만, Kubernetes에서는 Pod가 실행 단위이며, Deployment가 Pod를 관리한다.

### 정리
- Docker는 컨테이너를 실행한다.
- Kubernetes는 Pod를 실행한다.
- Pod는 컨테이너를 담는 실행 단위이다.
- Deployment는 Pod를 관리하는 관리자 역할을 한다.
- 실무에서는 Pod보다 Deployment를 더 자주 다룬다.
---

## 6. Service
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

## 10. ConfigMap과 Secret
Kubernetes에서는 애플리케이션 설정값을 ConfigMap과 Secret으로 관리할 수 있다.
Docker에서는 환경변수를 다음과 같이 직접 전달했다.
```bash
docker run -e APP_MODE=dev my-app
```
- Kubernetes에서는 ConfigMap이나 Secret에 값을 저장하고, Deployment를 통해 Pod 내부 컨테이너에 주입할 수 있다.
```text
ConfigMap / Secret
 ↓
Deployment
 ↓
Pod
 ↓
Container 환경변수
 ↓
Spring Boot / FastAPI
```

### ConfigMap
```yaml
APP_MODE=dev
LOG_LEVEL=info
REDIS_HOST=redis-service
SPRING_PROFILES_ACTIVE=prod
```
- 노출되어도 큰 문제가 없는 설정값을 넣는다.

### Secret
민감한 설정값을 저장하는 Kubernetes 객체
```yaml
DB_PASSWORD
JWT_SECRET
API_KEY
ACCESS_TOKEN
```
- 비밀번호, 토큰, 키 같은 값은 ConfigMap이 아니라 Secret으로 관리하는 것이 일반적이다.

### Docker와 Kubernetes 비교
- Docker
  ```bash
  docker run -e SPRING_PROFILES_ACTIVE=prod my-spring-app
  ```
- Kubernetes
  - ConfigMap

### 장애 확인 흐름
```text
kubectl get pods
 ↓
kubectl logs <pod-name>
 ↓
kubectl describe pod <pod-name>
```
- `describe`에서 환경변수와 이벤트를 확인할 수 있다.
---

## 11. Kubernetes 포트 연결
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

## 12. Label과 Selector
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

## 13. Readiness Probe와 Liveness Probe
- Kubernetes에서는 Probe를 사용해 컨테이너의 상태를 주기적으로 확인할 수 있다.
- Pod가 `Running` 상태라고 해서 애플리케이션이 사용자 요청을 정상적으로 처리할 준비가 된 것은 아니다.

### Readiness Probe
- Readiness Probe는 애플리케이션이 요청을 받을 준비가 되었는지 확인한다.
- 검사에 실패하면 Pod는 계속 실행되지만 Service의 요청 대상에서 제외된다.
```text
Readiness 성공 → Service가 요청 전달
Readiness 실패 → Service가 해당 Pod로 요청을 보내지 않음
```

### Liveness Probe
- Liveness Probe는 애플리케이션이 정상적으로 살아 있는지 확인한다.
- 설정된 횟수만큼 계속 실패하면 Kubernetes는 Pod를 재시작할 수 있다.
```text
Liveness 성공 → Pod 계속 실행
Liveness 실패 → Pod 재시작
```

### 차이
```text
Readiness → 트래픽을 보내도 되는가?

Liveness → 컨테이너를 재시작해야 하는가?
```

### nginx Probe 설정
```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
```
- `httpGet` : 지정한 HTTP 경로로 상태를 확인한다.
- `initialDelaySeconds` : 컨테이너 실행 후 검사를 시작하기 전 대기 시간
- `periodSeconds` : Probe 검사 주기

### Readiness 실패 실습
Readiness Probe 경로를 존재하지 않는 경로로 변경
```yaml
readinessProbe:
  httpGet:
    path: /wrong-health
    port: 80
```
- 확인
```bash
kubectl get pods
kubectl get endpoints nginx-service
kubectl describe pod <pod-name>
```
- 이때 Pod는 `Running` 상태지만 `READY`는 `0/1`이 될 수 있다.
- Service Endpoint에서도 준비되지 않은 Pod가 제외된다.
---

## 14. Resource Requests와 Limits
- Kubernetes에서는 여러 Pod가 Node의 CPU와 메모리를 함께 사용한다.
- 특정 컨테이너가 자원을 과도하게 사용하면 다른 애플리케이션에도 영향을 줄 수 있으므로 `requests`와 `limits`를 설정할 수 있다.

### Requests
- `requests`는 컨테이너 실행에 필요한 최소 자원을 의미한다.
- Kubernetes 스케쥴러는 이 값을 참고해 Pod를 어느 Node에 배치할지 결정한다.
```yaml
resources:
  requests:   # 자원 자리 예약
    cpu: "100m"
    memory: "64Mi"
```

### Limits
`limits`는 컨테이너가 사용할 수 있는 최대 자원을 의미한다.
```yaml
resources:
  limits:   # 자원 사용 상한선
    cpu: "200m"
    memory: "128Mi"
```

### CPU 단위
```text
1000m = CPU 1코어
500m  = CPU 0.5코어
100m  = CPU 0.1코어
```

### 메모리 단위
```text
128Mi
256Mi
512Mi
1Gi
```
- 대략 1024Mi = 1Gi

### 설정 확인
```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl describe pod <pod-name>
```
- `describe` 결과에서 `Requests`와 `Limits`를 확인할 수 있다.

### 장애 관점
#### Requests가 너무 큰 경우
- Node에 필요한 자원이 부족하면 Pod가 배치되지 못하고 `Pending` 상태가 될 수 있다.

#### Memory limit을 초과한 경우
- 컨테이너가 메모리 제한을 초과하면 `OOMKilled` 상태로 종료될 수 있다.

#### CPU limit이 너무 작은 경우
- 컨테이너가 종료되지 않더라고 애플리케이션 처리 속도가 느려지거나 타임아웃이 발생할 수 있다.

### 정리
```text
Pod Pending → requests와 Node 자원 확인

Pod 반복 재시작 → OOMKilled와 memory limit 확인

API 응답 지연 → CPU limit 확인
```
---

## 15. Deployment 롤링 업데이트와 롤백

Deployment의 Pod 템플릿이 변경되면 Kubernetes는 새로운 Pod를 생성하고 기존 Pod를 순차적으로 교체한다.

이 방식을 Rolling Update라고 한다.

### Rolling Update
```text
기존 Pod 실행
    ↓
새 버전 Pod 생성
    ↓
새 Pod 정상 여부 확인
    ↓
기존 Pod 종료
    ↓
순차적으로 교체
```
기존 Pod를 모두 한꺼번에 종료하지 않기 때문에 배포 중 서비스 중단 가능성을 줄일 수 있다.

### 이미지 확인
```bash
# 현재 이미지 확인
kubectl get deployment nginx-deployment -o jsonpath="{.spec.template.spec.containers[0].image}"

# Pod별 이미지 확인
kubectl get pods -o custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image"

# 이미지 변경
kubectl set image deployment/nginx-deployment nginx=nginx:1.28-alpine
# kubectl set image <Deployment 이름> <컨테이너 이름=새 이미지>
```

### 롤아웃 상태 확인
```bash
kubectl rollout status deployment/nginx-deployment
```

Pod 교체 과정을 실시간으로 확인하려면 다음 명령어를 사용한다.
```bash
kubectl get pods -w
```

### 롤아웃 이력 확인
```bash
kubectl rollout history deployment/nginx-deployment

# 특정 revision 확인
kubectl rollout history deployment/nginx-deployment --revision=2
```

### 잘못된 이미지 배포 실습
```bash
# 존재하지 않는 이미지 태그 적용
kubectl set image deployment/nginx-deployment nginx=nginx:not-exist-version

# Pod 상태 확인
kubectl get pods
```

새 Pod에서 `ErrImagePull`, `ImagePullBackOff` 상태를 확인할 수 있다.

```bash
# 상세 원인 확인
kubectl describe pod <pod 이름>
```

`Events` 영역에서 이미지 다운로드 실패 원인을 확인할 수 있다.

### 이전 버전으로 롤백
```bash
kubectl rollout undo deployment/nginx-deployment

# 롤백 상태 확인
kubectl rollout status deployment/nginx-deployment
kubectl get pods
```

특정 revision으로 롤백하려면 다음과 같이 실행할 수 있다.

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

### 선언 파일과 실제 상태

`kubectl set image`로 Deployment를 변경하면 실제 클러스터 상태는 바뀌지만, 로컬 `deployment.yaml`은 자동으로 수정되지 않는다.

이 상태에서 다시 `kubectl apply -f deployment.yaml`을 실행하면 YAML에 적힌 이미지 버전이 적용된다.

따라서 실무에서는 클러스터를 직접 변경한 경우 선언 파일에도 변경 내용을 반영해야 한다.


### 장애 대응 흐름
```text
새 버전 배포
    ↓
롤아웃 상태 확인
    ↓
Pod 상태 확인
    ↓
describe와 logs로 원인 확인
    ↓
복구가 급하면 이전 revision으로 롤백
```

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get nods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
```
---

## 16. RollingUpdate 상세 설정

Deployment의 RollingUpdate 전략은 `maxUnavailable`과 `maxSurge`를 사용해 Pod 교체 방식을 제어할 수 있다.

### 설정 예시
```yaml
spec:
  replicas: 2

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
```

### maxUnavailable

업데이트 중 사용할 수 없어도 되는 최대 Pod 수

```yaml
maxUnavailable: 0
```

이 설정은 기존에 준비된 Pod 수가 업데이트 과정에서 줄어드는 것을 허용하지 않는다.

### maxSurge

업데이트 중 기존 replica 수보다 추가로 생성할 수 있는 최대 Pod 수

```yaml
maxSurge: 1
```

replica가 2개라면 업데이트 중 일시적으로 총 3개의 Pod가 존재할 수 있다.

### 업데이트 흐름
```text
기존 Pod 2개
    ↓
새 Pod 1개 추가 생성
    ↓
총 Pod 3개
    ↓
새 Pod Ready
    ↓
기존 Pod 1개 종료
    ↓
순차 교체
    ↓
새 버전 Pod 2개
```

### 설정 확인
```bash
kubectl describe deployment nginx-deployment
```
- 확인 항목: `StrategyType`, `RollingUpdateStrategy`, `max unavailable`, `max surge`

### 롤링 업데이트 관찰
```bash
kubectl get pods -w
```

다른 터미널에서 이미지 버전을 변경한다.

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.27-alpine
```

롤아웃 상태 확인

```bash
kubectl rollout status deployment/nginx-deployment
```

### ReplicaSet 확인

```bash
kubectl get rs
```

Deployment는 롤링 업데이트 시 새로운 ReplicaSet을 생성하고, 이전 ReplicaSet의 Pod 수를 줄이면서 새 ReplicaSet의 Pod 수를 늘린다.

```text
Deployment
 ├─ 이전 ReplicaSet
 │    └─ 이전 버전 Pod
 └─ 새 ReplicaSet
      └─ 새 버전 Pod
```

### 설정 비교
```yaml
maxUnavailable: 0
maxSurge: 1
```
- 기존 가용 Pod 수 유지
- 새 Pod 먼저 추가
- 임시 추가 자원 필요

```yaml
maxUnavailable: 1
maxSurge: 0
```
- 기존 Pod를 먼저 줄일 수 있음
- 추가 Pod를 만들지 않음
- 업데이트 중 가용 Pod 감소 가능

### Readiness Probe와의 관계
새 Pod가 생성됐다고 바로 트래픽을 받을 수 있는 것은 아니다.

Readiness Probe가 성공해 새 Pod가 Ready 상태가 된 뒤에야 안전하게 기존 Pod를 교체할 수 있다.

---

## 17. Kubernetes Service 타입

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
- Kubernetes 내부 DNS가 `nginx-service`라는 이름ㅇ르 Service 주소로 해석한다.
---

## 18. Kubernetes Namespace

Namespace는 하나의 Kubernetes 클러스터 안에서 리소스를 논리적으로 구분하는 공간이다.

환경이나 팀별로 Deployment, Pod, Service 등의 리소스를 나어 관리할 수 있다.

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