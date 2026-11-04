# syntax=docker/dockerfile:1.7

# -----------------------------------------------------------------------------
# Stage 1 - dependencies
# -----------------------------------------------------------------------------

FROM python:3.12-slim AS builder

WORKDIR /build

RUN apt-get update \
    && apt-get install --no-install-recommends -y build-essential libffi-dev \
    && rm -rf /var/lib/apt/lists/*
# --no-install-recommends: Không cài gợi ý đi kèm tiết kiệm dung lượng.

COPY requirements.txt .

RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt
# --no-cache-dir: Không lưu cache.
# --wheel-dir: Set đường dẫn cho .whl sau khi compile.

# -----------------------------------------------------------------------------
# Stage 2 - runtime
# -----------------------------------------------------------------------------

FROM python:3.12-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \ 
    PYTHONUNBUFFERED=1

RUN apt-get update \
    && apt-get install --no-install-recommends -y curl tini \
    && rm -rf /var/lib/apt/lists/* \
    && useradd --create-home --shell /usr/sbin/nologin --uid 10001 appuser
# --create-home: Tạo thư mục /home/appuser.
# --shell: Khóa quyền đăng nhập shell của user này cho bảo mật.
# --uid: Cố định user id.

WORKDIR /app

COPY --from=builder /wheels /wheels
COPY requirements.txt .

RUN pip install --no-cache-dir --no-index --find-links=/wheels -r requirements.txt \
    && rm -rf /wheels
# --no-index: Cấm kết nối ra ngoài Internet, thư viện lấy từ nguồn local.
# --find-links: Tìm thư mục wheels.

COPY --chown=appuser:appuser app.py . 

USER appuser
EXPOSE 5000

HEALTHCHECK --interval=15s --timeout=5s --start-period=30s --retries=5 \
     CMD curl --fail --silent http://127.0.0.1:5000/health || exit 1
# --fail: Báo mỗi HTTP >= 400.
# --silent: Chạy im lặng, không log.

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD ["gunicorn", \
    "--bind" , "0.0.0.0:5000", \
    "--workers", "2", \
    "--threads", "4", \
    "--timeout", "60", \
    "--access-logfile", "-", \
    "app:app"]