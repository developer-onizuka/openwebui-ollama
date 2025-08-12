
# 1. Building Docker image
```
docker build -t llama-cpp-python:arm64 .
```

# 2. Download gguf model image
```
#wget https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q4_K_M.gguf
wget https://huggingface.co/unsloth/gpt-oss-20b-GGUF/resolve/main/gpt-oss-20b-Q4_K_M.gguf
```

# 3. Run the docker image with gguf
```
#docker run -d --rm --name=test -p 8000:8000 -v "$(pwd)/models:/app/models" llama-cpp-python:arm64 python3 -m llama_cpp.server --host 0.0.0.0 --port 8000 --model /app/models/Llama-3.2-1B-Instruct-Q4_K_M.gguf
docker run -d --rm --name=test -p 8000:8000 -v "$(pwd)/models:/app/models" llama-cpp-python:arm64 python3 -m llama_cpp.server --host 0.0.0.0 --port 8000 --model /app/models/gpt-oss-20b-Q4_K_M.gguf
```

# 4. Confirm the response from the llama_cpp server
```
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Who is Tokugawa Yoshinobu ?"
      }
    ],
    "stream": false,
    "max_tokens": 128
  }'
```
