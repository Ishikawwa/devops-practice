# ---------- Stage 1: builder ----------
FROM python:3.11-alpine AS builder

RUN apk add --no-cache build-base libffi-dev python3-dev

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ---------- Stage 2: runtime ----------
FROM python:3.11-alpine

RUN apk add --no-cache curl \
    && addgroup -S appgroup \
    && adduser -S -G appgroup appuser \
    && mkdir -p /var/log/app \
    && chown appuser:appgroup /var/log/app

WORKDIR /app

COPY --from=builder /install /usr/local
COPY main.py .

ENV LOG_FILE=/var/log/app/app_logs.txt \
    PYTHONUNBUFFERED=1

VOLUME ["/var/log/app"]

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
