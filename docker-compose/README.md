# docker-compose
Docker Compose 기초 실습 디렉토리.

# 1. nginx Compose 실행
## compose.yaml
```yaml
services: # 실행할 컨테이너 목록
  nginx:  # 서비스 이름
    image: nginx  # nginx 이미지 사용
    ports:  # 포트 연결
      - "8080:80"
```

### 실행
```bash
docker compose up -d
```
`compose.yaml` 파일 있는 디렉토리에서 실행

### 확인
```bash
docker ps
```
브라우저에서 확인 `http://localhost:8080`

### 종료
```bash
docker compose down
```

### 정리
- Docker Compose는 컨테이너 실행 설정을 파일로 관리하는 방식이다.
- `docker run -d -p 8080:80 nginx` 명령을 `compose.yaml`로 표현할 수 있다.
- services 아래에 실행할 컨테이너들을 정의한다.
- `docker compose up -d`는 설정 파일을 기준으로 컨테이너를 실행한다.
- `docker compose down`는 Compose로 실행한 컨테이너를 정리한다.