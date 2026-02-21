---
type: doc
doc: troubleshooting
updated: 2026-01-17
---

# Troubleshooting

Issue log for docker-mcp Python integration.

## Quick Reference

| Date | Issue | Status |
|------|-------|--------|
| 2026-01-17 | Migrated from Ruby to Python | Complete |

---

## Migration History

### 2026-01-17: Ruby → Python Migration

**Status:** Complete

**Reason:** Ruby version (docker_mcp gem v0.3.0) had 3 critical Windows bugs:
1. Empty `required: []` arrays in JSON Schema (tool_forge bug)
2. Windows path corruption (D:/tools/ruby34/... → D:/tools)
3. File operations not Windows-compatible (mcp-ruby)

**Resolution:**
- Uninstalled Ruby gems
- Archived Ruby documentation to `D:/workspace/archive/ruby-docker-mcp/`
- Cloned Python `mcp-server-docker` from ckreiling
- Installed via `uv sync`
- Created new documentation

**Files Changed:**
- `D:\servers\docker-mcp\` - Replaced with Python fork
- `D:\workspace\archive\ruby-docker-mcp\` - Ruby backup

---

## Known Issues

### None (Python version works on Windows)

---

## Common Issues

### Docker Desktop Not Running

**Status:** Common

**Symptoms:**
- Tools return "Cannot connect to Docker daemon"
- Timeout errors
- "Error while fetching server API version"

**Fix:**
1. Start Docker Desktop
2. Wait for it to fully initialize (green icon)
3. Verify: `docker ps` works in terminal
4. Retry MCP operation

### uv Not Installed

**Status:** Prevention

**Symptoms:**
```
uv: command not found
```

**Fix:**
```bash
# Install uv (Python package manager)
pip install uv

# Or via official installer
# https://github.com/astral-sh/uv
```

### Python Version Too Old

**Status:** Prevention

**Symptoms:**
```
ERROR: This package requires Python 3.12+
```

**Fix:**
```bash
# Check version
python --version

# Install Python 3.12+
# https://www.python.org/downloads/
```

---

## Debugging

### Enable Verbose Logging

```bash
# Run with debug output
DEBUG=1 uv run mcp-server-docker
```

### Check Docker Connectivity

```bash
# Basic connectivity test
docker ps

# API version
docker version

# Windows socket test
//./pipe/docker_engine
```

### Test with MCP Inspector

```bash
npx @modelcontextprotocol/inspector uv run mcp-server-docker
```

---

## Proxy Logs

Check master-mcp-proxy logs for tool invocation issues:

```bash
tail -100 D:/workspace/logs/master-mcp/master-mcp-*.log
```
