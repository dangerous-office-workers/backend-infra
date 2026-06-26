# Pod, Deployment, Kubernetes 구조

Kubernetes의 기본 실행 단위인 Pod와 이를 관리하는 Deployment의 관계를 정리한다.

[← README](../README.md)

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
