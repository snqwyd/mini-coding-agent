# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Commands

```bash
# Lint
python -m ruff check .          # or: uv run python -m ruff check .

# Run all tests
python -m pytest -q              # or: uv run python -m pytest -q

# Run a single test
python -m pytest -q tests/test_mini_coding_agent.py::test_agent_runs_tool_then_final

# Install for development
uv sync --group dev              # or: pip install -e . pytest ruff

# Run the agent (requires Ollama running locally)
uv run mini-coding-agent         # or: python mini_coding_agent.py
```

## Architecture

The entire agent lives in a single file: `mini_coding_agent.py`. There are no external runtime dependencies — only the Python standard library. The Ollama model backend is reached via `urllib` HTTP calls to `/api/generate`.

The code is organized around six core components, each noted with section comments in the source:

1. **Live Repo Context** (`WorkspaceContext`) — Collects workspace facts at startup: cwd, repo root, git branch/status/commits, and project docs (AGENTS.md, README.md, pyproject.toml, package.json). Gathers everything via `git` subprocess calls.

2. **Prompt Shape and Cache Reuse** (`build_prefix`, `memory_text`, `prompt`) — The prompt is assembled as: stable prefix (system prompt, rules, tool definitions, workspace context) + volatile parts (memory, transcript, user request). The prefix is computed once so repeated model calls can reuse it.

3. **Structured Tools, Validation, and Permissions** (`build_tools`, `run_tool`, `validate_tool`, `approve`, `parse`) — Seven tools: `list_files`, `read_file`, `search`, `run_shell`, `write_file`, `patch_file`, `delegate`. Risky tools (shell, write, patch) require approval. The model emits tool calls as `<tool>{"name":..., "args":{...}}</tool>` or XML-style `<tool name="..." path="..."><content>...</content></tool>`. Parsing handles both formats.

4. **Context Reduction and Output Management** (`clip`, `history_text`) — Long tool output is clipped to `MAX_TOOL_OUTPUT` (4000 chars). History is compressed: older entries get tighter limits, repeated `read_file` calls on the same path are deduplicated unless a `write_file` or `patch_file` on that path occurred in between. Total history is capped at `MAX_HISTORY` (12000 chars).

5. **Transcripts, Memory, and Resumption** (`SessionStore`, `record`, `note_tool`, `ask`, `reset`) — Sessions are JSON files stored in `.mini-coding-agent/sessions/` under the repo root. Each session has a `history` array (full transcript) and a `memory` dict (task summary, tracked files, notes). Sessions can be resumed via `--resume`.

6. **Delegation and Bounded Subagents** (`tool_delegate`) — The `delegate` tool creates a child `MiniAgent` with `read_only=True`, `approval_policy="never"`, and incremented `depth`. Delegation is bounded by `max_depth` (default 1), so children cannot delegate further.

### Key classes

- `MiniAgent` — The main agent class. Holds the model client, workspace, session, tools, and prompt prefix.
- `OllamaModelClient` — Sends prompts to Ollama's `/api/generate` endpoint.
- `FakeModelClient` — Pre-scripted model outputs used in tests (no real LLM calls).
- `SessionStore` — Persists session JSON to disk.
- `WorkspaceContext` — Immutable snapshot of the workspace state.

### Testing approach

Tests in `tests/test_mini_coding_agent.py` use `FakeModelClient` to avoid requiring a running Ollama instance. The `build_agent` helper constructs a `MiniAgent` with a temp workspace and fake model outputs. Tests verify tool execution, parsing, session persistence, delegation, path validation, history deduplication, and prompt formatting.
