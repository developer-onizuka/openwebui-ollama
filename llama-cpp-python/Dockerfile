FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-dev \
    build-essential \
    git \
    cmake \
    libopenblas-dev \
    libblas-dev \
    libcurl4-openssl-dev \
    ninja-build \
    --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /app
RUN git clone https://github.com/ggerganov/llama.cpp.git
WORKDIR /app/llama.cpp
RUN mkdir build
WORKDIR /app/llama.cpp/build
RUN cmake ..
RUN cmake --build . --config Release
RUN pip install llama-cpp-python \
    "fastapi>=0.104.0" \
    "uvicorn>=0.23.2" \
    "sse-starlette>=2.0.0" \
    "starlette-context>=0.4.0" \
    "pydantic-settings>=2.0.0"
CMD ["python3", "-m", "llama_cpp.server", "--help"]
