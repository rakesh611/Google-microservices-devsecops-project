# ---------- Base ----------
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# ---------- Builder ----------
FROM base AS builder
COPY package*.json ./
RUN npm ci --omit=dev --ignore-scripts

COPY index.js ./
# COPY src ./src   # only if needed
# COPY config ./config

# ---------- Runtime (no grpc probe) ----------
FROM node:20-alpine AS runtime
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src
EXPOSE 50051
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime (with grpc health probe) ----------
FROM runtime
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe
