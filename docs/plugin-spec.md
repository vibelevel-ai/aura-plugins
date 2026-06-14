# Plugin spec

The contract every Aura plugin implements. Keep it simple: **read locally, redact, send a
small packet.**

## Lifecycle

```
1. LOCATE   find the agent's local session store
2. PARSE    split into individual sessions
3. REDACT   truncate excerpts, strip secrets, paths-only (mandatory)
4. ENRICH   add measured telemetry where the tool exposes it
5. SUBMIT   POST to the Aura MCP with the user's PAT
```

## The Aura MCP endpoint & auth

- **Endpoint:** the user's MCP URL (shown in their Aura "Connect your agent" card), e.g.
  `https://aura-mcp.vibelevel.ai/mcp`. It speaks **MCP over streamable HTTP**.
- **Auth:** `Authorization: Bearer aura_…` (the user's Personal Access Token). Read it from
  the user's config/env; never log or commit it.
- **Tools to call:**
  - `score_this_session(evidence)` — one session, returns the score.
  - `import_history(sessions)` — a list of packets (chunk it; see below).

You can talk to the server with any MCP client library, or speak the JSON-RPC directly.

## The evidence packet

The single shape both tools accept. Only `source` is required; everything else is optional
but **more complete = more accurate**.

```jsonc
{
  "source": "claude_code | cursor | codex | windsurf | gemini_cli | vscode | claude_desktop | claude_ai | chatgpt | perplexity | web",
  "modality": "coding | noncoding",     // optional; server classifies if omitted
  "title": "short human label",
  "started_at": "ISO-8601",
  "ended_at": "ISO-8601",
  "model": "primary model used",
  "task_type": "debugging | architecture | feature_build | code_review | writing | research | infrastructure | other",

  "turns": [
    { "role": "user | assistant | tool",
      "text_excerpt": "TRUNCATED / redacted snippet — NOT the full message",
      "ts": "ISO-8601", "token_est": 123, "tool_name": "Edit" }
  ],

  "files_touched": [
    { "path": "relative/or/basename", "lang": "python", "bytes": 1024, "ops": "edited" }
  ],

  "local_stats": {
    "tokens": { "total": 1250000, "human": 8000, "measured": true },
    "tools_used": { "Edit": 42, "Read": 30 },
    "subagent_count": 12,
    "skills_used": ["/code-review"],
    "plan_mode": false
  }
}
```

### Completeness matters

`turns` is the biggest driver of the score. Include **every user prompt** plus the
assistant/tool turns that show real iteration — truncate each excerpt for privacy, but do
**not** cap the *number* of turns to keep the packet small. A long, well-steered session
must arrive as a long packet, or it reads as passive and scores too low.

## Redaction rules

Mandatory. A plugin must, before sending:

- **Truncate** every `text_excerpt` — never the full message.
- **Strip** secrets/tokens/keys/credentials and obvious PII.
- **Abstract** proprietary names and client/customer identities in any free text.
- Send `files_touched` as **paths + metadata only** — never contents or diffs.

Ship a self-check that asserts the output packet contains no file contents and no secret
patterns.

## Telemetry & provenance

Tag token stats honestly:

- `measured: true` — only when the value comes from the tool's **real** telemetry
  (e.g. parsed per-turn usage, or a `/context` / `/usage` reading).
- `measured: false` (estimated) — derived from session volume. Estimates are display-only
  and never affect the score, so never promote an estimate to `measured`.

Behavioral counts (prompts, tool calls, sub-agents) should be **counted from the logs**,
not estimated — they're the reliable scoring basis.

## Idempotency

The server deduplicates by a per-session fingerprint, so:

- **Re-submitting is safe** — already-scored sessions are skipped.
- For `import_history`, send in **chunks** (~10–20 sessions per call). If a call fails,
  just resend that chunk; dedup prevents double-counting. There's no server-side queue —
  your retry is what resumes the import.

## Submitting

- One fresh session → `score_this_session`.
- A backlog / history import → `import_history`, chunked, resubmit-safe.
- Default to a **preview/confirm** before the first upload so the user sees exactly what's
  sent.
