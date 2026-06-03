FROM python:3.11

# 컨테이너 내부 작업 폴더
WORKDIR /app

COPY requirements.txt .

# 이미지 생성 중 실행할 명령
RUN pip install -r requirements.txt

COPY . .

# 컨테이너 시작 시 실행
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]