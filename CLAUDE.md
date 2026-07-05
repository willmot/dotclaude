# How I work

My work is mostly Node/JavaScript LLM-application code — agentic systems, prompt
tuning, eval-driven development, tool/model routing — often at an engineering
agency. Match each repo's own conventions; the rules below are cross-project.

## Code & change style

- **Fix root causes, not symptoms.** Prefer a fix at the source — the prompt, the
  data, the actual bug — over a post-processing guard or band-aid that masks it.
- **No speculative config knobs.** Hardcode the constant and change the code when
  the need changes. Env vars are only for genuine per-environment values
  (credentials, IDs, URLs) — never for timeouts, thresholds, intervals, or tuning.
- **Don't add explanatory labels or chrome when the affordance is self-evident.**
  Scarce UI space is for content.
- **Match the surrounding code** — its naming, idioms, comment density, error
  handling. Consistency is a shared vocabulary, not identical layout.
- **Show evidence, not assertions.** When you say something works, include the test
  output, the command and its result, or a screenshot — the thing I'd check anyway.
  If tests fail or a step was skipped, say so plainly.

## Delegation (auto-delegate execution by tier)

Two coding subagents in `~/.claude/agents/` execute below the frontier lead.
Capability-tier names, so model upgrades never force renames:

- **`jnr-code-engineer`** (Sonnet) — fully-specified mechanical work: pre-approved
  plans, renames, schema fields, tests-from-pattern, mechanical translations. Every
  decision must already be named.
- **`snr-code-engineer`** (Opus, high effort) — semi-defined work where the approach
  is visible but not fully specified: debugging with a clear surface, style-sensitive
  refactors, consolidations needing judgment among existing patterns.

**Default to delegating coding execution** — including small, unplanned tasks. The
lead scopes, decides, and reviews. Route by what's *missing* from the task:

- Nothing missing (decisions named) **and** small, clean blast radius → **jnr**
- Decisions named but the diff is large or the surrounding code is messy → **snr**.
  Cheap-tier quality drops sharply on big/messy diffs even when fully specified —
  size is the dominant quality predictor, so specification alone doesn't qualify a
  task for jnr.
- Technical judgment missing (which seam/shape/approach among visible patterns) → **snr**
- Product or stack decision missing (which library, does infra exist, breaking
  upgrade) → **ask me first**
- Choosing the approach IS the work (exploration, open-ended debugging, design,
  architecture, security-sensitive) → **stay on the lead**
- Trivial (≤ ~2 small edits in files already in context) → **stay inline**; spawn
  overhead exceeds the saving

When either agent returns a `BLOCKER:` line, answer the question and re-spawn with
the decision included. If I asked for a review, run the `/code-review` skill against
the resulting diff before reporting back.

### Verification delegation

Verification execution follows the same rule — it doesn't stay on the lead. Every
inline verify turn re-reads the whole session as cache; screenshots are the worst
offenders (image tokens paid on read, then re-read every later turn).

- Everything scriptable — test/build/lint matrices, API/CLI checks, render checks,
  screenshot acceptance → **`qa-runner`** (Sonnet): I specify the check matrix; it
  executes, views its own screenshots, and returns a ≤40-line pass/fail report with
  evidence paths.
- Stay inline only for a single quick check on state already in context.
- For a correctness gate on a diff, use the `/code-review` skill (fresh-context
  reviewer). Scope reviewers to correctness and stated requirements — **not style** —
  so they don't invent over-engineering.

## Model routing (subagents & workflows)

Everything anchors to the session lead. These are defaults, not limits — judge the
output, not the price tag. If a cheaper tier misses the bar, redo it on a smarter
one without asking; escalating costs less than shipping mediocre work.

- **Fable 5 (session lead)** — scoping, architecture, design, security-sensitive
  code, review synthesis. Runs at **high** effort; xhigh is a token furnace —
  reserve it for a single hard reasoning step, never default there.
- **opus** — the snr execution band, UI/runtime verification of hard cases, and
  token-hungry investigation or bulk analysis. Also in-harness heavy workflow stages
  (verify/judge panels).
- **sonnet** — the jnr execution band, `qa-runner` verification, and
  specified-judgment stages (classification, summaries, edits against written rules).
- **haiku** — non-intelligent grunt only: `web-runner`, mechanical transforms, dumb
  bash sweeps. **Never coding.**

**Lead fallback.** If Fable is unavailable — dropped from the plan, out of credits,
or rate-limited — the lead falls back to **Opus 4.8**. Whether you switch models
yourself or the harness downgrades you, route relative to whichever model is leading.
Run Opus at high effort; `max` (Opus-only) unlocks for the single hardest step, but
it's the priciest tier — use it sparingly. Nothing else moves: snr stays Opus,
jnr/`qa-runner` stay Sonnet, `web-runner` stays Haiku. With the lead and snr now on
the same tier the capability step-up is gone — so take the hardest reasoning inline,
and delegate execution for **context hygiene**, not tier savings.

## Context-cost discipline (long sessions)

- **Batch shell work into few chunky scripts**, not many small turns — each turn
  re-reads the full session as cache, so turn count (not command count) drives cost.
- **Cap subagent return contracts.** Give research/exploration spawns an explicit
  max length and structure; a verbose report lands in my context and is cache-re-read
  every turn after.
- **Delegate investigation to the `Explore` agent** so file-reading noise never
  touches the lead's context.
- `/clear` between unrelated tasks. After two failed corrections on the same issue,
  `/clear` and restart with a sharper prompt rather than piling on.
