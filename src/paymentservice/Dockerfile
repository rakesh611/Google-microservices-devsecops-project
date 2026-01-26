# ---------- Base Image ----------
FROM node:20.2.0-alpine AS base

# Install required build tools for some Node packages
RUN apk add --update --no-cache python3 make g++ wget

WORKDIR /usr/src/app

# ---------- Builder Stage ----------
FROM base AS builder

# Copy package.json and install only production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy main application files
COPY index.js ./
# COPY src ./src       # Uncomment if needed
# COPY config ./config # Uncomment if needed

# ---------- Runtime Stage (non-root) ----------
FROM base AS runtime

# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

WORKDIR /usr/src/app

# Copy node_modules and main files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src   # Uncomment if needed

EXPOSE 50051

ENTRYPOINT ["node", "index.js"]

# ---------- Runtime with grpc-health-probe ----------
FROM runtime AS grpc-probe

ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

# Temporarily switch to root to install grpc_health_probe
USER root

RUN wget -qO /usr/local/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /usr/local/bin/grpc_health_probe

# Switch back to non-root user
USER appuser
