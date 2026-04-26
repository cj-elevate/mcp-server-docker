---
type: project
area: mcp-servers
path: D:\servers\docker-mcp
status: deprecated
updated: 2026-04-26
tags: [docker, containers, images, compose, mcp, deprecated]
---

# Docker MCP (Python) [DEPRECATED]

**Deprecated 2026-04-26.** Removed from master-mcp-proxy active tool graph.
Docker operations now use CLI via Bash. Code preserved for reference and
possible future narrow MCP rebuild (3-5 intent-based tools).

Python-based MCP server providing 25 Docker management tools via natural language.

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

## Historical Patterns (no longer active)

This server was previously accessible via `enable_scopes(["docker"])` through
master-mcp-proxy. That scope and backend registration were removed 2026-04-26.
Use Docker CLI via Bash for all container operations.

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
