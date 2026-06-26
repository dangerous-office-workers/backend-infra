# ConfigMap, Secret, Probe, Resource

애플리케이션 설정 주입, 상태 검사, 리소스 요청과 제한을 실습 관점에서 정리한다.

[← README](../README.md)

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

## 11. Readiness Probe와 Liveness Probe
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

## 12. Resource Requests와 Limits
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
