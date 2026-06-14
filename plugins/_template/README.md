# Plugin template

Copy this folder to `plugins/<agent-name>/` and build your collector. Any language is fine;
this template just defines the expected shape and checklist.

## Suggested layout

```
plugins/<agent-name>/
├── README.md          # install, usage, what it captures, limitations
├── <collector>        # your code (script / CLI / hook)
├── tests/             # tests against SYNTHETIC fixtures only
└── fixtures/          # synthetic sample sessions (NO real user data)
```

## The five steps (see ../../docs/plugin-spec.md)

1. **Locate** the agent's local session store (see ../../docs/agents/README.md).
2. **Parse** into individual sessions.
3. **Redact** — truncate excerpts, strip secrets, paths-only. *(mandatory)*
4. **Enrich** — measured telemetry where the tool exposes it; honest provenance.
5. **Submit** — `score_this_session` (one) or `import_history` (chunked, resubmit-safe),
   with `Authorization: Bearer aura_…` read safely from config/env.

## README to write for your plugin

- **Install** — prerequisites, how to set it up (including any hook wiring).
- **Usage** — one fresh session vs. full history import; preview/consent behavior.
- **Captured signals** — which fields you populate and which are `measured` vs `estimated`.
- **Limitations** — what this tool can't expose.

## Before you open a PR

- [ ] Output matches the [evidence packet](../../docs/plugin-spec.md#the-evidence-packet).
- [ ] Redaction self-check passes (no file contents, no secrets).
- [ ] Only network call is the Aura MCP.
- [ ] PAT never logged or committed; fixtures are synthetic.
- [ ] Status table updated in the root README.

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for the full rules.
