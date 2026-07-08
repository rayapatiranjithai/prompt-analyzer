# prompt-analyzer

A Claude Code plugin that **analyzes and coaches how you write prompts**, measured against
Anthropic's documented prompt-engineering standards — with persistent, per-session statistics.

It watches how you prompt, scores each prompt, shows the gap between what you *meant* and what you
*wrote*, tracks how often you correct yourself, detects voice vs. typed input, and teaches you the
standard-compliant way to ask — with a concrete rewrite every time.

## Skills

| Skill | Invoke | What it does |
|---|---|---|
| `analyze-prompt` | `/prompt-analyzer:analyze-prompt` | Live per-prompt coaching + a running **Analysis Window** of session stats. |
| `prompt-report` | `/prompt-analyzer:prompt-report` | Read-only end-of-session **scorecard**: best/worst prompt, improvement sparkline, your one focus habit, recommendations. |

## Features

- **Auto-invocation (hooks)** — a `UserPromptSubmit` hook coaches every prompt automatically; no need to call the skill manually. Prefix a prompt with `*` to skip it once.
- **Stage-aware, context-aware scoring** — classifies each prompt (exploration / diagnosis / execution / verification / action) and scores only the criteria that stage needs, so efficient short prompts like `run the tests` aren't unfairly penalized.
- **Standards-based rubric** — reproducible, normalized 10-point score drawn from Anthropic's prompt-engineering guidance (clarity, context, examples, role, structure, output format, scoping…), with labeled bands (Excellent → Needs work) and a safety→correctness→format→style priority order.
- **One focus habit at a time** — headlines the single highest-impact habit to build (from a canonical taxonomy) instead of dumping every flaw.
- **Clarifying questions** — for vague prompts, lists the exact questions you left unanswered.
- **Requirement-vs-delivery gap** — reconstructs your true intent and contrasts it with the literal reading of your prompt.
- **Self-correction & deviation tracking** — counts restatements and drift across the session.
- **Voice detection** — 3-tier: explicit `[voice]` / `[typed]` marker → host flag → heuristic (fillers, run-ons, missing punctuation, homophones).
- **Structured 4-step rewrite** — every rewrite follows Anthropic's Prompt Improver method (role → structured draft → let-Claude-think → example) and preserves `{{variables}}`.
- **Persistent stats** — per-session JSON keyed to `${CLAUDE_SESSION_ID}` (no race conditions, no cross-session clobbering).
- **Reliable timestamps** — injected from the system clock, never guessed.
- **Privacy modes** — `raw`, `redact_pii` (default), `hashed` (store only a hash, dedupe repeats), or `metadata_only`; PII masking for emails, phones, keys, IPs, names, and more.
- **Token & time efficiency** — estimates prompt token cost, tokens/minutes wasted on redos, and an efficiency %.

## Requirements

- Claude Code CLI (recent version with plugin + skills support).
- A shell with `date`, `cat`, `test`, `ls`, `grep`, `printf` (used for context injection and the hook).

## Install

### Option A — local (development / personal use)
Run Claude Code with the plugin directory mounted:

```bash
claude --plugin-dir /path/to/prompt-analyzer
```

Then invoke a skill:

```
/prompt-analyzer:analyze-prompt
/prompt-analyzer:prompt-report
```

Use `/reload-plugins` after editing any file to pick up changes without restarting.

### Option B — from a marketplace
If the plugin is published to a marketplace you've added:

```
/plugin install prompt-analyzer
```

(Manage marketplaces with `/plugin marketplace add <owner/repo | url>` and browse with `/plugin`.)

## Configuration

Privacy is controlled by `.prompt-analyzer/config.json`:

```json
{
  "privacy_mode": "redact_pii",
  "store_raw_prompts": true,
  "auto_observe": true,
  "coach_one_habit_at_a_time": true
}
```

| `privacy_mode` | Behavior |
|---|---|
| `raw` | Store prompts verbatim (requires `store_raw_prompts: true`). |
| `redact_pii` *(default)* | Store prompts with PII masked (`[EMAIL]`, `[KEY]`, `[IP]`, `[NAME]`, …). |
| `hashed` | Store only a hash prefix — no readable text — used to dedupe repeated prompts. |
| `metadata_only` | Store no prompt text or hash — only scores, habits, and derived metrics. |

Other config keys: `auto_observe` (hook coaches every prompt), `coach_one_habit_at_a_time` (headline
a single focus habit). See the `_help` block in `config.json`.

Session data lives in `.prompt-analyzer/<session-id>.json` and is **git-ignored** by default; only
`config.json` is tracked.

## Usage

1. Run `/prompt-analyzer:analyze-prompt`, then prompt as normal. Each turn you get an Analysis
   Window (running totals) plus coaching: your score, the intent-vs-literal gap, what broke, and a
   standard-compliant rewrite.
2. Force voice mode anytime by prefixing a prompt with `[voice]` (or `[typed]` to force typed).
3. At the end, run `/prompt-analyzer:prompt-report` for the session scorecard.

## Project structure

```
prompt-analyzer/
├── .claude-plugin/
│   ├── plugin.json                 # plugin manifest
│   └── marketplace.json            # marketplace listing (source: ".")
├── hooks/
│   └── hooks.json                  # UserPromptSubmit auto-invocation (bypass with *)
├── skills/
│   ├── analyze-prompt/
│   │   ├── SKILL.md                # live coaching + stats
│   │   └── reference.md            # PII regexes, habit taxonomy, rewrite method, templates
│   └── prompt-report/
│       └── SKILL.md                # end-of-session summary (read-only)
├── .prompt-analyzer/
│   ├── config.json                 # privacy settings (tracked)
│   └── <session-id>.json           # per-session stats (runtime, git-ignored)
├── CHANGELOG.md
├── LICENSE
├── .gitignore
└── README.md
```

## Validate & publish to a marketplace

Validate the manifest and skills before sharing:

```bash
claude plugin validate /path/to/prompt-analyzer
```

This repo doubles as its own **marketplace** — `.claude-plugin/marketplace.json` lists this plugin
with `"source": "."`, so once the repo is public, users can add it and install directly:

```
/plugin marketplace add rayapatiranjithai/prompt-analyzer
/plugin install prompt-analyzer@rayapatiranjithai-plugins
```

To submit to the **official community directory**, use Anthropic's plugin submission flow
   (via `claude.ai` admin settings → directory → plugin submissions, or the Claude platform
   plugin submission page). Ensure `plugin validate` passes and your repo is public first.

## Author

Created and maintained by **Ranjith Rayapati** — [@rayapatiranjithai](https://github.com/rayapatiranjithai).

Repository: <https://github.com/rayapatiranjithai/prompt-analyzer>

## License

MIT © [Ranjith Rayapati](https://github.com/rayapatiranjithai)
