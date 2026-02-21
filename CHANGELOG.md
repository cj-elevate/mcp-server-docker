---
type: doc
doc: changelog
updated: 2026-01-17
---

# Changelog

All notable changes to docker-mcp integration.

Format: [Keep a Changelog](https://keepachangelog.com/)

## [2026-01-17] - Python Migration

### Changed
- **COMPLETE REPLACEMENT**: Migrated from Ruby docker_mcp gem to Python mcp-server-docker
- Language: Ruby → Python 3.12
- Package manager: gem → uv
- Tool count: 22 → 18 (removed high-risk exec/copy tools)
- Source: afstanton/docker_mcp → ckreiling/mcp-server-docker

### Added
- Python 3.12 environment
- uv package manager
- `docker_compose` natural language prompt
- Windows compatibility (Ruby had 3 bugs)
- CLAUDE.md for Python version
- TROUBLESHOOTING.md for Python version

### Removed
- Ruby docker_mcp gem (uninstalled)
- `exec_container` tool (security risk)
- `copy_to_container` tool (security risk)
- `pause_container` / `unpause_container` tools
- `restart_container` tool
- `tag_image` tool
- `inspect_network` / `inspect_volume` tools

### Planned
- Add back `pause_container` / `unpause_container`
- Add back `restart_container`
- Add back `tag_image`
- Add back `inspect_network` / `inspect_volume`
- Optional: `exec_container` with `confirm_dangerous` toggle

### Archived
- Ruby documentation backed up to `D:\workspace\archive\ruby-docker-mcp\`
- Ruby bugs documented in debug session `MCP-001`

---

## [2026-01-17] - Ruby Version (Abandoned)

### Issues Found
1. **Invalid JSON Schema** - tool_forge emits empty `required: []` arrays
2. **Windows Path Corruption** - json-schema truncates paths
3. **File Operations** - mcp-ruby not Windows-compatible

### Resolution
- Abandoned Ruby implementation
- Switched to Python for Windows compatibility
- Debug session: `D:\workspace\docs\debug\sessions\MCP-001-docker_mcp-schema\`
