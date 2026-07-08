# analyze-prompt — Reference

Heavy detail kept out of SKILL.md. Consult when redacting PII or when a score needs justifying.

## 1. PII redaction patterns
Apply these (case-insensitive) before writing any prompt text, `intent`, or `literal_reading` to
disk when `privacy_mode` is `redact_pii`. Replace the match with the bracket token.

| Token | Regex (ECMAScript) | Notes |
|---|---|---|
| `[EMAIL]` | `[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}` | |
| `[PHONE]` | `(?:\+?\d{1,3}[\s-]?)?(?:\(?\d{2,4}\)?[\s-]?){2,4}\d{2,4}` | 7+ digits total; ignore short counts |
| `[CARD]` | `\b(?:\d[ -]?){13,16}\b` | credit-card-length digit runs |
| `[SSN]` | `\b\d{3}-\d{2}-\d{4}\b` | US SSN shape |
| `[KEY]` | `\b(?:sk-|pk-|ghp_|xox[baprs]-|AKIA|AIza)[A-Za-z0-9_\-]{10,}\b` | common API-key prefixes |
| `[TOKEN]` | `\b[A-Fa-f0-9]{32,}\b` | long hex (hashes/tokens) |
| `[JWT]` | `\beyJ[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+\b` | JWTs |
| `[IP]` | `\b(?:\d{1,3}\.){3}\d{1,3}\b` | IPv4 |
| `[URL]` | `https?://[^\s]+` | keep host only if useful, else mask whole |
| `[ADDRESS]` | `\b\d{1,5}\s+([A-Z][a-z]+\s){1,3}(St|Street|Ave|Avenue|Rd|Road|Blvd|Lane|Ln|Dr|Drive)\b` | street lines |
| `[NAME]` | contextual — mask capitalized personal names following "I'm", "my name is", "this is", signatures | judgment call; do not mask product/tech names |
| `[CREDENTIAL]` | `(?i)(password\|passwd\|secret\|api[_-]?key)\s*[:=]\s*\S+` | key/value secrets |

Order: run the specific patterns (email, key, JWT) before the generic ones (TOKEN, PHONE) so a
match isn't half-consumed. If two patterns overlap, the longer match wins.

## 2. Scoring rubric — worked examples
Score = (applicable points earned / applicable points) × 10. Criteria not needed by the stage are
auto-satisfied ("free"). The examples below are all `execution` stage, where ALL criteria apply.
Criteria: clear goal · success criteria/detail · context+motivation · example (when format-specific) ·
role (when expertise/tone needed) · structure/separation · output format · positive phrasing ·
single scope · no guess-forcing ambiguity · let-Claude-think (reasoning tasks).

### Example A — score 2/10
> "make it better"
- ✗ clear goal (what is "it"?) ✗ detail ✗ context ✓ example(free) ✓ role(free) ✗ structure
  ✗ output format ✓ positive ✗ scope (undefined) ✗ ambiguity (forces guess)
- **Fix:** "Rewrite the function below for readability — rename unclear vars, add docstrings, keep
  behavior identical. Return only the revised code." → 8/10

### Example B — score 5/10
> "Write a Python script that reads a CSV and prints the average of the price column"
- ✓ goal ✓ detail ✗ context/why ✓ example(free) ✓ role(free) ✗ structure ✗ output format
  (nothing on errors/edge cases) ✓ positive ✓ scope ✓ no ambiguity
- **Fix:** add "CSV path is an arg; skip rows with missing/non-numeric price; print rounded to 2
  decimals" → 8/10

### Example C — score 9/10
> "You are a senior data engineer. Given the schema below <schema>, write a BigQuery SQL query that
> returns the top 10 customers by 2025 revenue. Output only the SQL, no explanation."
- Loses 1 only for no example of expected output shape. Role, context, structure, format, scope all
  present. Strong prompt.

## 3. Anti-pattern → canonical habit (habit_tally key)
Canonical habits keep coaching consistent and let the same habit be tracked across sessions.

