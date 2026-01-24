# --------------------------------------------------
# Base image (use supported Alpine)
# --------------------------------------------------
FROM node:20-alpine AS base

# --------------------------------------------------
# Builder stage
# --------------------------------------------------
FROM base AS builder

# Required for native npm modules
RUN apk add --no-cache \
    python3 \
    make \
    g++

WORKDIR /usr/src/app

# Copy only package files first (layer caching)
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# --------------------------------------------------
# Runtime image (without grpc probe)
# --------------------------------------------------
FROM base AS runtime

WORKDIR /usr/src/app

# Copy node_modules from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules

# Copy application source
COPY . .

# Create non-root user (SECURITY BEST PRACTICE)
RUN addgroup -S nodegroup && adduser -S nodeuser -G nodegroup
USER nodeuser

EXPOSE 50051

ENTRYPOINT ["node", "index.js"]

# --------------------------------------------------
# Runtime image WITH grpc health probe
# --------------------------------------------------
FROM runtime AS with-grpc-health-probe

# renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

