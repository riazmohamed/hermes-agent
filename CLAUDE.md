# Hermes Agent — Claude Code Guide

## Project overview

Hermes Agent is a self-improving AI agent by Nous Research. It creates skills from experience, improves them during use, and runs on local/cloud/serverless backends. Supports 200+ models via OpenRouter and others.

Current version: **0.11.0**

## Install & run

```bash
# Development install (editable)
uv pip install -e ".[dev]"

# Run the agent
python run_agent.py
hermes          # after global install
```

> `pip3` is broken on Python 3.14 (dylib issue with expat). Always use `uv` or `python3.12`/`python3.13`.

## Key entry points

| File | Purpose |
|------|---------|
| `cli.py` | Interactive CLI / TUI |
| `run_agent.py` | Main agent runner |
| `agent/` | Core agent logic (adapters, models, memory) |
| `tools/` | Built-in tools |
| `skills/` | Bundled skills |
| `hermes/` | `hermes` CLI subcommands |
| `gateway/` | Messaging gateway (Telegram, Discord, Slack…) |
| `cron/` | Built-in cron scheduler |
| `tui_gateway/` | TUI backend |

## Architecture

- **Adapters** (`agent/*_adapter.py`): per-provider API wrappers (Anthropic, Gemini, OpenAI-compat, Moonshot…)
- **Credential pool** (`agent/credential_pool.py`, `agent/credential_sources.py`): multi-key rotation
- **Skills system**: agent creates/improves skills; bundled skills live in `skills/`, optional ones in `optional-skills/`
- **Toolsets** (`toolsets.py`, `toolset_distributions.py`): group tools by capability, configurable per session
- **Context compressor** (`agent/context_compressor.py`): keeps context within model limits
- **ACP adapter** (`acp_adapter/`): Agent Communication Protocol support

## Dev conventions

- Bug fixes > cross-platform compat > security > perf > new skills > new tools
- New capabilities almost always belong as **skills**, not tools — see `CONTRIBUTING.md`
- Tests: `pytest tests/` — hits real integrations, no mocks for DB/API tests
- Linting: `ruff check .` / `ruff format .`
- Type checking: `ty check` (via `ty` package)

## Branches

- `main` — stable trunk
- `study` — study guide docs (Hermes-Agent-Study-Guide.pdf, hermes-study-guide.html)

## Build notes

- `pyproject.toml` extras: `[dev]`, `[all]`, `[modal]`, `[daytona]`, `[messaging]`, `[voice]`, `[tts-premium]`, `[cron]`, `[slack]`, `[matrix]`, `[cli]`
- `package.json` installs Node browser tools (`@askjo/camofox-browser`, `agent-browser`) — requires Node ≥ 20
- `uv.lock` is the authoritative lock file

## Study branch extras

- `hermes-study-guide.html` — 10-section HTML guide with Mermaid architecture diagrams
- `Hermes-Agent-Study-Guide.pdf` — PDF export of the study guide
