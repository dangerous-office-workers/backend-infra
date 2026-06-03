# docker-fastapi
FastAPI 애플리케이션을 Docker 이미지로 만들고 컨테이너로 실행한다.

## 1. 디렉토리 구조
```text
docker-fastapi
 ├─ main.py
 ├─ requirements.txt
 └─ Dockerfile
```
---

## 2. FastAPI 애플리케이션
## main.py
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello():
    return {"message": "hello docker"}
```
- FastAPI 서버의 시작 코드
- `/` 경로로 요청이 들어오면 아래 JSON을 응답
```json
{
  "message": "hello docker"
}
```
---

## 3. 의존성 파일
### requirements.txt
```text
fastapi
uvicorn
```
- `fastapi`: API 서버를 만들기 위한 프레임워크
- `uvicorn`: FastAPI 애플리케이션을 실행하는 ASGI 서버 
- FastAPI 코드는 `uvicorn` 같은 서버 프로그램을 통해 실행된다.
---

## 4. Dockerfile
```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
- `FROM python:3.11` : 이 이미지는 python 3.11 실행 환경을 기반으로 사용한다. docker 이미지는 보통 완전히 처음부터 만드는 것이 아니라, 기존 이미지를 기반으로 만든다.
- `WORKDIR /app` : 컨테이너 내부의 작업 디렉토리를 `/app`으로 지정한다. 이후 실행되는 명령어들은 기본적으로 `/app` 위치를 기준으로 동작한다.
  - `COPY requirements.txt .` 이 명령은 실제로 컨테이너 내부의 아래 위치로 복사된다. `/app/requirements.txt`
  - 즉 `WORKDIR`은 컨테이너 안에서 작업할 기본 폴더를 정하는 역할
- `COPY requirements.txt .` : 현재 로컬 폴더에 있는 `requirements.txt` 파일을 컨테이너 내부 현재 위치로 옮긴다.
- `RUN pip install -r requirements.txt` : `requirements.txt`에 적힌 파이썬 패키지를 설치한다.
  - `RUN`은 컨테이너 실행 시점이 아니라 이미지 빌드 시점에 실행됨
- `COPY . .` : 현재 로컬 폴더의 나머지 파일들을 컨테이너 내부 현재 위치로 복사한다.
- `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]` : 컨테이너가 시작될 때 실행할 기본 명령어
  - `uvicorn` : FastAPI 앱을 실행하는 서버 프로그램
  - `main:app` : `main.app` 파일 안에 있는 `app` 객체 실행
  - `--host 0.0.0.0` : 컨테이너 외부에서도 접근 가능하도록 모든 네트워크 인터페이스에서 요청을 받겠다는 의미.
    Docker 컨테이너에서는 `127.0.0.1`만 사용하면 외부에서 접근이 안 될 수 있으므로 `0.0.0.0`을 사용
  - `--port 8000` : 컨테이너 내부에서 FastAPI 서버를 8000번 포트로 실행한다.
---

## 5. 이미지 빌드
```bash
docker build -t my-fastapi .
```
- `docker build`: Dockerfile을 기준으로 이미지를 만든다.
- `-t my-fastapi`: 이미지 이름을 `my-fastapi`로 지정한다.
- `.`: 현재 폴더를 빌드 기준 경로로 사용한다.
---

## 6. 컨테이너 실행
```bash
docker run -d -p 8000:8000 my-fastapi
```
- `-d`: 백그라운드 실행
- `-p 8000:8000`: 내 PC 8000번 포트를 컨테이너 내부 8000번 포트와 연결
- `my-fastapi`: 실행할 이미지 이름
- localhost:8000 에서 확인
---

## 7. 정리
- Dockerfile은 애플리케이션 실행 환경을 만드는 설명서이다.
- `FROM`은 기반 이미지를 지정한다.
- `WORKDIR`은 컨테이너 내부 작업 폴더를 지정한다.
- `COPY`는 로컬 파일을 이미지 안으로 복사한다.
- `RUN`은 이미지 빌드 중 실행된다.
- `CMD`는 컨테이너 시작 시 실행된다.
- FastAPI 앱은 `uvicorn` 서버를 통해 실행된다.
- 컨테이너 외부에서 접근하려면 `--host 0.0.0.0`과 `-p` 포트 연결이 필요하다.