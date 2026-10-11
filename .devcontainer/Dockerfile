FROM mcr.microsoft.com/devcontainers/python:3.12-bookworm
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl git ripgrep jq tzdata \
    && rm -rf /var/lib/apt/lists/*

RUN GH_RELEASE=$(curl -fsSL https://api.github.com/repos/cli/cli/releases/latest) \
    && GH_DEB=$(echo "$GH_RELEASE" | grep -o 'https://[^"]*linux_amd64\.deb' | head -1) \
    && if [ -n "$GH_DEB" ]; then \
         curl -fsSL "$GH_DEB" -o /tmp/gh.deb \
         && apt-get update && apt-get install -y /tmp/gh.deb \
         && rm -f /tmp/gh.deb; \
       else \
         GH_TAR=$(echo "$GH_RELEASE" | grep -o 'https://[^"]*linux_amd64\.tar\.gz' | head -1) \
         && curl -fsSL "$GH_TAR" -o /tmp/gh.tar.gz \
         && tar -xzf /tmp/gh.tar.gz -C /usr/local --strip-components=1 \
         && rm -f /tmp/gh.tar.gz; \
       fi \
    && rm -rf /var/lib/apt/lists/* \
    && gh --version

RUN pip install --no-cache-dir \
    "fastapi>=0.115" \
    "uvicorn[standard]>=0.32" \
    "httpx>=0.27" \
    "pytest>=8.3" \
    "pytest-asyncio>=0.24"

USER vscode
ENV PATH="/home/vscode/.local/bin:${PATH}"

RUN curl -fsSL https://claude.ai/install.sh | bash -s stable \
    && claude --version \
    && mkdir -p /home/vscode/.claude
