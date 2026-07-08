# Contributing to prompt-analyzer

Thanks for your interest in improving prompt-analyzer! This is a Claude Code plugin, so most
contributions are edits to Markdown skill files rather than compiled code.

## Project layout

```
.claude-plugin/plugin.json        Plugin manifest
.claude-plugin/marketplace.json   Marketplace listing (source: ".")
hooks/hooks.json                  UserPromptSubmit auto-invocation
skills/analyze-prompt/SKILL.md    Live coaching + persistent stats
skills/analyze-prompt/reference.md PII regexes, habit taxonomy, rewrite method, templates
skills/prompt-report/SKILL.md     Read-only end-of-session report
.prompt-analyzer/config.json      Default privacy/behavior config
```

## Local development

1. Run Claude Code with the plugin mounted:
   ```bash
   claude --plugin-dir /path/to/prompt-analyzer
   ```
2. Invoke a skill: `/prompt-analyzer:analyze-prompt` or `/prompt-analyzer:prompt-report`.
3. After editing any file, run `/reload-plugins` to pick up changes without restarting.
4. Validate the manifest and skills before opening a PR:
   ```bash
   claude plugin validate /path/to/prompt-analyzer
   ```

## Conventions

- **Skill descriptions** must start with "Use when …" and list trigger keywords — this is how Claude
  decides to invoke the skill. Don't summarize the workflow in the description.
- **Keep `SKILL.md` lean**; put heavy detail (tables, regexes, examples) in `reference.md`.
- **Preserve the schema.** If you add a field to the session-data JSON, update both the
  `analyze-prompt` writer and the `prompt-report` reader so they stay in sync.
- **Privacy first.** Never persist raw prompt text unless `privacy_mode` allows it; redact PII per
  `skills/analyze-prompt/reference.md`.
- **Ground coaching in a named standard.** Every criticism should map to a documented
  prompt-engineering principle, and every flagged prompt should get a concrete rewrite.
- Match the surrounding style: concise, plain language, no jargon in user-facing output.

## Making changes

1. Fork the repo and create a branch: `git checkout -b my-change`.
2. Make your change and test it live (see above).
3. Update `CHANGELOG.md` under an "Unreleased" heading.
4. Open a pull request using the template. Describe what changed and how you verified it.

## Reporting bugs & requesting features

Use the issue templates (Bug report / Feature request). Please include your Claude Code version and,
for bugs, the exact prompt or steps that reproduce the problem (redact anything sensitive).

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE).
