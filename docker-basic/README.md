# docker-basic
Docker의 기본 개념과 명령어를 연습하는 디렉토리

## TIL
- Docker는 실행 환경까지 포장하기 위해 사용한다.
- Image는 실행 설계도이다.
- Container는 Image를 실행한 상태이다.
- `docker run nginx` : nginx 이미지 실행 후 컨테이너 생성
- `docker start 컨테이너ID` : 이미 존재하는 컨테이너 다시 실행
- `docker images` : 현재 PC에 저장된 Docker 이미지 목록을 보여줌
- `docker ps` : 실행 중인 컨테이너 확인
- `docker ps -a` : 종료된 컨테이너까지 확인
- `docker logs 컨테이너ID` : 컨테이너 로그 확인
- `docker stop 컨테이너ID` : 컨테이너 종료
- `docker rm 컨테이너ID` : 컨테이너 삭제
- `docker rmi 이미지명` : 이미지 삭제

## 1. Python 버전 출력 컨테이너
### Dockerfile
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
---

## 2. nginx 컨테이너 실행
### nginx 실행
```bash
docker run -d -p 8080:80 nginx
```
- `docker run` : 컨테이너 실행
- `-d` : 백그라운드 실행 (detached) (터미널 안 잡고 계속 실행)
- `-p 8080:80` : 내 pc 8080 포트 -> 컨테이너 내부 80 포트 연결
- 브라우저 접속 : http://localhost:8080
---

## 3. 컨테이너 내부 접속
```bash
docker exec -it 컨테이너ID
```
- `docker exec` : 실행 중인 컨테이너 안에서 명령 실행
- `-it` : 컨테이너 안에서 직접 명령 입력 가능하게 만드는 옵션
- `bash` : 컨테이너 내부 쉘 실행(리눅스 터미널 들어가는 느낌)
---

## 4. 환경변수 전달(`-e`)
```bash
docker run --rm -e MY_NAME=bblackbean python:3.11 env
```
- `-e` : 환경변수 설정
- `MY_NAME=bblackbean` : 이 값을 컨테이너 안에 전달
- `env` : 현재 환경변수 출력 명령
- `--rm` : 컨테이너 종료 후 자동 삭제
---

## 5. Volume 연결
### volume-test 폴더 생성
```bash
mkdir volume-test
echo "hello docker volume" > volume-test/test.txt
```

### nginx 컨테이너 실행
```bash
# docker run [옵션들] 이미지명
docker run -d -p 8080:80 -v $(pwd)/volume-test:/usr/share/nginx/html nginx
```
- `-v` : 볼륨(volume) 연결
- `$(pwd)/volume-test` : 호스트(OS) 쪽 실제 폴더(현재 폴더)
- `usr/share/nginx/html` : 컨테이너 내부 nginx 기본 웹 폴더
- 내 로컬 폴더 내용을 컨테이너 내부 웹 폴더에 연결

### 정리
- 확인 : 브라우저 접속 (http://localhost:8080/test.txt)
- 호스트 폴더와 컨테이너 폴더 연결 가능
- 컨테이너가 삭제되어도 파일 유지 가능
- 업로드 파일 / DB 데이터 / 로그 저장 등에 사용