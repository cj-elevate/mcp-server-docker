---
type: project
area: mcp-servers
path: D:\servers\docker-mcp
status: active
updated: 2026-01-17
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

## Gotchas

1. **Docker Desktop must be running** - Tools fail silently if daemon unavailable
2. **Windows socket path** - Uses named pipe `//./pipe/docker_engine`
3. **No exec_container** - Intentionally excluded for security (use shell MCP if needed)
4. **uv required** - Uses `uv` package manager for dependency management
5. **Python 3.12+** - Requires modern Python (installed at `C:\Python312`)

## Tools (18 Total)

### Container Management (8)
- `list_containers` - List all containers
- `create_container` - Create without starting
- `run_container` - Create and start
- `recreate_container` - Stop, remove, create, start
- `start_container` - Start stopped container
- `stop_container` - Stop running container
- `remove_container` - Delete container
- `fetch_container_logs` - Get container logs

### Image Management (5)
- `list_images` - List local images
- `pull_image` - Download from registry
- `push_image` - Upload to registry
- `build_image` - Build from Dockerfile
- `remove_image` - Delete local image

### Network Management (3)
- `list_networks` - List Docker networks
- `create_network` - Create new network
- `remove_network` - Delete network

### Volume Management (3)
- `list_volumes` - List Docker volumes
- `create_volume` - Create new volume
- `remove_volume` - Delete volume

### Prompts (1)
- `docker_compose` - Natural language container deployment with plan+apply loop

## Missing from Ruby Version

These Ruby tools are NOT included (intentionally excluded for security):
- `exec_container` - Use shell MCP server instead
- `copy_to_container` - Use volume mounts instead
- `pause_container` / `unpause_container` - Rarely used
- `restart_container` - Can do: stop + start
- `tag_image` - Can do: `docker tag` via shell
- `inspect_network` / `inspect_volume` - Use `docker inspect` via shell

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
- Tools: 18 (8 container, 5 image, 3 network, 3 volume)
- Source: https://github.com/ckreiling/mcp-server-docker

## Fork Modifications

This is a fork of `ckreiling/mcp-server-docker` with planned enhancements:
- [ ] Add `pause_container` / `unpause_container`
- [ ] Add `restart_container`
- [ ] Add `tag_image`
- [ ] Add `inspect_network` / `inspect_volume`
- [ ] Optional: `exec_container` with `confirm_dangerous` toggle
