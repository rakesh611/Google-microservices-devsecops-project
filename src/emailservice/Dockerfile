FROM python:3.10-slim AS base

# ---------- Builder ----------
FROM base AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    wget g++ \
    libunwind-dev \
    libstdc++6 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ---------- Runtime ----------
FROM base AS runtime

RUN apt-get update && apt-get install -y --no-install-recommends \
    libunwind8 \
    libstdc++6 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ENV PYTHONUNBUFFERED=1
ENV ENABLE_PROFILER=1

WORKDIR /email_server

COPY --from=builder /usr/local/lib/python3.10 /usr/local/lib/python3.10
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe

COPY . .

EXPOSE 8080
ENTRYPOINT ["python", "email_server.py"]
