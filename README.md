# vLLM CPU Docker Compose Setup

This Docker Compose configuration builds and runs vLLM with CPU support using the OpenAI-compatible API server.

## Quick Start

1. **Build and run the service:**

   ```bash
   docker-compose up --build
   ```

2. **Run in detached mode:**

   ```bash
   docker-compose up -d --build
   ```

3. **View logs:**

   ```bash
   docker-compose logs -f vllm-cpu
   ```

4. **Stop the service:**
   ```bash
   docker-compose down
   ```

## Configuration

### Environment Variables

Before running, you should customize the environment variables in the `docker-compose.yml` file according to your system:

- **`VLLM_CPU_KVCACHE_SPACE`**: KV cache space in GB (default: 4)
- **`VLLM_CPU_OMP_THREADS_BIND`**: CPU cores for inference (default: 0-7)

### Example configurations:

**For a 4-core system:**

```yaml
environment:
  - VLLM_CPU_KVCACHE_SPACE=2
  - VLLM_CPU_OMP_THREADS_BIND=0-3
```

**For a 16-core system:**

```yaml
environment:
  - VLLM_CPU_KVCACHE_SPACE=8
  - VLLM_CPU_OMP_THREADS_BIND=0-15
```

**For specific core binding:**

```yaml
environment:
  - VLLM_CPU_KVCACHE_SPACE=4
  - VLLM_CPU_OMP_THREADS_BIND=0,2,4,6,8,10,12,14
```

### Model Configuration

To change the model, modify the `command` section in `docker-compose.yml`:

```yaml
command: >
  --model=microsoft/DialoGPT-medium
  --dtype=float16
  --host=0.0.0.0
  --port=8000
```

## API Usage

Once running, the OpenAI-compatible API will be available at `http://localhost:8000`.

### Example API call:

```bash
curl http://localhost:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.2-1B-Instruct",
    "prompt": "Hello, how are you?",
    "max_tokens": 50
  }'
```

## Resource Management

The compose file includes optional resource limits. Adjust these based on your system:

```yaml
deploy:
  resources:
    limits:
      memory: 16G # Maximum memory usage
    reservations:
      memory: 8G # Reserved memory
```

## Troubleshooting

1. **Out of memory errors**: Reduce `VLLM_CPU_KVCACHE_SPACE` or increase system memory
2. **Slow inference**: Adjust `VLLM_CPU_OMP_THREADS_BIND` to use more CPU cores
3. **Model download issues**: The compose file mounts `~/.cache/huggingface` to cache models locally

## Original Docker Commands

This compose file replaces these manual Docker commands:

```bash
# Build
docker build -f Dockerfile.cpu --tag vllm-cpu-env --target vllm-openai .

# Run
docker run --rm \
  --privileged=true \
  --shm-size=4g \
  -p 8000:8000 \
  -e VLLM_CPU_KVCACHE_SPACE=4 \
  -e VLLM_CPU_OMP_THREADS_BIND=0-7 \
  vllm-cpu-env \
  --model=meta-llama/Llama-3.2-1B-Instruct \
  --dtype=bfloat16 \
  --host=0.0.0.0 \
  --port=8000
```
