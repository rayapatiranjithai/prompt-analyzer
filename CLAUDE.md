# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`prompt-analyzer` is a **Claude Code plugin** (no compiled code, no test runner). It coaches the
user on *how they write prompts*, scores each prompt against Anthropic's documented
prompt-engineering standards, and persists per-session statistics. The "logic" lives in Markdown
skill instructions that Claude executes — not in a program. Editing this repo means editing
prompts/config, then validating with the Claude CLI.

## Commands

```bash
claude plugin validate .          # validate plugin.json + skills before sharing (run after any edit)
claude --plugin-dir .             # run Claude Code with this plugin mounted (local dev)
```

Inside a Claude session:
- `/reload-plugins` — pick up file edits without restarting.
- `/prompt-analyzer:analyze-prompt` — live per-prompt coaching + Analysis Window (the `analyze-prompt` skill).
- `/prompt-analyzer:prompt-report` — read-only end-of-session scorecard (the `prompt-report` skill).

There is no build/lint/test step. "Testing" a change = validate, `/reload-plugins`, then submit a
prompt and confirm the coaching/stats behave.

## Architecture — the three moving parts

The whole plugin is a loop between a **hook**, two **skills**, and a **per-session JSON file**:

1. **`hooks/hooks.json`** — auto-invocation is **disabled by default** (`"hooks": {}`); coaching
   runs only when the user explicitly invokes `/prompt-analyzer:analyze-prompt`. The original
   `UserPromptSubmit` hook is preserved in the `_disabled_UserPromptSubmit` block: it fires on
   *every* prompt and injects a `<system-reminder>` telling Claude to run the skill (a leading `*`
   exempts a prompt). To turn auto-coaching back on, move that block back under `hooks` and set
   `auto_observe: true` in `config.json`. Note the hook does **not** read `auto_observe` — the flag
   is documentation-only, so the hook's presence/absence is the real switch.

2. **`skills/analyze-prompt/SKILL.md`** — the coach. It is stateful and follows a strict
   **READ → analyze → UPDATE → WRITE → render** protocol every turn:
   - Reads the injected session data (or initializes from the embedded schema on `NO_DATA_YET`).
   - Classifies the prompt's **stage** (exploration / diagnosis / execution / verification / action)
     and scores only the criteria that stage requires (stage-aware scoring — short `action` prompts
     riding on live context are not penalized for missing detail).
   - Maps each gap to a **canonical habit** key, increments `habit_tally`, picks ONE `focus_habit`.
   - **Writes the updated JSON back to disk with the Write tool**, then renders the Analysis Window
     + coaching from the written data. The written file is the source of truth for stats — never
     render stats without writing first, and never reset counters mid-session.

3. **`skills/prompt-report/SKILL.md`** — the summarizer. **Read-only**: it computes a scorecard
   from the persisted JSON and must never write to any data file. If data is missing it tells the
   user to run `analyze-prompt` first.

4. **`skills/analyze-prompt/reference.md`** — heavy detail kept out of SKILL.md: PII redaction
   regexes, worked scoring examples, the symptom→habit table, stage→criteria table, the 4-step
   rewrite method, the gold-standard template, and the checklist→Anthropic-principle mapping. Edit
   this when changing *how* prompts are scored or redacted; keep SKILL.md as the concise protocol.

### Data & privacy contract

- Session data lives at `.prompt-analyzer/<session-id>.json`, keyed to `${CLAUDE_SESSION_ID}` so
  concurrent sessions never clobber each other. This file is **git-ignored**; only
  `.prompt-analyzer/config.json` is tracked.
- `config.json` `privacy_mode` controls what prompt text is written to disk: `raw` /
  `redact_pii` (default, masks emails/keys/IPs/names/etc.) / `hashed` (hash prefix only, dedupes
  repeats via `repeat_count`) / `metadata_only` (no text or hash). Redaction is applied to
  `prompt_redacted`, `intent`, and `literal_reading` before any write — see `reference.md` §1.
- The plugin runs **entirely locally with no telemetry** (see `PRIVACY.md`). Preserve that
  guarantee in any change.
- History is bounded: keep only the last 20 raw `history[]` entries; roll older ones into
  `totals`/`habit_tally`. Timestamps come from an injected clock, never guessed.

## Conventions when editing skills

- The scoring rubric must be **reproducible** — identical prompts must score identically. If you
  change scoring, update both the rubric in `SKILL.md` and the worked examples in `reference.md`.
- Every criticism must be grounded in a named Anthropic principle (mapping in `reference.md` §7) and
  paired with a concrete rewrite; coach **one** focus habit at a time.
- `plugin.json` and `.claude-plugin/marketplace.json` both carry a `version` — bump both together,
  and update `CHANGELOG.md`. The marketplace uses `"source": "."` (this repo is its own marketplace).
- The skill never performs the user's underlying task — it only analyzes and coaches.
