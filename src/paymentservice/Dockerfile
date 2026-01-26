# ---------- Base ----------
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# ---------- Builder ----------
FROM base AS builder
# Copy package files and install only production dependencies
COPY package*.json ./
RUN npm ci --omit=dev
# Copy main application files
COPY index.js ./
# COPY src ./src       # Uncomment only if needed
# COPY config ./config # Uncomment only if needed

# ---------- Runtime ----------
FROM node:20-alpine AS runtime
WORKDIR /usr/src/app
# Copy only node_modules and main files from builder
COPY --from=builder /usr/src/app/node_modules ./node_modules
COPY --from=builder /usr/src/app/index.js ./index.js
# COPY --from=builder /usr/src/app/src ./src  # Uncomment if needed
EXPOSE 50051
ENTRYPOINT ["node", "index.js"]

# ---------- Runtime with grpc-health-probe ----------
FROM runtime AS grpc-probe
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 \
    && chmod +x /bin/grpc_health_probe

# ---------- OPTIONAL: Create non-root user for security ----------
RUN addgroup -S nonroot && adduser -S nonroot -G nonroot
USER nonroot
