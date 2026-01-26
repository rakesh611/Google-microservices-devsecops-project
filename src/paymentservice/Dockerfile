# ---------- Base ----------
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# ---------- Builder ----------
FROM base AS builder
# Copy package.json and package-lock.json
COPY package*.json ./
# Install only production dependencies
RUN npm ci --omit=dev
# Copy app source files
COPY index.js ./
# COPY src ./src   # Uncomment if needed
# COPY config ./config   # Uncomment if needed

# ---------- Runtime (no grpc probe) ----------
FROM node:20-alpine AS runtime
WORKDIR /usr/src/app
# Copy only node_modules and main app files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src   # Uncomment if needed
EXPOSE 50051
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime (with grpc-health-probe) ----------
FROM runtime AS grpc-probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

# Use non-root user (optional but recommended)
RUN addgroup -S nonroot && adduser -S nonroot -G nonroot
USER nonroot
