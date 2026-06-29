# Configmap

애플리케이션 코드와 별도로 구성 데이터를 설정하기 위하여 Configmap 관련 내용을 정리한다.

[← README](../README.md)

---

## 20. ConfigMap 환경변수 주입

Configmap은 비밀번호가 아닌 일반 애플리케이션 설정을 Kubernetes 리소스로 분리해 관리할 때 사용한다.

애플리케이션 코드와 환경별 설정을 분리하면 설정이 달라질 때마다 Docker 이미지를 다시 만들 필요가 없다.

- `Docker Image` → 애플리케이션 코드
- `ConfigMap` → 환경별 설정

### 실습 파일 구조
```text
kubernetes-basic/ 
└─ configmap-basic/ 
    ├─ configmap.yaml 
    └─ deployment.yaml
```

### ConfigMap 생성

`configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: dev
  LOG_LEVEL: INFO
  API_TIMEOUT: "30"
```

ConfigMap의 값은 문자열로 다루므로 숫자처럼 보이는 값도 문자열로 작성할 수 있다.

```yaml
API_TIMEOUT: "30"
```

- 적용
```bash
kubectl apply -f configmap.yaml
```

- 확인
```bash
kubectl get configmaps
kubectl get cm
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

### 특정 ConfigMap 키 주입

`deployment.yaml`에서 `configMapKeyRef`를 사용하면 필요한 키만 환경변수로 주입할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configmap-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configmap-demo
  template:
    metadata:
      labels:
        app: configmap-demo
    spec:
      containers:
        - name: busybox
          image: busybox:1.36
          command:
            - sh
            - -c
            - while true; do sleep 3600; done

          env:
            - name: APP_MODE
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_MODE

            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: LOG_LEVEL
```

- 적용
```bash
kubectl apply -f deployment.yaml
```

- Pod 상태 확인
```bash
kubectl get pods
```

### CreateContainerConfigError 확인

처음 Deployment를 적용했을 때 다음 상태가 발생했다.

```text
READY   STATUS
0/1     CreateContainerConfigError
```

- 상세 원인 확인
```bash
kubectl describe pod <pod-name>
```

- 오류 확인
```text
Error: couldn't find key APP_MODE in ConfigMap default/app-config
```

ConfigMap 자체는 존재하지만 Deployment가 참조하는 `APP_MODE` 키가 ConfigMap 안에 없다는 뜻

오탈자 확인 필요.

### Pod 내부 환경변수 확인

- Pod 내부 셸에 접속
```bash
kubectl exec -it <pod-name> -- sh
```

- 환경변수 확인
```sh
echo $APP_MODE
echo $LOG_LEVEL
```

- 결과 확인
```text
dev
INFO
```

### ConfigMap 전체 주입

ConfigMap의 키를 하나씩 지정하지 않고 모든 값을 한 번에 주입하려면 `envFrom`을 사용할 수 있다.

기존 `env` 설정을 다음과 같이 변경한다.

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

- 변경사항 적용
```bash
kubectl apply -f deployment.yaml
```

- 새 Pod에서 환경변수 확인
```bash
kubectl exec -it <pod-name> -- sh
```

```sh
echo $APP_MODE
echo $LOG_LEVEL
echo $API_TIMEOUT
```

- 결과 확인
```text
dev
INFO
30
```

### env와 envFrom의 차이

#### env와 configMapKeyRef

```yaml
env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_MODE
```
- 필요한 키만 선택 가능
- 어떤 설정을 사용하는지 명확함
- 환경변수 이름을 다르게 지정할 수 있음

#### envFrom과 configMapRef

```yaml
envFrom:
  - configMapRef:
      name: app-config
```
- ConfigMap의 모든 키를 한 번에 주입
- YAML이 간단함
- 사용하지 않는 값도 함께 주입될 수 있음

### ConfigMap 값 변경

`configmap.yaml`의 값을 변경한다.

```yaml
APP_MODE: test
```

- ConfigMap 다시 적용
```bash
kubectl apply -f configmap.yaml
```

- ConfigMap 자체의 변경 여부 확인
```bash
kubectl get configmap app-config -o yaml
```

- 하지만 기존 Pod에서 환경변수를 확인하면 이전 값인 `dev`가 그대로 남아 있을 수 있다.
```bash
kubectl exec -it <pod-name> -- sh
```

```sh
echo $APP_MODE
```

```text
dev
```

환경변수로 주입한 ConfigMap 값은 컨테이너가 시작될 때 설정된다.

따라서 실행 중인 컨테이너의 환경변수는 ConfigMap이 변경되어도 자동으로 바뀌지 않는다.

### ConfigMap 변경값 반영

변경된 ConfigMap 값을 반영하기 위해 Deployment를 재시작한다.

```bash
kubectl rollout restart deployment/configmap-demo
```

- 롤아웃 상태 확인
```bash
kubectl rollout status deployment/configmap-demo
kubectl get pods
```

- 새로 생성된 Pod에서 값 확인
```bash
kubectl exec -it <new-pod-name> -- sh
```
```sh
echo $APP_MODE
```

- 전체 흐름
```text
ConfigMap 수정
→ ConfigMap 리소스 변경
→ 기존 Pod 환경변수는 유지
→ Deployment 재시작
→ 새 Pod 생성
→ 새 ConfigMap 값 주입
```

### Spring Boot 활용 예시

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-api-config
data:
  SPRING_PROFILES_ACTIVE: dev
  SERVER_PORT: "8080"
  LOGGING_LEVEL_ROOT: INFO
```

- Deployment.yaml
```yaml
envFrom:
  - configMapRef:
      name: order-api-config
```

`Spring Profile`, `로그 레벨`, `서버 포트`, `외부 API 주소`, `타임아웃`, `기능 활성화 여부` 와 같은 일반 설정을 ConfigMap으로 분리할 수 있다.

비밀번호, 인증키, 토큰 등의 민감 정보는 ConfigMap이 아니라 Secret을 사용해야 한다.

### 정리
```bash
kubectl delete -f deployment.yaml
kubectl delete -f configmap.yaml
```

- 확인
```bash
kubectl get deployments
kubectl get pods
kubectl get configmaps
```