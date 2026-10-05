# tiny-agent-mods

A tiny agent middleware inspired by Anthropic's Claude Code Mods, in one TypeScript file.
It runs the same scripted agent through three pipelines: no mods, useful mods, and useful mods with a `sec-default` that loads first.
The agent is mocked. No model, no API key.

## Why it matters

Hooks watch. Mods rewrite.

On October 3, 2026, Anthropic [introduced Mods for Claude Code](https://claude.com/blog/claude-code-mods): small TypeScript functions that hook into events inside the tool. A mod can rewrite a prompt, block or retry a tool call, redact secrets from tool output, or change the UI.

Anthropic's own line: "Hooks helped give users some of this control, but hooks can't rewrite events, draw new UI, or replace features. Mods can."

Mods are not sandboxed. Anthropic ships a built-in `sec-default` that loads first on Team and Enterprise plans so user-installed mods cannot override permission deny rules.

## Run it

You need Node.js 18 or newer.

```bash
npm install
npx tsx mods.ts
```

## Example output

This is real output from `npx tsx mods.ts` (trimmed):

```text
=== no mods ===
tool.call Read(.env) -> OPENAI_API_KEY=sk-live-ABCDEF1234567890XYZSECRET
tool.call Bash(rm -rf /) -> ran: rm -rf /
permission Bash(kubectl apply -f prod.yaml): deny

=== useful mods (no sec-default) ===
  [redact-secrets] redacted a secret in Read output
tool.call Read(.env) -> OPENAI_API_KEY=[REDACTED]
  [block-dangerous-shell] blocked: rm -rf /
tool.call Bash(rm -rf /) -> BLOCKED by mod: dangerous shell
  [auto-allow] flipping deny -> allow
permission Bash(kubectl apply -f prod.yaml): allow

=== with sec-default first ===
  [auto-allow] flipping deny -> allow
  [sec-default] kept deny (a later mod tried to allow)
permission Bash(kubectl apply -f prod.yaml): deny
```

Without mods, the secret prints and the dangerous shell runs.
With useful mods, the secret is redacted and the shell is blocked, but a hostile auto-allow mod still flips deny to allow.
With `sec-default` first, the deny stays a deny.

## How it works

```text
Event
  ↓
sec-default (first)
  ↓
Your mods (rewrite / block / redact)
  ↓
Agent loop
  ↓
Result (maybe rewritten on the way back)
```

| File | What it does |
| --- | --- |
| `mods.ts` | Event types, pipeline, four mods, hostile auto-allow, scripted run |
| `output.txt` | Captured stdout from one run |

## Write-up

Full tutorial: SUBSTACK_URL

Also: DEV_URL

## License

MIT
