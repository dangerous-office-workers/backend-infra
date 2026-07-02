# Volume

Kubernetes Volume을 사용하여 컨테이너에 데이터를 저장하거나 파일을 공유한다.

[← README](../README.md)

## 22. Kubernetes Volume과 emptyDir

Kubernetes 컨테이너 내부의 파일은 컨테이너나 Pod가 재생성될 때 사라질 수 있다.

Volume을 사용하면 컨테이너의 특정 경로를 별도 저장 공간에 연결할 수 있다.

### Volume 구성

Kubernetes Volume은 두 부분으로 구성한다.

```text
volumes → Pod에서 사용할 저장 공간 정의

volumeMounts → 컨테이너 내부에 Volume을 연결
```

### emptyDir 설정

```yaml
spec:
  containers:
    - name: busybox
      image: busybox:1.36
      command:
        - sh
        - -c
        - while true; do sleep 3600; done

      volumeMounts:
        - name: shared-data
          mountPath: /data

  volumes:
    - name: shared-data
      emptyDir: {}
```

`volumes.name`과 `volumeMounts.name`은 같아야 한다.

### 파일 생성

```bash
kubectl exec -it <pod-name> -- sh
```

- Pod 내부
```sh
echo "hello kubernetes volume" > /data/message.txt
cat /data/message.txt
```

### 컨테이너 재시작

- 같은 Pod 안에서 컨테이너만 재시작되면 `emptyDir` 데이터는 유지될 수 있다.
- 컨테이너 재시작 → 같은 Pod 유지 → emptyDir 유지

### Pod 재생성

Pod를 삭제하면 Deployment가 새 Pod를 생성한다.

```bash
kubectl delete pod <pod-name>
```

새 Pod의 `emptyDir`은 새로운 빈 저장 공간이다.

```text
기존 Pod 삭제 → 기존 emptyDir 삭제

새 Pod 생성 → 새로운 emptyDir 생성
```

### 다중 컨테이너 간 공유

같은 Pod의 여러 컨테이너가 동일한 Volume을 마운트하면 파일을 공유할 수 있다.

```text
Pod
├─ writer 컨테이너
├─ reader 컨테이너
└─ shared-data Volume
```

- 특정 컨테이너의 로그 확인

```bash
kubectl logs <pod-name> -c reader
```

- 특정 컨테이너에 접속

```bash
kubectl exec -it <pod-name> -c writer -- sh
```

### Docker와 비교

```text
Docker → -v 옵션으로 Volume 연결

Kubernetes
→ volumes로 저장 공간 정의
→ volumeMounts로 컨테이너 경로 연결
```

### emptyDir 사용 예

```text
임시 파일
캐시
배치 작업 중간 결과
같은 Pod의 컨테이너 간 파일 공유
```

### 주의점

`emptyDir`은 Pod가 삭제되면 데이터도 함께 삭제된다.

DB 파일이나 반드시 보존해야 할 데이터에는 PersistentVolume과 PersistentVolumeClaim 같은 영구 저장 방식을 사용해야 한다.
