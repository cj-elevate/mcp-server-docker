---
type: handoff
created: 2026-01-18T09:55:00Z
updated: 2026-01-19T22:55:00Z
status: in_progress
project: docker-mcp
phase: Phase 1 of 6
summary: Add 6 new Docker tools (pause, unpause, restart, tag, inspect)
next_action: Implement Phase 1 schemas with validation fixes (ge=0, min_length=1)
branch: main
task_type: enhance
---

# Handoff: Docker MCP Tool Additions

## Context

Adding 6 new tools to the docker-mcp Python MCP server (fork of ckreiling/mcp-server-docker).

**Task Plan:** `D:\servers\docker-mcp\docs\task_plan_tool_additions.md`

## Current State

| Item | Status |
|------|--------|
| Task plan | Complete |
| /team #deep review | Complete (Codex, Gemini, Perplexity) |
| Phase 1: Input Schemas | Not started |
| Phase 2: Output Helpers | Not started |
| Phase 3: Tool Registration | Not started |
| Phase 4: Tool Handlers | Not started |
| Phase 5: Testing | Not started |
| Phase 6: Documentation | Not started |

## Tools to Add

| Tool | Schema | Notes |
|------|--------|-------|
| `pause_container` | Reuse `ContainerActionInput` | Simple |
| `unpause_container` | Reuse `ContainerActionInput` | Simple |
| `restart_container` | New `RestartContainerInput` | Add `timeout: int = 10` |
| `tag_image` | New `TagImageInput` | `image`, `repository`, `tag` |
| `inspect_network` | New `InspectNetworkInput` | Add `include_attrs` flag |
| `inspect_volume` | New `InspectVolumeInput` | Add `include_attrs` flag |

## Key Decisions (from /team reviews)

1. **Follow existing pattern** - Pydantic schemas + elif branches
2. **Use `asyncio.to_thread()`** - Docker SDK is blocking
3. **Call `container.reload()`** - After mutations for fresh state
4. **Curated inspect output** - Use helpers, `include_attrs=False` default
5. **Docker-specific exceptions** - Catch `NotFound`, `APIError`, `ImageNotFound`
6. **10s restart timeout** - Matches Docker CLI default
7. **Input validation** (2026-01-19) - `ge=0` for timeout, `min_length=1` for strings
8. **Idempotent pause/unpause** (2026-01-19) - Return success if already in target state

## Key Files

| File | Purpose |
|------|---------|
| `src/mcp_server_docker/input_schemas.py` | Add 4 new Pydantic schemas |
| `src/mcp_server_docker/output_schemas.py` | Add 2 inspect helpers |
| `src/mcp_server_docker/server.py` | Register tools, add handlers |
| `docs/task_plan_tool_additions.md` | Full implementation plan |

## Next Action

Start Phase 1: Add input schemas to `input_schemas.py`

```python
# Add these new schemas (with validation fixes from /team review):
class RestartContainerInput(JSONParsingModel):
    container_id: str = Field(..., min_length=1, description="Container ID or name")
    timeout: int = Field(10, ge=0, description="Seconds to wait before killing")

class TagImageInput(JSONParsingModel):
    image: str = Field(..., min_length=1, description="Image ID or name:tag")
    repository: str = Field(..., min_length=1, description="Repository for new tag")
    tag: str = Field("latest", min_length=1, description="Tag name")

class InspectNetworkInput(JSONParsingModel):
    network_id: str = Field(..., min_length=1, description="Network ID or name")
    include_attrs: bool = Field(False, description="Include full raw attributes")

class InspectVolumeInput(JSONParsingModel):
    volume_name: str = Field(..., min_length=1, description="Volume name")
    include_attrs: bool = Field(False, description="Include full raw attributes")
```

## Verification

After implementation, verify with:
```
restart_backend("docker-mcp")
search_tools("pause")  # Should find pause_container
search_tools("inspect")  # Should find inspect_network, inspect_volume
```

## Research Reference

Team review saved at: `D:\workspace\docs\research\20260118_095221_MCP_Model_Context_Protocol_server_b.json`

---

Resume with: `/start docker-mcp continue Phase 1`
