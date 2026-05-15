# docker-basic
Docker의 기본 개념과 명령어를 연습하는 디렉토리

## TIL
- Docker는 실행 환경까지 포장하기 위해 사용한다.
- Image는 실행 설계도이다.
- Container는 Image를 실행한 상태이다.
- `docker run nginx` : nginx 이미지 실행 후 컨테이너 생성
- `docker images` : 현재 PC에 저장된 Docker 이미지 목록을 보여줌
- `docker ps` : 실행 중인 컨테이너 확인
- `docker ps -a` : 종료된 컨테이너까지 확인
- `docker logs 컨테이너ID` : 컨테이너 로그 확인


## Dockerfile
```dockerfile
FROM python:3.11

CMD ["python", "--version"]
```
- `FROM` : 기반 이미지 지정
- `CMD` : 컨테이너 실행 시 수행할 명령어

### Build
```bash
docker build -t python-version-test .
```

### Run
```bash
docker run python-version-test
```