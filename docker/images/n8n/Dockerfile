# ==================================================================
# Stage 1: Base system dependencies (same as your original)
# ==================================================================
ARG NODE_VERSION=22
ARG N8N_VERSION=1.109.2
ARG LAUNCHER_VERSION=1.1.3
ARG TARGETPLATFORM

FROM n8nio/base:${NODE_VERSION} AS system-deps

# ==================================================================
# Stage 2: Application Source
# ==================================================================
FROM alpine:3.22.0 AS app-artifact-processor

# Copy source code (frontend + backend)
COPY . /app/

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm

# Set Node memory limit for heavy frontend builds
ENV NODE_OPTIONS="--max-old-space-size=4096"

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build all packages including @n8n/chat (AI frontend)
RUN pnpm run build

# ==================================================================
# Stage 3: Task Runner Launcher
# ==================================================================
FROM alpine:3.22.0 AS launcher-downloader
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION

RUN set -e; \
    case "$TARGETPLATFORM" in \
        "linux/amd64") ARCH_NAME="amd64" ;; \
        "linux/arm64") ARCH_NAME="arm64" ;; \
        *) echo "Unsupported platform: $TARGETPLATFORM" && exit 1 ;; \
    esac; \
    mkdir /launcher-temp && cd /launcher-temp; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz"; \
