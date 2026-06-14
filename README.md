# Aura Plugins (Community)

**Local collectors that gather richer session telemetry for your AI agent and send a
redacted summary to [VibeLevel Aura](https://github.com/vibelevel-ai/aura).**

> **Status: early / pre-1.0 — seeking contributors.** The Aura MCP already lets any agent
> score a session with no install. Plugins make that *better and automatic* for a specific
> tool: they read that tool's own logs, gather **measured** telemetry (real token usage,
> tool counts, sub-agents, lifecycle events), redact it, and call the Aura MCP for you.
> We'd love the community to build one per agent.

---

## Why plugins?

Out of the box, your agent can score a session by summarizing it itself — that's enough to
get a profile. But each tool stores richer signals locally (exact token counts, every tool
call, sub-agent activity, plan-mode, deploy/test events). A plugin taps those for:

- **`measured` token stats** instead of estimates (more accurate, and unlocks token-aware
  scoring — see the [Aura docs](https://github.com/vibelevel-ai/aura)).
- **Automatic** capture (e.g. on session end) instead of asking each time.
- **Complete** evidence — every prompt and tool call, which is the single biggest driver
  of an accurate score.

A plugin never changes the trust model: **raw code and prompts still never leave the
machine.** It only sends the same redacted [evidence packet](docs/plugin-spec.md#the-evidence-packet)
the MCP already accepts.

---

## How a plugin works

```
  your agent's local logs          plugin (runs locally)                Aura MCP
  ┌────────────────────┐    read   ┌─────────────────────┐   HTTPS    ┌──────────────┐
  │ ~/.claude / cursor │ ────────▶ │ 1. parse sessions   │  + Bearer  │ score_this_  │
  │  / codex / … logs  │           │ 2. REDACT           │ ─────────▶ │  session  or │
  │                    │           │ 3. gather telemetry │  evidence  │ import_      │
  └────────────────────┘           │ 4. build packet     │   packet   │  history     │
                                   └─────────────────────┘            └──────────────┘
                                   raw never leaves the box
```

The plugin authenticates with the user's **Personal Access Token** (`aura_…`) and posts to
the Aura MCP endpoint. See [docs/plugin-spec.md](docs/plugin-spec.md) for the full contract.

---

## Target agents & status

Help wanted on all of these. Pick one and open a draft PR.

| Agent | Local telemetry available | Status | Notes |
|---|---|---|---|
| **Claude Code** | transcripts (JSONL), real token usage, tool calls, sub-agents, hooks | 🟡 reference WIP | Richest source; good first plugin. See [agents reference](docs/agents/README.md). |
| **Cursor** | session logs, model, tool usage | 🔴 needs owner | |
| **Codex** | `/status`, logs | 🔴 needs owner | |
| **Windsurf** | logs | 🔴 needs owner | |
| **Gemini CLI** | logs | 🔴 needs owner | |
| **VS Code (MCP / Copilot)** | varies | 🔴 needs owner | |
| **Claude Desktop** | limited local telemetry | 🔴 needs owner | |
| **ChatGPT / Perplexity / Claude.ai** | connector-based, limited local data | 🔵 exploring | |

Legend: 🟢 maintained · 🟡 in progress · 🔴 needs an owner · 🔵 exploring

---

## Get involved

- Read **[CONTRIBUTING.md](CONTRIBUTING.md)** and the **[plugin spec](docs/plugin-spec.md)**.
- Start from the **[plugin template](plugins/_template/)**.
- Check the **[per-agent reference](docs/agents/README.md)** for where each tool keeps its
  data and what telemetry is available.
- Questions / coordination: **[Discord](https://REPLACE-ME-discord-invite)**.

By contributing you agree to the [Code of Conduct](CODE_OF_CONDUCT.md).

*License: TBD — a permissive license (MIT or Apache-2.0) is the likely choice; it must be
finalized before merging third-party contributions. See [CONTRIBUTING.md](CONTRIBUTING.md#licensing).*
