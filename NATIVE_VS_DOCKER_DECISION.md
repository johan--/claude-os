# Native vs Docker Architecture Decision

## Decision

Claude OS uses native Ollama installation. No Docker/containerization.

## Rationale

**GPU acceleration performance:**
- Native: Direct Metal (macOS) or CUDA/ROCm (Linux) access
- Docker: Requires GPU passthrough, adds virtualization overhead
- macOS Docker GPU support is limited/experimental

**Target use case:**
- Single-user local development tool
- Not multi-tenant or production deployment
- Simpler mental model: `ollama serve` vs container orchestration

**Setup complexity trade-off:**
- Native wins for local dev (one command: `./setup.sh`)
- Docker would require: compose files, volume mounts, GPU runtime configuration
- macOS users would need Docker Desktop (resource overhead)

## Performance Data

**Current (M4 Pro, Native Ollama):**
- Query response: ~40s with llama3.1:8b
- Full Metal GPU utilization
- Memory: 8-10GB (model + context)
- Indexing: 30s for 10k files (hybrid system)

**Docker overhead (estimated):**
- +10-20% latency from virtualization
- GPU passthrough complexity on macOS
- Additional memory for Docker daemon
- No formal benchmarks yet

## Trade-offs

| Aspect | Native | Docker |
|--------|--------|--------|
| GPU Access | Direct | Passthrough required |
| Setup | Package manager | Compose + GPU runtime |
| Isolation | System-wide | Container-level |
| Updates | `brew upgrade ollama` | Image rebuild |
| Multi-version | Difficult | Trivial |
| macOS Support | Full Metal GPU | Limited GPU support |
| Linux Support | Full CUDA/ROCm | nvidia-docker required |

## Docker Alternative (Not Implemented)

For users who prefer containers:

```yaml
# docker-compose.yml (hypothetical)
services:
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  mcp-server:
    build: .
    ports:
      - "8051:8051"
    depends_on:
      - ollama
      - redis
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  frontend:
    build:
      context: ./frontend
    ports:
      - "5173:5173"
```

macOS GPU passthrough still problematic.

## Reconsideration Triggers

Consider Docker if:
- Multi-tenant deployment needed
- CI/CD isolation required
- Version pinning becomes critical
- macOS Docker GPU support improves significantly

## Related

- [README_NATIVE_SETUP.md](README_NATIVE_SETUP.md) - Native installation guide
- [PERFORMANCE_TEST_RESULTS.md](PERFORMANCE_TEST_RESULTS.md) - Benchmark data
- README.md lines 778-793 - Current performance metrics
