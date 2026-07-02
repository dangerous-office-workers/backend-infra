# kubernetes-basic

Docker로 컨테이너를 실행하는 단계에서 출발해 Kubernetes의 기본 리소스와 배포 흐름을 실습하는 디렉토리입니다.

## 학습 목표

- Docker와 Kubernetes의 역할 차이 이해
- Pod, Deployment, Service의 기본 구조 이해
- YAML로 Kubernetes 리소스 선언 및 적용
- kubectl로 상태 확인, 로그 확인, 장애 원인 추적
- ConfigMap, Secret, Probe, Resource 설정 실습
- Rolling Update, Rollback, Namespace 기본 흐름 이해

## 학습 순서

1. [왜 Kubernetes가 필요한가](./docs/01-why-kubernetes.md)
2. [Pod, Deployment, Kubernetes 구조](./docs/02-core-resources.md)
3. [YAML과 kubectl 기본 명령어](./docs/03-yaml-and-kubectl.md)
4. [ConfigMap, Secret, Probe, Resource](./docs/04-config-and-healthcheck.md)
5. [Service, Label, Selector, 네트워킹](./docs/05-service-networking.md)
6. [Rolling Update와 Rollback](./docs/06-rollout-and-rollback.md)
7. [Namespace](./docs/07-namespace.md)
8. [Configmap](./docs/08-configmap.md)
9. [secret](./docs/09-secret.md)
10. [volume](./docs/10-volume.md)

## 실습 환경

- Docker Desktop Kubernetes
- kubectl
- nginx 예제 리소스

## 디렉토리 구조

```text
kubernetes-basic/
├─ first-yaml/
│    ├─ deployment.yaml
│    └─ service.yaml
├─ configmap-basic/
│    ├─ configmap.yaml
│    └─ deployment.yaml
├─ secret-basic/
│    ├─ deployment.yaml
│    └─ secret.yaml
├─ volume-basic/
│    ├─ deployment.yaml
├─ README.md
└─ docs/
   ├─ 01-why-kubernetes.md
   ├─ 02-core-resources.md
   ├─ 03-yaml-and-kubectl.md
   ├─ 04-config-and-healthcheck.md
   ├─ 05-service-networking.md
   ├─ 06-rollout-and-rollback.md
   ├─ 07-namespace.md
   ├─ 08-configmap.md
   ├─ 09-secret.md
   └─ 10-volume.md
```
