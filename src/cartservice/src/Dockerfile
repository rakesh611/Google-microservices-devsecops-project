# ------------------ BUILD STAGE ------------------
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS builder
WORKDIR /app

COPY cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-musl-x64

COPY . .
RUN dotnet publish cartservice.csproj \
    -c Release \
    -r linux-musl-x64 \
    --self-contained true \
    -p:PublishSingleFile=true \
    -p:PublishTrimmed=true \
    -p:TrimMode=Link \
    -o /cartservice \
    --no-restore

# ------------------ RUNTIME STAGE ------------------
FROM mcr.microsoft.com/dotnet/runtime-deps:8.0-alpine AS runtime
WORKDIR /app

COPY --from=builder /cartservice .
EXPOSE 7070

ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070

USER 1000
ENTRYPOINT ["/app/cartservice"]

# ------------------ GRPC HEALTH PROBE ------------------
USER root
ENV GRPC_HEALTH_PROBE_VERSION=v0.4.18
RUN wget -qO /bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

USER 1000
