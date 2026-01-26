# ---------- Base ----------
FROM node:20.2.0-alpine AS base
WORKDIR /usr/src/app

# ---------- Builder ----------
FROM base AS builder

# Install build dependencies for post-install scripts
RUN apk add --update --no-cache python3 make g++ 

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production

# Copy source code (optional if needed)
COPY index.js ./
# COPY src ./src
# COPY config ./config

# ---------- Runtime (no grpc probe) ----------
FROM base AS runtime

# Create a non-root user
RUN addgroup -S nonroot && adduser -S nonroot -G nonroot

WORKDIR /usr/src/app

# Copy node_modules and main files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src
# COPY --from=builder /usr/src/app/config ./config

# Expose port
EXPOSE 50051

# Switch to non-root user
USER nonroot

# Start the application
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime (with grpc-health-probe) ----------
FROM runtime AS grpc-probe

ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

# Download grpc_health_probe securely
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe
