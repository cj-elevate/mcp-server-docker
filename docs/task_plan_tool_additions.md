---
type: doc
doc: task_plan
project: docker-mcp
created: 2026-01-18
updated: 2026-01-19
status: active
priority: P1
---

# Task Plan: Docker MCP Tool Additions

Add 6 new tools to the docker-mcp server following established patterns with improvements from /team #deep review (Codex, Gemini, Perplexity).

## Overview

| Tool | Category | Priority | Complexity |
|------|----------|----------|------------|
| `pause_container` | Container | High | Low |
| `unpause_container` | Container | High | Low |
| `restart_container` | Container | High | Medium |
| `tag_image` | Image | Medium | Low |
| `inspect_network` | Network | Medium | Medium |
| `inspect_volume` | Volume | Medium | Medium |

## Architecture Decisions (from /team #deep)

### 1. Follow Existing Pattern
- Pydantic schemas in `input_schemas.py`
- Tool registration in `list_tools()`
- Handler branches in `call_tool()`
- Output via `docker_to_dict()` helper

### 2. Error Handling Improvements
Add Docker-specific exception handling:
```python
from docker.errors import NotFound, APIError, ImageNotFound

try:
    # tool logic
except NotFound as e:
    return [types.TextContent(type="text", text=f"ERROR: Resource not found: {e}")]
except APIError as e:
    return [types.TextContent(type="text", text=f"ERROR: Docker API error: {e}")]
```

### 3. Async Threading for Blocking Ops
Docker SDK is synchronous. Wrap blocking calls:
```python
import asyncio
result = await asyncio.to_thread(container.restart, timeout=timeout)
```

### 4. State Refresh After Mutations
Call `container.reload()` before `docker_to_dict()` for accurate status:
```python
container.restart(timeout=timeout)
container.reload()  # Refresh state
result = docker_to_dict(container)
```

### 5. Inspect Tools: Curated + Optional Full
Return summarized data by default, add `include_attrs: bool = False` for full raw data.

---

## Phase 1: Input Schemas (input_schemas.py)

**Files:** `src/mcp_server_docker/input_schemas.py`

### 1.1 Reuse Existing
- `pause_container` → use `ContainerActionInput`
- `unpause_container` → use `ContainerActionInput`

### 1.2 New Schemas (Updated per Team Review 2026-01-19)

```python
class RestartContainerInput(JSONParsingModel):
    """Restart a container with optional timeout."""
    container_id: str = Field(..., min_length=1, description="Container ID or name")
    timeout: int = Field(10, ge=0, description="Seconds to wait before killing")


class TagImageInput(JSONParsingModel):
    """Tag an image with a new repository/tag."""
    image: str = Field(..., min_length=1, description="Image ID, name, or name:tag to tag")
    repository: str = Field(..., min_length=1, description="Repository name for the new tag")
    tag: str = Field("latest", min_length=1, description="Tag name")


class InspectNetworkInput(JSONParsingModel):
    """Inspect a Docker network."""
    network_id: str = Field(..., min_length=1, description="Network ID or name")
    include_attrs: bool = Field(False, description="Include full raw attributes")


class InspectVolumeInput(JSONParsingModel):
    """Inspect a Docker volume."""
    volume_name: str = Field(..., min_length=1, description="Volume name")
    include_attrs: bool = Field(False, description="Include full raw attributes")
```

**Validation Fixes (from /team review):**
- `timeout: ge=0` - Negative timeouts invalid for Docker
- `min_length=1` on all string fields - Docker rejects empty strings
- Clear descriptions without "(default: X)" redundancy

**Acceptance Criteria:**
- [ ] All schemas inherit from `JSONParsingModel`
- [ ] Field descriptions are clear for LLM consumption
- [ ] Defaults match Docker CLI conventions
- [ ] Input validation prevents invalid Docker API calls

---

## Phase 2: Output Helpers (output_schemas.py)

**Files:** `src/mcp_server_docker/output_schemas.py`

### 2.1 Add Inspect Output Helpers

```python
def network_inspect_to_dict(network: Network, include_attrs: bool = False) -> dict[str, Any]:
    """Extended network info for inspect operations."""
    base = docker_to_dict(network)
    base.update({
        "ipam": network.attrs.get("IPAM"),
        "options": network.attrs.get("Options"),
        "containers": list(network.attrs.get("Containers", {}).keys()),
        "internal": network.attrs.get("Internal", False),
        "attachable": network.attrs.get("Attachable", False),
    })
    if include_attrs:
        base["attrs"] = network.attrs
    return base


def volume_inspect_to_dict(volume: Volume, include_attrs: bool = False) -> dict[str, Any]:
    """Extended volume info for inspect operations."""
    base = docker_to_dict(volume)
    base.update({
        "options": volume.attrs.get("Options"),
        "status": volume.attrs.get("Status"),
    })
    if include_attrs:
        base["attrs"] = volume.attrs
    return base
```

**Acceptance Criteria:**
- [ ] Helpers return curated data by default
- [ ] `include_attrs=True` adds full raw `attrs`
- [ ] No sensitive data leaked in default output

---

## Phase 3: Tool Registration (server.py - list_tools)

**Files:** `src/mcp_server_docker/server.py`

Add to `list_tools()`:

```python
types.Tool(
    name="pause_container",
    description="Pause a running Docker container",
    inputSchema=ContainerActionInput.model_json_schema(),
),
types.Tool(
    name="unpause_container",
    description="Unpause a paused Docker container",
    inputSchema=ContainerActionInput.model_json_schema(),
),
types.Tool(
    name="restart_container",
    description="Restart a Docker container",
    inputSchema=RestartContainerInput.model_json_schema(),
),
types.Tool(
    name="tag_image",
    description="Tag an image with a new repository and tag",
    inputSchema=TagImageInput.model_json_schema(),
),
types.Tool(
    name="inspect_network",
    description="Get detailed information about a Docker network",
    inputSchema=InspectNetworkInput.model_json_schema(),
),
types.Tool(
    name="inspect_volume",
    description="Get detailed information about a Docker volume",
    inputSchema=InspectVolumeInput.model_json_schema(),
),
```

**Acceptance Criteria:**
- [ ] All 6 tools registered
- [ ] Descriptions are clear and actionable
- [ ] Tool count increases from 19 to 25

---

## Phase 4: Tool Handlers (server.py - call_tool)

**Files:** `src/mcp_server_docker/server.py`

### 4.1 Import Updates
```python
import asyncio
from docker.errors import NotFound, APIError, ImageNotFound

from .input_schemas import (
    # ... existing imports ...
    RestartContainerInput,
    TagImageInput,
    InspectNetworkInput,
    InspectVolumeInput,
)
from .output_schemas import (
    docker_to_dict,
    network_inspect_to_dict,
    volume_inspect_to_dict,
)
```

### 4.2 Handler Implementations (Updated per Team Review 2026-01-19)

**Idempotency Handling (Gemini recommendation):** Make pause/unpause idempotent for LLM-friendly behavior.

```python
elif name == "pause_container":
    args = ContainerActionInput(**arguments)
    container = _docker.containers.get(args.container_id)
    if container.status == "paused":
        result = {"status": "already_paused", "message": "Container was already paused", **docker_to_dict(container)}
    else:
        await asyncio.to_thread(container.pause)
        container.reload()
        result = docker_to_dict(container)

elif name == "unpause_container":
    args = ContainerActionInput(**arguments)
    container = _docker.containers.get(args.container_id)
    if container.status != "paused":
        result = {"status": "already_running", "message": "Container was not paused", **docker_to_dict(container)}
    else:
        await asyncio.to_thread(container.unpause)
        container.reload()
        result = docker_to_dict(container)

elif name == "restart_container":
    args = RestartContainerInput(**arguments)
    container = _docker.containers.get(args.container_id)
    await asyncio.to_thread(container.restart, timeout=args.timeout)
    container.reload()
    result = docker_to_dict(container)

elif name == "tag_image":
    args = TagImageInput(**arguments)
    image = _docker.images.get(args.image)
    await asyncio.to_thread(image.tag, args.repository, tag=args.tag)
    result = {
        "status": "tagged",
        "image": args.image,
        "new_tag": f"{args.repository}:{args.tag}",
    }

elif name == "inspect_network":
    args = InspectNetworkInput(**arguments)
    network = _docker.networks.get(args.network_id)
    result = network_inspect_to_dict(network, include_attrs=args.include_attrs)

elif name == "inspect_volume":
    args = InspectVolumeInput(**arguments)
    volume = _docker.volumes.get(args.volume_name)
    result = volume_inspect_to_dict(volume, include_attrs=args.include_attrs)
```

### 4.3 Enhanced Error Handling

Update the try/except block:
```python
except NotFound as e:
    return [types.TextContent(type="text", text=f"ERROR: Resource not found: {e}")]
except APIError as e:
    await app.request_context.session.send_log_message("error", str(e))
    return [types.TextContent(type="text", text=f"ERROR: Docker API error: {e}")]
except ImageNotFound as e:
    return [types.TextContent(type="text", text=f"ERROR: Image not found: {e}")]
except ValidationError as e:
    # ... existing handler ...
```

