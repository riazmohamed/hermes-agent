# Hermes Agent — Claude Code Guide

## Project overview

Hermes Agent is a self-improving AI agent by Nous Research. It creates skills from experience, improves them during use, and runs on local/cloud/serverless backends. Supports 200+ models via OpenRouter and others.

Current version: **0.12.0** (released 2026-04-30 — see `RELEASE_v0.12.0.md`)

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

- **Adapters** (`agent/*_adapter.py`): per-provider API wrappers (Anthropic, Gemini, OpenAI-compat, Moonshot, Bedrock, Codex Responses…)
- **Credential pool** (`agent/credential_pool.py`, `agent/credential_sources.py`): multi-key rotation
- **Skills system**: agent creates/improves skills; bundled skills live in `skills/`, optional ones in `optional-skills/`. Skill input preprocessing in `agent/skill_preprocessing.py`.
- **Toolsets** (`toolsets.py`, `toolset_distributions.py`): group tools by capability, configurable per session
- **Context compressor** (`agent/context_compressor.py`) + **context engine** (`agent/context_engine.py`): keep context within model limits
- **Image routing** (`agent/image_routing.py`): routes image inputs across multimodal-capable providers
- **Onboarding** (`agent/onboarding.py`): first-run setup flow
- **Nous rate guard** (`agent/nous_rate_guard.py`): rate-limit handling for Nous-hosted endpoints
- **ACP adapter** (`acp_adapter/`): Agent Communication Protocol support
- **Curator** (`agent/curator.py`, `hermes_cli/curator.py`): per-run reports written to `logs/curator/` (`run.json` + `REPORT.md`)
- **Runtime footer** (`gateway/runtime_footer.py`): replaced the old `gateway/builtin_hooks/boot_md.py` boot-MD hook
- **Skill usage tracking** (`tools/skill_usage.py`): records skill invocation telemetry
- **LM Studio reasoning** (`agent/lmstudio_reasoning.py`): reasoning support for LM Studio-hosted models
- **Browser connect CLI** (`hermes_cli/browser_connect.py`): `hermes` subcommand for connecting browser tools
- **Memory provider** (`agent/memory_provider.py`): pluggable memory backend abstraction (alongside `agent/memory_manager.py`)
- **Error classifier** (`agent/error_classifier.py`): centralized provider-error categorization
- **Auxiliary client** (`agent/auxiliary_client.py`): side-channel LLM calls (curation, summarization) with transport autodetection
- **Platform registry** (`gateway/platform_registry.py`): central registry for gateway platform adapters; out-of-tree platforms (Teams, IRC) live under `plugins/platforms/`
- **Signal rate limit** (`gateway/platforms/signal_rate_limit.py`): rate-limit handling for the Signal gateway adapter
- **Achievements plugin** (`plugins/hermes-achievements/`): bundled dashboard plugin tracking skill/agent achievements
- **ComfyUI skill** (`skills/creative/comfyui/`): bundled skill for ComfyUI workflow generation and execution
- **Hermes CLI** (`hermes_cli/`): split into focused subcommand modules — `_parser.py`, `relaunch.py`, `vercel_auth.py`, plus existing `auth*`, `curator`, `doctor`, `gateway`, `models`, `platforms`, `plugins*`, `profiles`, `providers`, `setup`, `status`, etc.
- **Docker**: `Dockerfile` + `docker-compose.yml` for containerized runs (runs as host user to avoid root-owned bind mounts)

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

- `pyproject.toml` extras: `[dev]`, `[all]`, `[modal]`, `[daytona]`, `[messaging]`, `[voice]`, `[tts-premium]`, `[cron]`, `[slack]`, `[matrix]`, `[cli]`, `[google]`, `[mcp]`, `[acp]`, `[honcho]`, `[homeassistant]`, `[sms]`, `[pty]`, `[mistral]`, `[bedrock]`, `[web]`, `[dingtalk]`, `[feishu]`, `[termux]`, `[rl]`, `[yc-bench]`
- `[matrix]` extra now includes `aiohttp-socks` for SOCKS proxy support
- `package.json` installs Node browser tools (`@askjo/camofox-browser`, `agent-browser`) — requires Node ≥ 20
- `uv.lock` is the authoritative lock file
- `AGENTS.md` — guidance for AI agents contributing to the repo

## Study branch extras

- `hermes-study-guide.html` — 10-section HTML guide with Mermaid architecture diagrams
- `Hermes-Agent-Study-Guide.pdf` — PDF export of the study guide

## Last sync

- 2026-04-28 — merged 745 commits from `main` into `study`; regenerated `uv.lock`; rebuilt via `uv pip install -e ".[dev]"`
- 2026-04-29 — synced fork `main` with upstream `NousResearch/hermes-agent` (143 commits) and merged into `study`; added curator/runtime-footer/skill-usage/lmstudio-reasoning/browser-connect modules; rebuilt via `uv pip install -e ".[dev]"`
- 2026-04-30 — merged 236 commits from `main` into `study` (release v0.12.0); resolved `uv.lock` conflict (took main); added platform_registry / memory_provider / error_classifier / auxiliary_client / signal_rate_limit / achievements plugin / ComfyUI skill / Teams + IRC platform plugins; rebuilt via `uv pip install -e ".[dev]"` → hermes-agent 0.11.0 → 0.12.0
