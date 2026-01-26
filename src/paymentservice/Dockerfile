# ---------- Base ----------
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# Install build dependencies for some npm packages
RUN apk add --no-cache python3 make g++

# ---------- Builder ----------
FROM base AS builder
WORKDIR /usr/src/app

# Copy package files and install only production dependencies
COPY package*.json ./
RUN npm ci --omit=dev

# Copy application files
COPY index.js ./
# COPY src ./src       # Uncomment if needed
# COPY config ./config # Uncomment if needed

# ---------- Runtime (no grpc-health-probe) ----------
FROM node:20-alpine AS runtime
WORKDIR /usr/src/app

# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy node_modules and main files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src       # Uncomment if needed
# COPY --from=builder /usr/src/app/config ./config # Uncomment if needed

EXPOSE 50051
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime (with grpc-health-probe) ----------
FROM runtime AS grpc-probe
USER appuser

ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18

# Download grpc_health_probe into the user's folder and add to PATH
RUN wget -qO /usr/src/app/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /usr/src/app/grpc_health_probe

ENV PATH="/usr/src/app:${PATH}"