### 4.4 Edge Cases (from Team Review)

| Operation | Edge Case | Handling |
|-----------|-----------|----------|
| `pause_container` | Already paused | Return success: "Container was already paused" |
| `unpause_container` | Already running | Return success: "Container was not paused" |
| `restart_container` | Container in "dead"/"removing" state | Let Docker API error propagate with clear message |
| `tag_image` | Image not found | Caught by `ImageNotFound` exception |

**Acceptance Criteria:**
- [ ] All handlers use `asyncio.to_thread()` for blocking ops
- [ ] State mutations call `container.reload()` before output
- [ ] Docker-specific exceptions caught and returned as text
- [ ] No exceptions bubble up to crash server
- [ ] Pause/unpause are idempotent (LLM-friendly)

---

## Phase 5: Testing & Verification

### 5.1 Restart MCP Backend
```bash
# Via master-mcp-proxy
restart_backend("docker-mcp")
```

### 5.2 Verify Tool Discovery
```bash
search_tools("pause")
search_tools("restart")
search_tools("tag")
search_tools("inspect")
```

### 5.3 Functional Tests

| Test | Command | Expected |
|------|---------|----------|
| Pause | `pause_container(container_id="ollama")` | Status: paused |
| Unpause | `unpause_container(container_id="ollama")` | Status: running |
| Restart | `restart_container(container_id="ollama", timeout=5)` | Status: running |
| Tag | `tag_image(image="alpine", repository="test/alpine", tag="v1")` | Tagged |
| Inspect Net | `inspect_network(network_id="bridge")` | Network details |
| Inspect Vol | `inspect_volume(volume_name="aistack_open-webui")` | Volume details |

### 5.4 Error Tests

| Test | Command | Expected |
|------|---------|----------|
| Not Found | `pause_container(container_id="nonexistent")` | ERROR: Resource not found |
| Invalid | `restart_container(container_id="")` | ERROR: Invalid inputs |

**Acceptance Criteria:**
- [ ] All 6 tools discoverable via `search_tools()`
- [ ] All functional tests pass
- [ ] Error tests return friendly messages (not stack traces)

---

## Phase 6: Documentation Updates

### 6.1 Update CLAUDE.md
- Add new tools to tool list
- Update tool count (19 → 25)
- Remove items from "Missing from Ruby Version" checklist

### 6.2 Update CHANGELOG.md
```markdown
## [2026-01-XX] - Tool Additions

### Added
- `pause_container` - Pause a running container
- `unpause_container` - Unpause a paused container
- `restart_container` - Restart container with timeout
- `tag_image` - Tag an image with new repository/tag
- `inspect_network` - Detailed network information
- `inspect_volume` - Detailed volume information
- Docker-specific error handling (NotFound, APIError)
- Async threading for blocking Docker SDK calls
```

### 6.3 Update README.md
- Add new tools to API documentation
- Update examples

**Acceptance Criteria:**
- [ ] CLAUDE.md reflects new capabilities
- [ ] CHANGELOG.md has dated entry
- [ ] README.md documents all 25 tools

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Breaking existing tools | Low | High | Don't modify existing handler code |
| Docker SDK blocking | Medium | Medium | Use `asyncio.to_thread()` |
| State staleness | Medium | Low | Call `reload()` after mutations |
| Security: env var leak | Low | Medium | Curated output by default |

---

## Success Criteria

- [ ] Tool count: 19 → 25
- [ ] All new tools pass functional tests
- [ ] No regressions in existing tools
- [ ] Error handling improved (Docker-specific exceptions)
- [ ] Documentation updated

---

## Notes

- **Initial Review:** Codex, Gemini, Perplexity via /team #deep (2026-01-18)
- **Plan Update Review:** Codex, Gemini, Perplexity via /team (2026-01-19)
- **Key insight:** Use `asyncio.to_thread()` to prevent blocking
- **Validation insight:** Add `ge=0` for timeout, `min_length=1` for strings
- **Idempotency insight:** Make pause/unpause return success if already in target state
- **Future:** Consider decorator-based tool registry to reduce elif chain

## Team Review References

| Date | Agents | Findings | Saved To |
|------|--------|----------|----------|
| 2026-01-18 | Codex, Gemini, Perplexity | Initial architecture review | `D:\workspace\docs\research\20260118_095221_MCP_Model_Context_Protocol_server_b.json` |
| 2026-01-19 | Codex, Gemini, Perplexity | Schema validation fixes | `D:\workspace\docs\research\20260119_174852_Python_Pydantic_best_practices_2025.json` |
