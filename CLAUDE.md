---
type: project
area: mcp-servers
path: D:\servers\docker-mcp
status: active
updated: 2026-01-17
tags: [docker, containers, images, compose, mcp]
---

# Docker MCP (Python)

Python-based MCP server providing 18 Docker management tools via natural language.

## Quick Commands

```bash
# Run server (stdio mode)
cd D:/servers/docker-mcp
uv run mcp-server-docker

# Run via uvx (from anywhere)
uvx mcp-server-docker

# Verify Docker
docker ps  # Must work for MCP to function

# Test with inspector
npx @modelcontextprotocol/inspector uv run mcp-server-docker
```

## Key Files

| File | Purpose |
|------|---------|
| `src/mcp_server_docker/` | Main source code |
| `pyproject.toml` | Dependencies and config |
| `CLAUDE.md` | This file - AI navigation guide |
| `README.md` | Full tool documentation |
| `TROUBLESHOOTING.md` | Issue log |

## Patterns

- **Scope-based access**: Enable via `enable_scopes(["docker"])`
- **Tool discovery**: `search_tools("docker")` after enabling
- **Natural language compose**: Use `docker_compose` prompt for LLM-driven deployments
- **Disable when done**: `disable_scopes(["docker"])` to reduce attack surface

## Programmatic Testing

| Field | Value |
|-------|-------|
| Service | STDIO via master-mcp-proxy (PM2) |
| Host | n/a (STDIO, not HTTP) |
| Auth | none (proxy-mediated) |
| Secret Source | Proxy `.env` (if backend needs secrets) |
| Secret Keys | n/a (proxy handles auth to external APIs) |

### Backend Health (via proxy)
```bash
# Verify backend is reachable through the proxy
# MCP tool: health_check(backends=["docker-mcp"])
# Or via curl to proxy health:
curl -sf http://127.0.0.1:3005/health | python -m json.tool
```

### Tool Verification
```bash
# MCP tool: search_tools("docker-mcp")
# Expected: list of tools registered by this backend
```

**Note:** This server has no standalone HTTP endpoint. All access is mediated through
the master-mcp-proxy. To test specific tools, use `execute_indexed_tool` or the
tool's hot name if available. Enable scope first: `enable_scopes(["docker"])`.

## Gotchas

1. **Docker Desktop must be running** - Tools fail silently if daemon unavailable
2. **Windows socket path** - Uses named pipe `//./pipe/docker_engine`
3. **No exec_container** - Intentionally excluded for security (use shell MCP if needed)
4. **uv required** - Uses `uv` package manager for dependency management
5. **Python 3.12+** - Requires modern Python (installed at `C:\Python312`)

## Tools (25 Total)

### Container Management (11)
- `list_containers` - List all containers
- `create_container` - Create without starting
- `run_container` - Create and start
- `recreate_container` - Stop, remove, create, start
- `start_container` - Start stopped container
- `stop_container` - Stop running container
- `remove_container` - Delete container
- `fetch_container_logs` - Get container logs
- `pause_container` - Pause a running container (idempotent)
- `unpause_container` - Unpause a paused container (idempotent)
- `restart_container` - Restart with configurable timeout

### Image Management (6)
- `list_images` - List local images
- `pull_image` - Download from registry
- `push_image` - Upload to registry
- `build_image` - Build from Dockerfile
- `remove_image` - Delete local image
- `tag_image` - Tag image with new repository/tag

### Network Management (4)
- `list_networks` - List Docker networks
- `create_network` - Create new network
- `remove_network` - Delete network
- `inspect_network` - Detailed network info

### Volume Management (4)
- `list_volumes` - List Docker volumes
- `create_volume` - Create new volume
- `remove_volume` - Delete volume
- `inspect_volume` - Detailed volume info

### Prompts (1)
- `docker_compose` - Natural language container deployment with plan+apply loop

## Missing from Ruby Version

These Ruby tools are NOT included (intentionally excluded for security):
- `exec_container` - Use shell MCP server instead
- `copy_to_container` - Use volume mounts instead

## Security

This MCP has **root-equivalent access** to Docker daemon.

**Recommendations:**
1. Only enable scope when actively needed
2. Disable immediately after use
3. Audit all operations in logs
4. Never expose to untrusted clients

## Integration

Part of master-mcp-proxy scope system:
- Backend ID: `docker`
- Scope ID: `docker`
- Tools: 25 (11 container, 6 image, 4 network, 4 volume)
- Source: https://github.com/ckreiling/mcp-server-docker

## Fork Modifications

This is a fork of `ckreiling/mcp-server-docker` with enhancements:
- [x] Add `pause_container` / `unpause_container`
- [x] Add `restart_container`
- [x] Add `tag_image`
- [x] Add `inspect_network` / `inspect_volume`
- [ ] Optional: `exec_container` with `confirm_dangerous` toggle
