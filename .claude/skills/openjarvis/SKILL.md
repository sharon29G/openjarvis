---
name: openjarvis
description: Use and develop OpenJarvis (local-first personal AI framework, `jarvis` CLI) in this repository. Use when the user wants to run jarvis commands (ask, chat, skill, agents, registry, doctor, serve, eval), install or manage OpenJarvis skills, inspect agents/tools/engines, or build, test and lint the OpenJarvis codebase.
---

# OpenJarvis

OpenJarvis is a Python (>=3.11) framework for local-first personal AI agents,
with an optional Rust extension. The CLI entry point is `jarvis`
(`openjarvis.cli:main`); source lives in `src/openjarvis/`.

## Setup

Run from the repository root:

```bash
uv sync                    # runtime deps (add --extra dev for tests/linting)
uv run jarvis --help       # always prefix commands with `uv run`
```

If `.venv` is missing or imports fail, run `uv sync` again before anything else.

## Common CLI commands

| Goal | Command |
|---|---|
| Health check | `uv run jarvis doctor` |
| One-shot question | `uv run jarvis ask "..." [-a agent] [-e engine] [-m model] [--tools a,b] [--json]` |
| Interactive chat | `uv run jarvis chat` (interactive, avoid in non-TTY sessions) |
| Show config | `uv run jarvis config --help` |
| List registries / components | `uv run jarvis registry list`, `uv run jarvis registry show <name>` |
| Tools | `uv run jarvis tool --help` |
| Skills | `uv run jarvis skill list|search|info|install|remove|sync|sources` |
| Install skill from a source | `uv run jarvis skill install hermes:arxiv` |
| OpenAI-compatible server | `uv run jarvis serve` |
| Presets | `uv run jarvis init --preset <chat-simple|code-assistant|deep-research|scheduled-monitor|morning-digest-linux> --force` |

Built-in agents: `simple`, `orchestrator`, `native_react`, `native_openhands`,
`deep_research`, `operative`, `monitor_operative`, `morning_digest`.

### Inference engines

`jarvis ask`/`chat` need a model backend. Locally that is usually Ollama
(`src/openjarvis/engine/ollama.py`); cloud and OpenAI-compatible engines live in
`src/openjarvis/engine/` (`cloud.py`, `litellm.py`, `openai_compat_engines.py`).
In a cloud container without Ollama or API keys, inference commands will fail:
say so and fall back to non-inference commands (`doctor`, `registry`, `skill`,
`config`, `tool`) or to tests. Never invent model output.

## Development

Mirrors CI (`Makefile`, `.github/workflows/ci.yml`):

```bash
uv sync --extra dev
uv run pytest tests/ -n auto -q --tb=short -m "not live and not cloud and not hub"
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
```

- Run a focused subset first, e.g. `uv run pytest tests/skills -q`.
- The Rust extension (`make build`, needs `maturin` + cargo) is optional; most
  Python tests run without it.
- Read `CONTRIBUTING.md` and `REVIEW.md` before opening PRs.

## Codebase map

- `src/openjarvis/cli/` – click commands for `jarvis`
- `agents/`, `tools/`, `engine/`, `memory/`, `skills/`, `connectors/`,
  `channels/`, `scheduler/`, `server/`, `evals/`, `learning/` – core subsystems
- `skills/` – agentskills.io-compatible skill loader/importer (Hermes, OpenClaw, GitHub sources)
- `configs/` – presets and default configs; `docs/` – mkdocs site; `tests/` – pytest suite
