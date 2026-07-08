# Privacy Policy — prompt-analyzer

_Last updated: 2026-07-08_

**prompt-analyzer runs entirely on your machine. It collects no data, sends nothing to any server,
and has no telemetry, analytics, or tracking.**

## What data is processed
The plugin analyzes the prompts you write during a Claude Code session in order to coach you and
compute statistics. All processing happens locally inside your own Claude Code session.

## What is stored, and where
- Per-session statistics are written to a local file in your project:
  `.prompt-analyzer/<session-id>.json`.
- Configuration is stored in `.prompt-analyzer/config.json`.
- Nothing is written anywhere else, and nothing leaves your device.

## Prompt text and privacy modes
Whether your prompt text is stored is controlled by `privacy_mode` in `config.json`:

| Mode | Prompt text stored |
|---|---|
| `raw` | Stored verbatim (only if `store_raw_prompts` is true) |
| `redact_pii` *(default)* | Stored with PII masked (emails, phone numbers, cards, API keys, IPs, addresses, names, etc.) |
| `hashed` | Not stored — only a short one-way hash prefix, used to de-duplicate repeats |
| `metadata_only` | Not stored at all — only scores, stages, and habit counts |

The live, on-screen coaching may show full detail, but that output is displayed in your terminal and
is **not** persisted.

## No third parties
The plugin makes no network requests and shares no data with the author, Anthropic, or any third
party. Coaching and scoring occur within your local Claude Code session.

## Your control
- The local data files are **git-ignored by default** so they are never committed.
- You can change `privacy_mode` at any time by editing `config.json`.
- You can delete all stored data at any time by removing the `.prompt-analyzer/*.json` session files.

## Contact
Questions: open an issue at
<https://github.com/rayapatiranjithai/prompt-analyzer/issues>.
