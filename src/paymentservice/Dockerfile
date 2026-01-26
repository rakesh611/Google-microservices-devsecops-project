# ---------- Base ----------
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# ---------- Builder ----------
FROM base AS builder
COPY package*.json ./
RUN npm ci --omit=dev
COPY index.js ./
# COPY src ./src   # Uncomment if needed
# COPY config ./config   # Uncomment if needed

# ---------- Runtime (no grpc probe) ----------
FROM node:20-alpine AS runtime
WORKDIR /usr/src/app

# Copy node_modules and main files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src   # Uncomment if needed

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to non-root
USER appuser

EXPOSE 50051
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime (with grpc-health-probe) ----------
FROM runtime AS grpc-probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

# Temporarily switch to root to install grpc_health_probe
USER root
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

# Switch back to non-root
USER appuser
