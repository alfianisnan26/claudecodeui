# Self-hosted CloudCLI image for Kubernetes (isnan cluster).
# Builds the UI from this checkout and bundles the Claude Code CLI.
FROM node:22-bookworm AS build
WORKDIR /app
COPY package.json package-lock.json ./
COPY scripts ./scripts
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build && npm prune --omit=dev --no-audit --no-fund

FROM node:22-bookworm
ARG CLAUDE_CODE_VERSION=latest
RUN apt-get update && apt-get install -y --no-install-recommends \
      ripgrep jq sqlite3 zip unzip tree vim-tiny less tini \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g --no-audit --no-fund @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION} \
    && userdel -r node \
    && useradd -m -u 1000 -s /bin/bash cloudcli
COPY --from=build --chown=cloudcli:cloudcli /app /app
ENV NODE_ENV=production \
    HOST=0.0.0.0 \
    SERVER_PORT=3001 \
    HOME=/home/cloudcli \
    DATABASE_PATH=/home/cloudcli/.cloudcli/auth.db \
    WORKSPACES_ROOT=/home/cloudcli
USER cloudcli
WORKDIR /home/cloudcli
EXPOSE 3001
ENTRYPOINT ["tini", "--"]
CMD ["node", "/app/dist-server/server/index.js"]
