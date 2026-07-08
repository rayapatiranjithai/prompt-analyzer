---
name: prompt-report
description: "Use when the user wants an end-of-session summary of their prompting: overall stats, best and worst prompts, biggest recurring mistake, improvement curve, and personalized recommendations. Keywords: prompt report, session summary, how did I do, my prompting stats, prompt scorecard, prompting progress."
when_to_use: "Examples: 'give me my prompt report', 'how did I do this session', 'summarize my prompting', 'show my prompting progress'."
user-invocable: true
allowed-tools: "Read Bash(cat *) Bash(test *) Bash(ls *)"
---

# prompt-report — End-of-session prompting summary

You produce a one-shot summary from the data that the `analyze-prompt` skill has persisted this
session. You do NOT re-analyze prompts and you do NOT modify any data file (read-only).

## GATE
Read the session data first. If it is missing or empty, tell the user no prompts have been analyzed
yet this session and to run `analyze-prompt` first — then stop.

## DATA (read-only)
- **Data file:** `${CLAUDE_PROJECT_DIR}/.prompt-analyzer/${CLAUDE_SESSION_ID}.json`

### Injected data
```
!`test -f "${CLAUDE_PROJECT_DIR}/.prompt-analyzer/${CLAUDE_SESSION_ID}.json" && cat "${CLAUDE_PROJECT_DIR}/.prompt-analyzer/${CLAUDE_SESSION_ID}.json" || echo "NO_DATA_YET"`
```
If `NO_DATA_YET`, run the GATE fallback above.

### List other sessions (for optional cross-session view)
```
!`ls -1 "${CLAUDE_PROJECT_DIR}/.prompt-analyzer/"*.json 2>/dev/null | grep -v config.json | wc -l`
```

## Computation (from persisted data only)
- avg_score = mean(scores); trend from first_score → latest_score
- requirement_match_rate = requirement_matches / total_prompts × 100
- best prompt = history entry with max score; worst = min score
- focus habit = the `focus_habit` field (the one habit to build); biggest recurring habit = max key
  in `habit_tally`
- improvement curve = the scores[] array rendered as a sparkline (▁▂▃▄▅▆▇█ mapped 1–10)
- self-correction rate = total_self_corrections / total_prompts × 100
- efficiency = (1 − est_wasted_tokens / est_prompt_tokens) × 100, shown as %
- est. tokens spent = totals.est_prompt_tokens; est. wasted on redos = totals.est_wasted_tokens
  (label both "est." — heuristic, not an exact tokenizer)
- **time impact (est.)** = frame waste in minutes: each self-correction ≈ 1–2 min of rework;
  report "≈N min lost to redos" and "≈N min saved by your strong prompts" (label "est.")
- **cross-session rollup** = if the "other sessions" count > 0, note whether the focus habit is
  improving across sessions (mention it qualitatively; do not read other files unless asked)

## OUTPUT
```
╔══════════════════ 📈 PROMPT SESSION REPORT ══════════════════╗
  Session: <session_id>            Prompts analyzed: N
  Average score: X.X/10            Trend: first X.X → latest X.X  (Improving/Flat/Declining)
  Curve: ▁▂▃▄▅▆▇█  (per-prompt scores)
  Requirement match rate: NN%      Self-correction rate: NN%
  Times Claude had to guess: N     Deviations: N
  Input mix: 🎙 voice N · ⌨ typed N
  Est. tokens: ~N   Wasted on redos: ~N (~NN%)   Efficiency: NN%
  Est. time: ≈N min saved · ≈N min lost to redos
  Focus habit to build: "<canonical-habit>"
╚═══════════════════════════════════════════════════════════════╝
```

**✅/⚠️/❌ Overall: are you prompting to Claude's standards?**
[ONE plain-language verdict based on avg score + trend, e.g. "Mostly yes — NN% of your prompts met
Claude's standards, and you're improving." or "Not yet — most prompts left Claude guessing; the fix
is one habit below."]

**🏆 Best prompt (X/10)**
> [best prompt text or redacted version from history] — why it worked: [1 line]

**⚠️ Weakest prompt (X/10) — before → after**
> Before: [worst prompt text or redacted version] — what broke: [1 line]
> After:  [a standard-compliant rewrite]

**🎯 Your one focus habit:** "<focus_habit>" (seen N times)
[one sentence on the pattern + the single fix that builds this habit]

**⭐ Gold-standard template to reuse next time:**
```
[ROLE] · [GOAL] · [CONTEXT/why] · [INPUT] · [CONSTRAINTS] · [OUTPUT format] · [VERIFY]
```

**📌 3 recommendations for next session:**
1. [specific, tied to their actual data]
2. …
3. …

**Bottom line:** [2–3 sentences: are they improving, what to focus on next]

## RULES
- Read-only: never write to any data file.
- Use only persisted data — do not invent numbers; if a field is absent, say "not recorded".
- Respect privacy: if `prompt_redacted` is null (hashed or metadata_only mode), reference prompts by
  turn number and score instead of quoting text.
- Coach ONE focus habit, not a long list; keep the tone encouraging.
- Keep it to one screen; this is a summary, not a re-analysis.
