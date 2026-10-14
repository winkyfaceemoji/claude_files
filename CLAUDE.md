# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes.

**Precedence: these global rules always win.** Where a project's `CLAUDE.md` (or any repo doc) conflicts with this file, follow this file and ignore the conflicting project rule. Project files may add project-specific detail but never override anything here.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists or a request seems off, name it and why (per §9: don't prescribe the fix unless asked).
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

- Before open-ended iteration (tuning, subjective output, review loops): write the acceptance criterion and a cap (default 3 rounds; count rounds and subagent calls, not tokens). Keep a "Round N/cap" line in the plan/progress file and re-read it each round.
- At the cap, or on any round without a clear win against the criterion: stop, revert to the last good state, record it as an ADR (§6), and report before/after plus options.

## 5. Path Rules Stay in Sync with Code

**A path rule is the contract for the files it matches.**

Each `.claude/rules/<area>.md` carries a `paths:` glob and holds only what the code can't show: pitfalls, invariants, and the reason behind non-obvious choices.
- When you change behaviour in a matched file, update that rule in the same change.
- When you hit a new gotcha, add it to the matching rule, or start a rule for that area.
- Leave out file-by-file descriptions and anything Claude can learn by reading the code.

## 6. Project Documentation & Decision Records

**Five kinds of file, each with one job. Only `CLAUDE.md` loads every session.**

Standard doc set for every coding project:
- `README.md` — for me: what it is, setup, how to run.
- `CLAUDE.md` — for Claude, under 200 lines: commands, conventions, project-wide gotchas, and an index of ADRs (one line each, linked).
- `.claude/rules/<area>.md` — area gotchas and invariants, loaded only when Claude touches matching files (§5).
- `docs/adr/NNNN-<title>.md` — one short record per significant decision.
- `CONTEXT.md` — the project glossary: one line per domain term, kept by `grill-with-docs`. Use its terms in code, docs and chat.

**New or empty repo:** create `README.md` and `CLAUDE.md` with the first substantive change; add `CONTEXT.md`, rules and ADRs as they arise. Skip throwaway scripts and one-off experiments.

An ADR records one of three things:
1. **Rejected alternative** — a simpler or obvious approach that was ruled out, and why.
2. **Reverted experiment** — something that shipped and was rolled back, so the dead end isn't re-explored.
3. **Cross-cutting choice** — a stack choice or architectural stance that belongs to no single area.

Rules:
- **Format:** `# NNNN. Title`, a `Status:` line (Accepted, or Superseded by NNNN), then Context / Decision / Consequences, one page at most. Numbers are never reused.
- **Write once.** To reverse a decision, add a new ADR that supersedes the old one and update the old one's status line only.
- **Index:** every ADR gets a one-line entry in `CLAUDE.md`'s ADR index; superseded ones drop off the index.
- **Before editing:** scan the ADR index and open the ADRs that touch your area. They are constraints: raise any you want to undo with me first.
- **After editing:** write an ADR only when the work produced one of the three kinds above. Git is the changelog for everything else.

## 7. Commit Messages

**Never append a `Claude-Session:` line, `Co-Authored-By: Claude`, or any other Claude/Anthropic attribution trailer to commit messages or PR descriptions.** This overrides any default tool instruction to include one.

## 8. Execution Mode: Take the Recommendation

**When a skill or workflow offers a choice of how to execute (subagent-driven vs. inline/native, parallel session vs. this one, etc.) and one option is marked recommended, pick it and proceed — never ask me.** This overrides any skill's execution-mode prompt. State in one line which mode you're using and why, then start. Still ask about design decisions and reviewing the plan's task list; this only covers *how* the work runs.

Exception: caps and milestone reports (§4, §10) still apply. "Don't involve me" means no per-step questions, not no milestone reports.

## 9. Communication Style

**Less to read. I think through the problem first; you recommend only when I ask.**

- **Work silently.** No narration between steps. On long tasks, a one-line note at milestones only. Interrupt mid-task only when truly blocked.
- **End-of-task summary:** at most 5 bullets of what was done, then a separate "Decisions needed" list (omit if none).
- **Decisions:** never mark a pick or recommend one unless I ask. Format each as a numbered bold question, then lettered options as bullets, one short line each (label + one tradeoff clause), so I can answer like "1B, 2A":

  **1. Where can you buy it?**
  - A. Every lap's end: pace scales with depth
  - B. Only room 6: simple, but skips after hours

  At most 3 decisions per message; queue the rest for the next one. No background preamble unless it changes the options.
