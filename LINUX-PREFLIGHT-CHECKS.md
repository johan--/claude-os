# Linux Preflight

Quick checks before installing Claude OS on Debian/Ubuntu/Fedora/RHEL/Arch.

## Dependencies

**Python 3.11/3.12 + C toolchain**
```bash
python3 --version  # 3.11 or 3.12 (not 3.13+)
gcc --version      # For tree-sitter, cryptography compilation
```

**Required packages:**
- Debian/Ubuntu: `build-essential python3-dev libssl-dev libffi-dev`
- Fedora/RHEL: `@development-tools python3-devel openssl-devel libffi-devel`
- Arch: `base-devel python libffi openssl`

**Optional:**
- Node.js 18+ (React UI) - see Node.js installation below
- Redis (embedding cache)
- Ollama (local LLM)

## Node.js Installation (Optional)

**Via package manager:**
```bash
# Debian/Ubuntu - NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs
```

**Via version manager (recommended for multi-project environments):**
```bash
# mise (polyglot tool manager)
curl https://mise.run | sh
mise use --global node@20

# nvm (Node-specific)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
```

## Port Availability

Claude OS needs: 8051 (MCP), 5173 (UI), 11434 (Ollama), 6379 (Redis)

```bash
ss -ltn '( sport = :8051 or sport = :5173 or sport = :11434 or sport = :6379 )'
```

If occupied: `lsof -i :8051` and kill process or reconfigure.

## Quick Validation

```bash
# All in one check
python3 --version && \
gcc --version && \
pkg-config --version && \
test -w "$HOME" && \
echo "Ready for install"
```

## Post-Install Verification

```bash
curl -f http://localhost:8051/health  # MCP server
redis-cli ping                         # Redis
ollama list                            # Ollama
```

If MCP server fails: `tail -f ~/.claude/logs/server.log`

## Common Issues

**pip install fails on tree-sitter:**
- Missing: `python3-dev`, `build-essential`

**pip install fails on cryptography:**
- Missing: `libssl-dev`, `libffi-dev`

**Port conflicts:**
- Check `lsof -i :<port>` and kill or reconfigure

**Ollama GPU not detected:**
- NVIDIA: Verify `nvidia-smi` shows driver
- AMD: Set `HSA_OVERRIDE_GFX_VERSION` if needed

## Related

- [README_NATIVE_SETUP.md](README_NATIVE_SETUP.md) - Full installation guide
- [GAP-ANALYSIS.md](GAP-ANALYSIS.md) - Identified documentation gaps
