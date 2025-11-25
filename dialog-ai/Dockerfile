# ================================
# 1) Build Stage: Python 3.8 + libs 설치
# ================================
FROM python:3.8-slim AS builder

WORKDIR /app

# 시스템 의존성 설치 (PyAudio, soundfile, grpc 빌드 위해 필수)
RUN apt-get update && apt-get install -y \
    build-essential \
    portaudio19-dev \
    libsndfile1 \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# pip 업그레이드 + wheel 설치
RUN pip install --upgrade pip setuptools wheel

# requirements.txt 복사
COPY requirements.txt .

# 라이브러리 설치
RUN pip install --no-cache-dir -r requirements.txt

# 소스 코드 복사 (protobuf 컴파일을 위해)
COPY . .

# protobuf 파일 컴파일 (경로 수정)
RUN cd /app && python -m grpc_tools.protoc \
    --proto_path=. \
    --python_out=. \
    --grpc_python_out=. \
    stt/nest/nest.proto

# 생성된 파일 확인
RUN ls -la /app/stt/nest/

# __init__.py 생성
RUN echo "" > /app/stt/nest/__init__.py


# ================================
# 2) Run Stage
# ================================
FROM python:3.8-slim

WORKDIR /app

# 시스템 의존성 (런타임) - 최소화
RUN apt-get update && apt-get install -y \
    portaudio19-dev \
    libportaudio2 \
    libsndfile1 \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# builder에서 패키지 + 컴파일된 파일 복사
COPY --from=builder /usr/local/lib/python3.8 /usr/local/lib/python3.8
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /app /app

EXPOSE 8000

# FastAPI 실행
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]