# Rolling Update와 Rollback

Deployment의 롤링 업데이트, 실패 배포 확인, 롤백 흐름과 상세 설정을 정리한다.

[← README](../README.md)

---

## 17. Deployment 롤링 업데이트와 롤백

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
kubectl get nodes
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
```
---

## 18. RollingUpdate 상세 설정

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
