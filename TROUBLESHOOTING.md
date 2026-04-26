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
| 2026-04-26 | WSL2 VM deadlock — Docker engine init hangs | Reboot required |
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

### 2026-04-26: WSL2 VM Deadlock — Docker Engine Stuck on Starting

**Status:** Reboot required (non-reboot paths exhausted)

**Symptoms:**
- Docker Desktop stuck on "Starting the Docker Engine"
- `docker info` / `docker ps` return 500 Internal Server Error
- Backend log: init API `/ping` returns "context deadline exceeded" in infinite loop
- `wsl -l -v` hangs when docker-desktop distro is mid-boot

**Root Cause:** WSL2 Hyper-V VM layer deadlocked. The `docker-desktop` WSL distro boots but its init process never completes. vmcompute service restart alone doesn't clear it.

**What Did NOT Fix It:**
1. Killing all Docker + WSL processes + `wsl --shutdown`
2. `Restart-Service vmcompute` via gsudo (WSL responds but Docker still hangs)
3. `wsl --update` (fails: "WslService could not be stopped" even elevated)
4. Launching Docker Desktop as admin (`-Verb RunAs`)
5. Multiple full kill-restart cycles

**What Fixes It:**
- Full Windows reboot (clears Hyper-V/WSL stack completely)

**Contributing Factor:** Container `wyoming-kokoro` was crash-looping (exitCode:1, restartCount:6) prior to the deadlock. May have triggered the VM hang during restart attempts.

**Prevention:**
- Set crash-looping containers to `--restart=no` or `--restart=on-failure:3`
- Keep WSL updated: `gsudo wsl --update`
- UAC must be set to "Never notify" (not fully disabled) for gsudo elevation to work. Registry keys: `EnableLUA=1`, `ConsentPromptBehaviorAdmin=0`

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
