# Jetson-compatible n8n + PyTorch + Whisper Dockerfile
# Start from Jetson/L4T PyTorch base image (choose the tag for your JetPack version)
FROM dustynv/l4t-pytorch:r36.4.0

# Install system dependencies and build tools
RUN apt-get update && \
    apt-get install -y \
        curl \
        python3 \
        python3-pip \
        git \
        build-essential \
        ca-certificates \
        graphicsmagick \
        tzdata \
        poppler-utils

# Install Node.js (version 22.x, ARM64)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
    apt-get install -y nodejs

# Use build argument for n8n version
ARG N8N_VERSION=1.117.3
RUN npm install -g n8n@${N8N_VERSION}

# Install Whisper using jetson-containers 'uv' helpers (preferred pattern)
# Install ffmpeg runtime dependency
RUN apt-get update && \
    apt-get install -y --no-install-recommends ffmpeg && \
    rm -rf /var/lib/apt/lists/* && apt-get clean
## Ensure pip uses official PyPI (avoid Jetson mirrors)
RUN printf '[global]\nindex-url = https://pypi.org/simple\ntrusted-host = pypi.org files.pythonhosted.org\n' > /etc/pip.conf
ENV PIP_NO_CACHE_DIR=1 PIP_INDEX_URL=https://pypi.org/simple PIP_TRUSTED_HOST=pypi.org,files.pythonhosted.org

## Install necessary Python runtime packages from PyPI
RUN python3 -m pip install --upgrade pip setuptools wheel && \
    python3 -m pip install --no-cache-dir pandas scipy jiwer ipywebrtc || true

ADD https://api.github.com/repos/openai/whisper/git/refs/heads/main /tmp/whisper_version.json

RUN git clone https://github.com/openai/whisper/ /opt/whisper && \
    cd /opt/whisper && \
    sed -i 's/==/>=/g' requirements.txt && \
    sed -i 's/~=/>=/g' requirements.txt && \
    python3 -m pip wheel . --no-deps -w /opt/whisper/dist && \
    python3 -m pip install --no-deps /opt/whisper/dist/openai_whisper*.whl

# Set up working directory and user (optional: match n8n Dockerfile)
WORKDIR /home/node
RUN useradd -ms /bin/bash node || true
RUN mkdir -p /home/node/.n8n && chown -R node:node /home/node
USER node

EXPOSE 5678
CMD ["n8n"]
