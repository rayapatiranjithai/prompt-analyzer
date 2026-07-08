## Summary

<!-- What does this PR change, and why? -->

## Type of change

- [ ] Bug fix
- [ ] New feature / capability
- [ ] Coaching / scoring change
- [ ] Docs only
- [ ] Chore / tooling

## How I tested it

<!-- e.g. ran `claude --plugin-dir .`, invoked /prompt-analyzer:analyze-prompt on prompts X and Y,
     confirmed the verdict/checklist/rewrite, ran `claude plugin validate`. -->

## Checklist

- [ ] Tested live with `claude --plugin-dir` and `/reload-plugins`
- [ ] `claude plugin validate` passes
- [ ] If I changed the session-data schema, I updated **both** `analyze-prompt` (writer) and `prompt-report` (reader)
- [ ] Skill `description` still starts with "Use when …" and lists trigger keywords
- [ ] No raw prompt text is persisted outside the allowed `privacy_mode`
- [ ] Updated `CHANGELOG.md`
