# Contributing

Thanks for helping build the Aura plugin ecosystem! This guide covers what a plugin must
do, how to add one, and the rules that keep user trust intact.

## What a plugin is

A small, **local** program (any language) that:

1. **Locates** an AI agent's local session data on the user's machine.
2. **Parses** it into individual sessions.
3. **Redacts** each session into an [evidence packet](docs/plugin-spec.md#the-evidence-packet).
4. **Gathers measured telemetry** where the tool exposes it (token usage, tool calls, etc.).
5. **Submits** packets to the Aura MCP using the user's Personal Access Token.

Full details: **[docs/plugin-spec.md](docs/plugin-spec.md)**.

## Non-negotiable rules

These protect the users whose data you're handling. A PR that breaks any of them won't be
merged:

1. **Raw content never leaves the machine.** Truncate excerpts; send file *paths* +
   metadata only, never contents. (See the [redaction rules](docs/plugin-spec.md#redaction-rules).)
2. **Strip secrets** — tokens, keys, credentials, `.env` values — before building a packet.
3. **No hidden telemetry.** A plugin sends to the Aura MCP and nowhere else. No analytics,
   no third-party calls, no "phone home."
4. **Consent first.** Default to a preview/confirm step before the first upload; make
   automatic modes explicit and opt-in.
5. **Token safety.** Read the PAT from the user's config or env — **never** log it, print
   it, or commit it. Never commit real session data or fixtures containing private content.
6. **Honest provenance.** Only tag tokens `measured` when they come from the tool's real
   telemetry. Estimates must be tagged `estimated`.

## Adding a plugin

1. Copy [`plugins/_template/`](plugins/_template/) to `plugins/<agent-name>/`.
2. Implement the five steps above for your agent. Use the
   [per-agent reference](docs/agents/README.md) for where the tool stores data.
3. Include:
   - a short `README.md` (install, usage, what telemetry it captures, limitations),
   - tests against **synthetic** session fixtures (never real user data),
   - a redaction self-check (assert no file contents / secrets in the output packet).
4. Open a **draft PR** early so we can help. Update the status table in the root
   [README](README.md#target-agents--status).

## Local development

- Get an Aura account + a Personal Access Token (`aura_…`) for end-to-end testing against
  your own data.
- Point at the MCP endpoint from your account's "Connect your agent" card.
- Validate your packet shape against the [evidence packet contract](docs/plugin-spec.md#the-evidence-packet)
  before submitting.

## PR checklist

- [ ] Follows the [evidence packet](docs/plugin-spec.md#the-evidence-packet) contract.
- [ ] Redaction self-check passes (no contents, no secrets in output).
- [ ] No network calls except the Aura MCP.
- [ ] PAT is read safely and never logged/committed.
- [ ] Tests use synthetic fixtures only.
- [ ] README documents what's captured and any limitations.
- [ ] Status table updated.

## Licensing

This project is licensed under **[Apache-2.0](LICENSE)**. By submitting a contribution, you
agree that it is licensed under Apache-2.0, and you confirm you have the right to submit it
(per the inbound-=-outbound principle in Apache-2.0 §5). No separate CLA is required; if a
DCO sign-off is adopted later, it'll be documented here.

By participating you agree to the [Code of Conduct](CODE_OF_CONDUCT.md).
