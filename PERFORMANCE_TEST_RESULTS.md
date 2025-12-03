# Performance Test Results

## Baseline Configuration

**Hardware:** M4 Pro (12 cores), Metal GPU, macOS Sequoia 15.x
**Model:** llama3.1:8b via Ollama
**Test date:** 2025-12-03

## Current Metrics

### Query Performance

| Metric | Value |
|--------|-------|
| Response time | ~40s per query |
| GPU utilization | Full Metal GPU |
| Memory usage | 8-10GB (model + context) |
| CPU cores | 12 cores active |

### Indexing Performance (Hybrid System)

| Project Size | Before Hybrid | After Hybrid | Improvement |
|--------------|---------------|--------------|-------------|
| 10k files | 3-5 hours | 30s (BM25) + 20min (optional embeddings) | ~90% faster initial |
| Chunks embedded | 100,000+ | ~20,000 | 80% reduction |
| Resource usage | High Ollama load | Minimal CPU/memory | Significant |

Source: HYBRID_INDEXING_DESIGN.md

## Missing Benchmarks

**Not yet tested:**
- Latency distribution (p50, p95, p99)
- Concurrent query handling
- Different model sizes (7B vs 70B)
- Cross-platform comparison (Intel Mac, Linux NVIDIA/AMD)
- Time to first token
- Tokens per second throughput
- Memory usage over time
- Incremental indexing performance

## Planned Test Scenarios

### 1. Query Latency Distribution

```python
# Test plan
queries = [
    "Explain this function",
    "Write a JSON parser",
    "Find the bug in this code",
    "How does indexing work?"
]
runs = 100 per query type
metrics = [p50, p95, p99, max, tokens/sec]
```

### 2. Concurrent Load

```python
# Test plan
concurrent_users = [1, 5, 10, 20]
query_mix = "code_gen:50%, explanation:30%, debugging:20%"
metrics = [throughput, queue_time, error_rate]
```

### 3. Hardware Comparison

| Hardware | CPU | GPU | RAM | Storage | Status |
|----------|-----|-----|-----|---------|--------|
| M4 Pro | 12 cores | Metal | TBD | TBD | Baseline |
| M1 Mac | 8 cores | Metal | TBD | TBD | Not tested |
| Linux NVIDIA | Ryzen | RTX 4090 | TBD | TBD | Not tested |
| Linux AMD | Ryzen | RX 7900 | TBD | TBD | Not tested |
| CPU-only | High-end | None | TBD | TBD | Not tested |

### 4. Model Size Impact

| Model | Parameters | Expected Latency | Expected Memory | Status |
|-------|------------|------------------|-----------------|--------|
| llama3.1:8b | 8B | 40s (baseline) | 8-10GB | Tested |
| llama3.1:70b | 70B | TBD | TBD | Not tested |
| codellama:7b | 7B | TBD | TBD | Not tested |
| codellama:13b | 13B | TBD | TBD | Not tested |
| mistral:7b | 7B | TBD | TBD | Not tested |

## Test Methodology (Planned)

**Setup:**
- Cold start: System restart, clear caches
- Warm-up: 5 queries before measurement
- Sample size: 100+ queries per test
- Repetitions: 3 runs per configuration
- Statistical analysis: mean, median, stddev, p95, p99

**Tools:**
- Custom Python profiling scripts
- `htop`/Activity Monitor for resource monitoring
- Ollama metrics API
- Structured logging with timestamps

**Query corpus:**
- Code generation (synthetic)
- Code explanation (from real repos)
- Debugging scenarios
- Documentation queries
- Architecture questions

## Optimization Targets

**Current bottlenecks (suspected, not profiled):**
- Serial query processing (no concurrent handling)
- Model loading time (if swapping models)
- Embedding generation for large files
- Context window management

**Performance goals:**
- Reduce query latency to <30s
- Support 10+ concurrent queries
- Optimize memory footprint
- Maximize GPU utilization %

## Contributing Benchmarks

To submit benchmark data:

1. Run test suite (scripts/benchmark.py - not yet implemented)
2. Include: hardware specs, OS, Ollama version, model
3. Provide raw data (JSON/CSV)
4. Note any anomalies

Submit via PR or issue with tag `performance`.

## Related

- [README.md](README.md) lines 778-793 - Current performance summary
- [README.md](README.md) lines 90-100 - Hybrid indexing gains
- [HYBRID_INDEXING_DESIGN.md](docs/HYBRID_INDEXING_DESIGN.md) - Architecture
- [tests/TEST_COVERAGE.md](tests/TEST_COVERAGE.md) line 178 - Performance testing TODO
