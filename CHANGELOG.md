# Changelog

## 1.1.0 — 2026-07-13

- **Manual invocation is now the default.** The `UserPromptSubmit` auto-invocation hook is
  disabled (`hooks/hooks.json` ships with empty `hooks: {}`); coaching runs only when you invoke
  `/prompt-analyzer:analyze-prompt`. The original hook is preserved in the file's
  `_disabled_UserPromptSubmit` block, and `auto_observe` now defaults to `false` — set it to `true`
  and restore that hook to re-enable coaching on every prompt.
- Docs updated (README, CLAUDE.md, config help) to reflect the manual-only default.
- Added `CLAUDE.md` with project guidance for contributors.

## 1.0.0 — 2026-07-08

- Initial release of prompt-analyzer.
- `analyze-prompt` skill: live per-prompt coaching with a standards verdict, checklist,
  stage-aware scoring, focus habit, and a session Analysis Window.
- `prompt-report` skill: read-only end-of-session scorecard.
- Auto-invocation hook, privacy modes (raw / redact_pii / hashed / metadata_only),
  a reference guide, and a marketplace listing.
- Project standards: LICENSE, CONTRIBUTING guide, issue forms, and a pull-request template.
