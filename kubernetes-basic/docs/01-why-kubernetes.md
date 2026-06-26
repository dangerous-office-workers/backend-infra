# 왜 Kubernetes가 필요한가

Docker에서 Kubernetes로 넘어가는 이유와 두 도구의 역할 차이를 정리한다.

[← README](../README.md)

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
