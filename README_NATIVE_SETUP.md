# Native Setup

Native installation for direct GPU access and optimal performance. No containerization.

## Installation

```bash
./install.sh  # Python venv, dependencies, symlink
./setup.sh    # + Ollama, Redis, environment configuration
```

Scripts auto-detect package manager (apt/dnf/yum/pacman).

## Dependencies

**Python 3.11 or 3.12 + dev headers**
- Python 3.13+ not supported (dependency constraints)
- Requires C toolchain for tree-sitter and cryptography compilation
- Linux: `python3-dev` or `python3-devel`, `libssl-dev`, `libffi-dev`, `build-essential`
- macOS: Xcode Command Line Tools

**Runtime (optional)**
- Ollama: Local LLM inference with GPU acceleration
- Redis: Embedding cache and session storage
- Node.js 18+: React UI (port 5173)

## Ports

- 8051: MCP server (FastAPI)
- 5173: Frontend dev server (Vite)
- 11434: Ollama API
- 6379: Redis

## Configuration

**Environment variables** (`.env`):
```bash
ANTHROPIC_API_KEY=sk-...           # Required for Claude API
OLLAMA_BASE_URL=http://localhost:11434  # Optional: remote Ollama
REDIS_HOST=localhost               # Optional: remote Redis
```

**Model selection** (`app/core/config.py`):
- Default: `llama3.1:8b` (Ollama)
- Alternative: OpenAI models via API

## Manual Component Installation

**Ollama:**
```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.ai/install.sh | sh
```

**Redis:**
```bash
# macOS
brew install redis && brew services start redis

# Linux (apt)
sudo apt install redis-server && sudo systemctl enable redis-server
```

**Node.js:**
```bash
# macOS
brew install node

# Linux (NodeSource)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs
```

## GPU Acceleration

**macOS (Metal):**
- Native Ollama automatically uses Metal GPU
- Verify: `ollama run llama3.1:8b` should show GPU activity

**Linux (NVIDIA CUDA):**
- Install NVIDIA drivers + CUDA toolkit
- Ollama detects CUDA automatically
- Verify: `nvidia-smi` while running inference

**Linux (AMD ROCm):**
- Install ROCm drivers
- Set `HSA_OVERRIDE_GFX_VERSION` if needed
- Verify: `rocm-smi` while running inference

## Troubleshooting

**Python compilation fails:**
- Missing dev headers: Install `python3-dev` + `libssl-dev` + `libffi-dev`
- Missing compiler: Install `build-essential` or equivalent

**Port conflicts:**
- Check: `lsof -i :8051` (or `:5173`, `:11434`, `:6379`)
- Kill process or reconfigure ports in `.env`

**Ollama not using GPU:**
- macOS: Metal should work by default, check Activity Monitor GPU usage
- Linux NVIDIA: Verify `nvidia-smi` shows driver, check `ollama serve` logs
- Linux AMD: Set ROCm environment variables, check `rocm-smi`

**MCP server won't start:**
- Check logs: `tail -f ~/.claude/logs/server.log`
- Verify Python venv: `which python` should point to `.venv/bin/python`
- Test manually: `python mcp_server/server.py`

## Script Internals

**install.sh:**
- Validates Python 3.11/3.12
- Creates `.venv` and installs `requirements.txt`
- Symlinks `claude` command to `~/.local/bin`
- Creates `~/.claude/` directory structure

**setup.sh:**
- Runs install.sh steps
- Installs Ollama (if missing)
- Installs/starts Redis (if missing)
- Pulls default model: `llama3.1:8b`
- Configures systemd/launchd services (optional)

**scripts/setup_native.sh:**
- macOS-specific native Ollama setup
- Homebrew installation and configuration
- Metal GPU verification

## Related Documentation

- [README.md](README.md) - Main documentation
- [install.sh](install.sh) - Installation script source
- [setup.sh](setup.sh) - Full setup script source
- [docs/API_REFERENCE.md](docs/API_REFERENCE.md) - MCP server API endpoints
