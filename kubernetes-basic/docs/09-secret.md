# Secret

Kubernetes Secret을 생성하고 Pod에 환경변수로 주입하는 내용을 정리한다.

[← README](../README.md)

## 21. Secret 생성과 환경변수 주입

Kubernetes Secret은 비밀번호, 토큰, 인증키처럼 민감한 값을 애플리케이션 코드와 분리해 관리할 때 사용하는 리소스이다.

ConfigMap과 사용 방식은 비슷하지만, 목적이 다르다.

```text
ConfigMap
→ 일반 설정

Secret
→ 민감 정보
```

### 실습 파일 구조

```text
kubernetes-basic/
└─ secret-basic/
   ├─ secret.yaml
   └─ deployment.yaml
```

### Secret 생성

`secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  DB_USERNAME: study-user
  DB_PASSWORD: study-password-1234
```

`stringData`를 사용하면 평문 문자열을 작성할 수 있다.

- 적용

```bash
kubectl apply -f secret.yaml
```

- 확인

```bash
kubectl get secrets
kubectl describe secret db-secret
kubectl get secret db-secret -o yaml
```

`kubectl describe secret`에서는 실제 값이 보이지 않고 키와 크기만 확인할 수 있다.

### Base64 확인

Secret을 YAML로 조회하면 값이 Base64 형태로 보일 수 있다.

```yaml
data:
  DB_USERNAME: c3R1ZHktdXNlcg==
  DB_PASSWORD: c3R1ZHktcGFzc3dvcmQtMTIzNA==
```

Base64는 암호화가 아니라 인코딩이다. 쉽게 원래 값으로 복원할 수 있으므로 실제 비밀번호를 Base64로 바꿨다고 해서 Git에 올려도 안전한 것은 아니다.

### Deployment에서 Secret 참조

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secret-demo
  template:
    metadata:
      labels:
        app: secret-demo
    spec:
      containers:
        - name: busybox
          image: busybox:1.36
          command:
            - sh
            - -c
            - while true; do sleep 3600; done
          env:
            - name: DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_USERNAME
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_PASSWORD
```

- 적용

```bash
kubectl apply -f deployment.yaml
kubectl get pods
```

### Pod 내부 환경변수 확인

```bash
kubectl exec -it <pod-name> -- sh
```

- Pod 내부

```sh
echo $DB_USERNAME
echo $DB_PASSWORD
```

- 결과

```text
study-user
study-password-1234
```

Secret은 Kubernetes 리소스에서 관리되지만, 컨테이너 안에서는 애플리케이션이 사용할 수 있는 평문 환경변수로 전달된다.

### Secret 전체 주입

Secret의 모든 키를 한 번에 주입하려면 `envFrom`을 사용할 수 있다.

```yaml
envFrom:
  - secretRef:
      name: db-secret
```

#### secretKeyRef

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

- 특징
  - 필요한 키만 선택 가능
  - 어떤 Secret 값을 사용하는지 명확함
  - 환경변수 이름을 다르게 지정할 수 있음

#### secretRef

```yaml
envFrom:
  - secretRef:
      name: db-secret
```

특징:

```text
Secret의 모든 키를 한 번에 주입
YAML이 간단함
사용하지 않는 값도 함께 들어갈 수 있음
```

### Secret 값 변경

`secret.yaml`의 값을 변경한 뒤 다시 적용한다.

```yaml
stringData:
  DB_USERNAME: study-user
  DB_PASSWORD: changed-password-5678
```

```bash
kubectl apply -f secret.yaml
```

환경변수로 주입한 Secret 값은 실행 중인 기존 Pod에 자동 반영되지 않는다.

```text
Secret 변경 ≠ 기존 Pod 환경변수 자동 변경
```

새 값을 반영하려면 Deployment를 재시작한다.

```bash
kubectl rollout restart deployment/secret-demo
kubectl rollout status deployment/secret-demo
```

### 장애 확인

Secret 이름이나 키가 틀리면 Pod가 `CreateContainerConfigError` 상태가 될 수 있다.

- 상세 원인 확인

```bash
kubectl describe pod <pod-name>
```

- 확인할 항목

```text
Secret이 존재하는가?
Deployment와 같은 Namespace에 있는가?
Secret 이름이 정확한가?
참조하는 key가 존재하는가?
key의 대소문자가 일치하는가?
```

### Spring Boot 활용 예

```yaml
env:
  - name: SPRING_DATASOURCE_USERNAME
    valueFrom:
      secretKeyRef:
        name: order-db-secret
        key: DB_USERNAME
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: order-db-secret
        key: DB_PASSWORD
```

### Git 저장소에 올릴 때 주의할 점

실제 비밀번호나 운영 계정 정보가 들어 있는 Secret YAML은 공개 저장소에 커밋하면 안 된다.

Base64로 작성한 경우도 안전하지 않다. Base64는 쉽게 디코딩할 수 있기 때문이다.

학습용 공개 저장소에는 가짜 값이나 예시 값만 사용한다.

```yaml
stringData:
  DB_USERNAME: study-user
  DB_PASSWORD: study-password-1234
```

실제 업무에서는 CI/CD Secret 변수, Cloud Secret Manager, Vault, Sealed Secrets, External Secrets 같은 별도 관리 방식을 사용할 수 있다.

### 정리

```bash
kubectl delete -f deployment.yaml
kubectl delete -f secret.yaml
```

### 확인

```bash
kubectl get deployments
kubectl get pods
kubectl get secrets
```