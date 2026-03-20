# =============================================
# 오프너(Opener) — Production Dockerfile
# =============================================
# 빌드:  docker build -t opener .
# 실행:  docker run -p 8501:8501 --env-file .env opener
# =============================================

# ── 1단계: 빌더 ─────────────────────────────
FROM python:3.11-slim AS builder

WORKDIR /build

# 시스템 빌드 의존성
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 의존성 먼저 설치 (레이어 캐시 최적화)
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt


# ── 2단계: 런타임 ────────────────────────────
FROM python:3.11-slim AS runtime

# 한국어 폰트 + tesseract + 폰트 도구
RUN apt-get update && apt-get install -y --no-install-recommends \
    fonts-noto-cjk \
    tesseract-ocr \
    tesseract-ocr-kor \
    libglib2.0-0 \
    libgomp1 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 의존성 복사
COPY --from=builder /install /usr/local

# 앱 코드
WORKDIR /app
COPY . .

# Streamlit 설정
RUN mkdir -p /app/.streamlit
COPY .streamlit/config.toml /app/.streamlit/config.toml

# matplotlib 캐시 디렉토리
ENV MPLCONFIGDIR=/tmp/matplotlib
RUN mkdir -p /tmp/matplotlib

# 비루트 사용자 (보안)
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app /tmp/matplotlib
USER appuser

# 포트
EXPOSE 8501

# 헬스체크
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD curl -f http://localhost:8501/_stcore/health || exit 1

# 실행
ENTRYPOINT ["streamlit", "run", "app.py", \
  "--server.port=8501", \
  "--server.address=0.0.0.0", \
  "--server.headless=true", \
  "--browser.gatherUsageStats=false"]