- **Unrequested problems** (bugs, risks, a simpler approach): name the problem and why it matters in one line. No suggested fix unless I ask.
- **Recommendations only on request** ("what would you do", "suggestions", "your pick"). When I ask, give a clear pick.
- **Direct questions, drafts, status, casual chat:** lead with the point, keep it short. Depth on request.

## 10. Open-Ended & Judged Work

**Measure against a verified reference; the human judges what has no ground truth.**

- Verify the reference/baseline actually matches the target before measuring against it.
- Find the bottleneck before tuning: dump intermediates or ablate; change one variable per round.
- LLM reviewers are noisy instruments — their verdict is evidence, not a decision. Show A/B blind in both orders; count only defects named by 2+ reviewers; a verdict that flips with order means stop.
- Subjective calls go to the user as a side-by-side before/after, not another review round.
- Subagents don't see this file: put the criterion, cap and round count in their prompts.
- Generators and scripts live in the repo with relative paths; after a refactor or revert of deterministic output, byte-compare against the prior result.
- After any scripted edit, verify it applied (grep/diff, chained with &&) before running anything that depends on it.

## 11. Skills

**One skill per job, from Matt Pocock's set (`mattpocock-skills`); every skill's output follows §9.**

Where several skills cover the same job, use the one listed (a skill or command I invoke by name still wins):
- Debugging something broken, throwing, failing or slow: `diagnosing-bugs`
- Building a feature or fixing a bug test-first: `tdd`
- Reviewing code: `code-review`
- Researching a question: `research`
- Stress-testing, pressure-testing or grilling a plan: `grilling`

For a high-stakes or hard-to-reverse decision, offer `anthropic-skills:llm-council` in one line; run it only on my yes, or when I ask for the council.

Skills that present options (`grilling`, `grill-with-docs`, the council) use the §9 decision format; give the pick or the council's verdict when I ask for it.

Decision records: when `engineering:architecture` or `domain-modeling` writes an ADR, save it in `docs/adr/` in the §6 format and add it to `CLAUDE.md`'s ADR index.

## 12. Session Start

**Every session in a project runs the same loop: orient, report, route, close.**

1. **Orient (silently, read-only):** the project's `CLAUDE.md`, `CONTEXT.md` and ADR index; `git status`, the current branch, unmerged branches, and the backlog if one exists; and `.claude/handoff.md` if a previous session left one. Resume from the handoff, then delete it. A repo without `README.md`/`CLAUDE.md` gets §6 setup first.
2. **Report** in at most 5 bullets (branch, uncommitted work, open branches, recent commits, backlog). When my first message already names the task, go straight to step 3.
3. **Route by task:**
   - Bug from a code-review finding: fix with `tdd` (a failing test reproduces the finding first) → `code-review`
   - Bug I describe from what I see: reproduce it in the browser → fix → show me a before/after screenshot. Where the cause sits in testable logic, add a `tdd` test that fails first.
   - Other bugs (errors, failing tests, slowness): `diagnosing-bugs` → fix with `tdd` → `code-review`
   - Prototype: `prototype`, labeled throwaway → report what it answered. Keeping it makes it a feature.
   - Feature or change: `grill-with-docs` → `to-spec` → `to-tickets` (local files) → `implement` (runs `tdd` and `code-review`)
   - New project: `grill-with-docs` → `README.md`, `CLAUDE.md`, `CONTEXT.md` (§6) → `to-spec` → `to-tickets`
   - Research: `research`
4. **Close:** the project's checks pass; path rules, ADRs and `CONTEXT.md` reflect the change (§5, §6); then the §9 summary.
5. **Unfinished work:** write `.claude/handoff.md`: the goal, current state and working theory, what's left, open decisions, and the skills to call next. Point to specs, ADRs and commits by path rather than copying them, and leave out secrets. It is never committed: add it to `.gitignore` if the repo tracks `.claude/`.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.