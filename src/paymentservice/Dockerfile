# =========================
# Base Image
# =========================
FROM node:20-alpine AS base

# =========================
# Builder Stage
# =========================
FROM base AS builder

RUN apk add --no-cache \
    python3 \
    make \
    g++

WORKDIR /usr/src/app

COPY package*.json ./

# Use modern npm flag
RUN npm ci --omit=dev

# =========================
# Runtime Stage
# =========================
FROM base AS runtime

WORKDIR /usr/src/app

# Copy node_modules from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules

# Copy application source
COPY . .

# =========================
# Install grpc health probe (AS ROOT)
# =========================
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

RUN wget -qO /usr/local/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /usr/local/bin/grpc_health_probe

# =========================
# Create Non-Root User
# =========================
RUN addgroup -S nodegroup && adduser -S nodeuser -G nodegroup

USER nodeuser

EXPOSE 50051

ENTRYPOINT ["node", "index.js"]
