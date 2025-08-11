
```
docker build -t llama-cpp-python:arm64 .
```
```
wget https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_0.gguf
```
```
docker run -d --rm --name=test -p 8000:8000 -e MODEL_URL="https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_0.gguf" -v "$(pwd)/models:/app/models" llama-cpp-python:arm64 python3 -m llama_cpp.server --host 0.0.0.0 --port 8000 --model /app/models/llama-2-7b-chat.Q4_0.gguf
```
```
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "日本の首都はどこですか？"
      }
    ],
    "stream": false,
    "max_tokens": 128
  }'
```
