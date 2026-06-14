# Per-agent reference

Where each agent keeps its session data and what telemetry it exposes. This is a starting
map for plugin authors — **verify paths on your own machine and OS**, and PRs that correct
or extend this are very welcome.

> General principle: most agents store transcripts as JSON/JSONL or in a local DB under the
> user's home directory or the tool's app-data folder. A plugin reads those, never the repo
> itself (unless the user explicitly opts in), and always redacts.

---

## Claude Code  🟡 (best-documented; good first plugin)

| Signal | Where / how |
|---|---|
| Transcripts | Session JSONL under `~/.claude/projects/<project>/*.jsonl` (one line per event; path is also provided to hooks). |
| Token usage | Real per-turn usage is in the JSONL; current totals via the `/context` and `/usage` slash commands. |
| Tool calls | Tool-use events in the JSONL → build `tools_used` counts. |
| Sub-agents | Sub-agent / Task events → `subagent_count`. |
| Skills / slash-commands | Invocations in the transcript → `skills_used`. |
| Plan mode | Detectable from the session events → `plan_mode`. |
| Automation hooks | `SessionEnd` (and `Stop` / `SubagentStop` / `PreToolUse`) hooks can trigger a plugin to read the just-finished transcript and submit automatically. |

Capture options: **(a)** a `SessionEnd` hook that runs the plugin on the transcript path,
or **(b)** a manual CLI run that scans `~/.claude/projects`.

---

## Codex  🔴 needs owner

| Signal | Where / how (verify) |
|---|---|
| Token usage | `/status` reading. |
| Transcripts / logs | The tool's local logs. |
| Tool calls / plan | Parse from logs where available. |

---

## Cursor  🔴 needs owner

| Signal | Where / how (verify) |
|---|---|
| Sessions | Local logs / workspace storage / SQLite. |
| Model | Available per session. |
| Tool usage | Parse where exposed. |

---

## Windsurf · Gemini CLI · VS Code (MCP / Copilot)  🔴 needs owner

These keep their own local logs; details vary by version and OS. Map the log location and
the available signals, then implement the standard lifecycle. Document what you find here.

---

## Claude Desktop · Claude.ai · ChatGPT · Perplexity  🔵 exploring

Local telemetry is limited or connector-based. A plugin may have less to work with (fewer
measured stats), but can still build a behavioral packet. Token stats will often be
`estimated` for these sources — which is fine; it just won't drive token-aware scoring.

---

## What to map for a new agent

When adding an agent, fill in:

1. **Log location** (per OS) and format (JSONL / JSON / SQLite / …).
2. **Session boundaries** — how to split the log into individual sessions.
3. **Available signals** — tokens (measured?), tool calls, sub-agents, plan mode, timing,
   model, files touched.
4. **Capture method** — hook/event-driven vs. manual scan.
5. **Redaction notes** — anything tool-specific to scrub.

Then add a row to the [status table](../../README.md#target-agents--status).