| Symptom | Habit to build (habit_tally key) |
|---|---|
| "make it better", "help with this" | `clarify-goal` |
| no audience/purpose/why | `add-context-motivation` |
| format-specific task, no sample | `give-examples` |
| expertise/tone needed, no persona | `assign-role` |
| big prompt, no sections/tags | `structure-with-tags` |
| no length/structure/format stated | `specify-output-format` |
| only "don't do X" phrasing | `state-positively` |
| several unrelated asks in one prompt | `scope-one-task` |
| huge multi-stage task in one shot | `scope-one-task` |
| ambiguous — forces a guess | `remove-ambiguity` |
| reasoning task, no room to think | `let-claude-think` |
| contains PII/secrets | `redact-sensitive-data` |

## 4. Stage → applicable criteria (for stage-aware scoring)
| Stage | Applicable criteria (others auto-satisfied) |
|---|---|
| exploration | clear goal · no-ambiguity · safety |
| diagnosis | clear goal · context · no-ambiguity · safety |
| execution | ALL ten criteria |
| verification | clear goal · context · output-format · safety |
| action (≤8 words + live context) | clear goal · no-ambiguity · safety |

## 5. The 4-step rewrite method (Anthropic Prompt Improver)
When producing "How to ask it, to standard", apply these in order:
1. **Identify examples** — pull any examples/format hints from the user's prompt.
2. **Structured draft** — add a role, restate the task with success criteria, separate parts with
   XML tags or headings, place long input/context before the instruction.
3. **Let Claude think** — for reasoning tasks add step-by-step instructions inside `<analysis>` tags.
4. **Enhance examples** — make examples show the desired reasoning and output shape.
Also apply, as relevant: positive phrasing (say what TO do), explicit output format, and remove
anything ambiguous. Preserve any `{{variables}}` the user used.

## 6. Gold-standard prompt template (reliably scores 90%+)
A six-part structure to teach users and to model rewrites on:
```
[ROLE]      You are a <expert role>.
[GOAL]      <one sentence: the outcome you want>.
[CONTEXT]   <why this matters, audience, how the output is used, relevant background/data first>.
[INPUT]     <the specific material to act on, in <tags> if long>.
[CONSTRAINTS] <scope limits, must/must-not, edge cases>.
[OUTPUT]    <format, length, structure; say what TO include>.
[VERIFY]    <how success is checked, or "before finishing, confirm X">.
```

## 7. Standards source — these are Anthropic's documented rules
Use these names when telling the user a rule is "Claude's standard", so it reads as documented
guidance, not the plugin's opinion. All come from Anthropic's prompt-engineering documentation.

| Checklist item | Anthropic principle (as documented) |
|---|---|
| Clear goal | Be clear, direct, and detailed |
| Context & why | Give Claude context / motivation for the task |
| Example | Use examples (multishot prompting) |
| Role | Give Claude a role with a system prompt |
| Structure | Use XML tags / structure to organize the prompt |
| Output format | Control the format of responses |
| Positive wording | Tell Claude what to do instead of what not to do |
| One task | Chain complex prompts; keep each step focused |
| No ambiguity | Be clear and direct (a smart colleague with no context should understand) |
| Let Claude think | Let Claude think (chain-of-thought) |

When helpful, tell the user these are from Anthropic's "Prompt engineering" guide so they know the
verdict reflects Claude's own standard.

## 8. Derived-metric formulas
(Section order: 1 PII · 2 scoring examples · 3 habits · 4 stages · 5 rewrite · 6 template ·
7 standards source · 8 formulas.)
- avg_score = mean(scores) · delta = latest − previous
- requirement_match_rate = requirement_matches / total_prompts × 100
- trend: latest−first ≥ +1 Improving · ≤ −1 Declining · else Flat
- focus_level: avg of last 3 scores ≥ 7 High · 4–6 Medium · <4 Low (drop one level if
  self-correction rate > 40%)
