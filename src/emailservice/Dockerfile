# ------------------------
# Base image
# ------------------------
FROM python:3.11-slim AS base

# ------------------------
# Builder stage
# ------------------------
FROM base AS builder

RUN apt-get -qq update && \
    apt-get install -y --no-install-recommends \
        wget g++ && \
    rm -rf /var/lib/apt/lists/*

# Download grpc health probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /install

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ------------------------
# Runtime image
# ------------------------
FROM base AS runtime

ENV PYTHONUNBUFFERED=1
ENV ENABLE_PROFILER=1

WORKDIR /email_server

# Copy installed python packages (✅ correct python version)
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy grpc health probe
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe

# Copy application code
COPY . .

EXPOSE 8080
ENTRYPOINT ["python", "email_server.py"]
