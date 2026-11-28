# Rigorous Product Engineering with pstack: Build Less, Prove More

> **How to use this file:** drop it into a Claude Code session and say:
> *"Use this md file for my project, building <your product>."*
> Claude should then add pstack's proof tools to the project and use them on the tasks where rigor pays off.

Last researched: **November 2026** (pstack 0.15.48, Claude Code 2.1.3xx, Claude Opus 5.5 / Fable 5.2). Builds on
[Posting 01](01-superpowers-daily-workflow.md) (daily build workflow) and [Posting 02](02-secure-agent-workstation.md)
(safe workstation). Pairs with [Posting 03](03-gstack-business-development.md) (deciding what to build).

---

## 1. The idea in one paragraph

Agents write code faster than anyone can read it, and AI-written pull requests carry about 1.7 times more issues
than human ones ([README L6](../README.md#l6-reviewing-ai-written-code-and-tests)). Garry Tan's gstack aims for
"a team of twenty". Lauren Tan ("poteto": React core team, ex-Meta and Netflix, now at Cursor) answered with
[pstack](https://github.com/cursor/plugins/tree/main/pstack): *"I don't want to ship like a team of twenty slop
artists... if you want to go fast, go deep first."* pstack is a free, MIT-licensed plugin of about 59 skills that
makes the agent **write less code and prove more**: drive the real app, keep only review findings a run proves,
improve a number one measured change at a time, and turn repeated mistakes into checks that fail. It's a Cursor
plugin, but Cursor ships an official Claude Code version of it. This posting takes the parts that **add** to Posting
01 and leaves out what Superpowers already does.

| What Posting 01 already gives you | What pstack adds here |
|---|---|
| `verify.sh`: tests, lint, types | **A verification skill that drives the real app like a user** and keeps a feature map (§4.1) |
| A fresh reviewer and the L6 checklist | **`/interrogate`**: several agents try to break the change; only findings proven by a command survive. **`/blast-radius`**: prove the one fact a change is safe because of (§4.2) |
| `/goal` loop until `verify.sh` passes (pass/fail) | **Hillclimb**: a loop that improves a *number* with keep-or-revert and a decision log (§4.3) |
| "Same mistake twice → add a rule or hook" (L2) | **`/correct`**: mines history for repeated mistakes and fixes each at the highest level: architecture, then types, then lint, then tests, docs last (§4.4) |
| Superpowers' spec → plan → TDD method | **`/pstack:poteto-mode`** for engineering-heavy work: perf, runtime bugs, refactors, migrations (§4.5) |

### 1.1 The idea that ties it together: proof beats claims

[L1](../README.md#l1-the-feedback-loop-the-core-idea) says the doer doesn't judge. pstack pushes that rule into
every step. Its `/blast-radius` skill has a ladder for "how sure are you?", and the same ladder fits every claim an
agent makes:

| Level | What the agent did | Counts as proof? |
|---|---|---|
| 1 | Said so ("this is safe", "tests pass") | **No** |
| 2 | Pointed at a real `file:line` | Weak |
| 3 | Walked the failure case step by step and showed it can't happen | Medium |
| 4 | **Ran it**: a failing test, a repro script, a measurement, a recorded drive of the real app | **Yes** |

Posting 01 gets "done" to level 4 for **tests**. pstack gets **reviews, safety claims, performance claims and
features in the real app** to level 4 too. Same loop, more sensors.

---

## 2. Is this worth it on the latest Claude Code? (verdict)

**Yes, for five specific tools. No, as a replacement for Superpowers or for small edits.**

- **It runs on Claude Code, officially, with known gaps.** Cursor's plugin repo has a Claude Code marketplace entry
  and a generated copy of pstack with a [Claude Code adapter](https://github.com/cursor/plugins/blob/main/pstack/docs/harness/claude-code.md)
  that maps Cursor names to Claude Code ones (`Task` → `Agent`, cloud agents → worktrees, rules →
  `~/.claude/rules/`). What doesn't carry over is listed in §3.4. Most important: in Cursor, review panels mix vendors
  (Claude, Fable, Gemini). In Claude Code the Gemini seat falls back to your session model, so panels are all-Claude
  and less independent.
- **It's expensive by default.** Every code role runs Opus 5.5 at extra-high effort, and `/arena` and `/interrogate`
  run three agents each. One developer's side-by-side test: the same task took about 30 minutes without pstack and
  about an hour with it, and the pstack run **caught three false claims** the agent had made
  ([agmazon guide, Oct 2026](https://agmazon.com/blog/articles/technology/202610/pstack-poteto-mode-guide-en.html);
  one person, one project). Set a budget (§3.2) and use it where a false claim is costly.
- **Its central habit is backed by data.** In gstack's November 2026 test, appending one sentence, *"Make the
  smallest change that fixes the issue, and nothing more"*, **halved** the non-test lines in fixes with about the same
  fix rate ([gstack evals, correction](https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-20-geva-565-skill-pack-h2h-correction/README.md)).
  pstack's first principle, the *laziness protocol* ("bias toward deletion and the smallest change"), is that sentence
  made permanent.
- **Its metric loop is the 2026 "autoresearch" pattern.** Andrej Karpathy's autoresearch loop (Mar 2026) has an
  agent change one thing, measure one number, keep the change if it improved and `git reset` if not, and log every
  try. Karpathy's two-day run found 20 improvements and an 11% training speedup; Shopify's CEO reported a 19% gain
  overnight ([Fortune, Mar 2026](https://fortune.com/2026/03/17/andrej-karpathy-loop-autonomous-ai-agents-future/)).
  pstack's **hillclimb** playbook is the same loop for product code, with a regression gate added.
- **You must type the skill.** Almost every pstack skill is set to run only when you type its name (none
  auto-trigger, so it costs little context). In gstack's test, pstack's procedure ran in 0% of sessions where nobody
  invoked it. Type `/pstack:poteto-mode` or the skill you want.
- **It disagrees with Superpowers on planning.** poteto: *"I don't believe in planning. The best spec is code."*
  pstack prototypes instead of asking and "never blocks on the human" for reversible work. Superpowers stops at a
  spec and a plan for your approval. Both are reasonable; mixing them on one task isn't. This posting keeps
  Superpowers for features where **what** to build is the open question, and uses pstack where **proof** is.

### 2.1 What do I actually need to learn?

1. The **proof ladder** in §1.1, so you can ask "which level is this claim at?"
2. The **five commands** in §4 and when each one pays off.
3. For loops: a **predicate** (a check that passes or fails) or a **metric with a target**, never a duration. *"Work
   on it for 4 hours"* gives the agent nothing to check.

You don't need the 24 principle skills or 22 playbooks; `/pstack:poteto-mode` picks them.

---

## 3. One-time setup (≈15 minutes)

### 3.1 Install the official Claude Code version

```text
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

- Commands get a prefix in Claude Code: `/poteto-mode` becomes **`/pstack:poteto-mode`**.
- **Leave auto-update off** (the default for third-party marketplaces). That matches Posting 02's rule: update on
  purpose with `claude plugin update pstack@cursor-plugins` after reading what changed.
- Read before trusting ([Posting 02 §3.6](02-secure-agent-workstation.md)). pstack ships skills, two subagents
  (`pstack:poteto-agent`, `pstack:comment-sicko`) and some helper scripts; it registers **no hooks** in this
  version. Community ports exist (for example `michael-denyer/pstack-claude`, which adds an auto-routing hook); this
  posting uses the official one.

### 3.2 Pick models and a budget

Run `/pstack:setup-pstack`. It writes `~/.claude/rules/pstack-models.md`, which Claude Code loads into every session.

| Budget | Effort for code roles | Use when |
|---|---|---|
| `large` | xhigh (the default) | Hard bugs, perf work, migrations where a wrong claim is expensive |
| **`medium`** | high | **Recommended start** for a solo product |
| `small` | medium | Routine work, tight budget |

Fable 5.2 runs only in review panels or when two Opus runs of the same brief fail, so it doesn't run up the bill on
its own. If you have a Codex login and want a different vendor in reviews, ask for an outside review explicitly.

### 3.3 Add the overrides to your project harness

pstack's defaults are tuned for a Cursor engineer at work: it proceeds without asking on reversible work, may post
to team chat or tickets, and can merge a stack overnight. Posting 02's rules win. Add to `CLAUDE.md`:

```markdown
## pstack (Posting 04)
- One method per task: Superpowers (Posting 01) for features where WHAT to build is open; /pstack:poteto-mode for perf,
  runtime bugs, refactors and migrations. Never both on one task.
- Proof standard: a claim is done only at level 4 (it was run: test, repro, measurement, recorded drive). Quote the output.
- External actions (chat, tickets, emails, comments) need my approval, whatever a skill says.
- Unattended runs: branch or worktree only, autopilot-stack (no merge), never autopilot-full, never "land even if CI flakes".
- Loops need a predicate or a metric + target + attempt floor, never a duration. Same failure 3x → NEEDS-HUMAN.md.
- Steps that need cursor-team-kit (/deslop, control-ui, control-cli): use /simplify and our verify-<app> skill instead.
```

### 3.4 What doesn't carry over to Claude Code (honest list)

| Cursor feature pstack uses | In Claude Code |
|---|---|
| Mixed-vendor panels (Gemini seat) | Gemini seat runs on your session model, so panels are Claude-only. Less independence |
| Cursor Automations (the "benny" Slack triage pack) | Not available; skipped with a one-line note |
| Bugbot, Cursor cloud agents, built-in PR tool | Bugbot steps skipped; cloud agents become local worktrees; PRs via `gh` |
| `cursor-team-kit` (`/deslop`, `control-ui`, `control-cli`) | Not in the Claude Code marketplace; use `/simplify` and a verification skill (§4.1) |
| "Custom mode" that keeps poteto-mode on every turn | Type `/pstack:poteto-mode` at the start of each task |
| Cursor's `/loop` | Claude Code's `/goal` (predicate) or `/loop` (heartbeat), from Posting 01 |

---

## 4. The five tools, and when to use each

### 4.1 Build the proof: `/pstack:create-verification-skill` (once per project, ≈30 min)

Unit tests show branches work. They don't show a user can sign up. This skill studies the repo and generates a
project skill, `.claude/skills/verify-<app>/`, that tells any future agent how to **launch the app from an empty
state, check it's healthy, drive it the way a user does** (browser, terminal or HTTP), **capture evidence**
(screenshots, transcripts, response bodies, database rows) and clean up. It also seeds a **feature map**: one file per
user-facing feature with how to reach it and what end state proves it works. Its proof rules: drive the real user
path, not test-only endpoints; check side effects, not just the screen; never weaken a journey to make it pass.
Before handing over, it runs its own instructions once end to end. *"A generated skill that was never executed is a
draft, not a deliverable."*

Then connect it to Posting 01's sensor:
> *"Add the top 3 feature-map journeys to `verify.sh` as end-to-end checks (Playwright for web), and make the Stop
> gate run them."*

Run `/pstack:maintain-verification-skill` when the feature map drifts from the app.

**Worth it when:** the product has a UI, CLI or API a user touches. That's almost every product.

### 4.2 Prove the review: `/pstack:interrogate` and `/pstack:blast-radius` (before merging risky PRs)

`/pstack:interrogate` restates the change's intent, then sends several adversaries to **break it**. Each finding must
come with the exact command, its output, and the output the intent requires. Findings nobody ran are dropped, and so
are style, naming and "risks that need a future change to matter". It reports findings; it doesn't fix them.

`/pstack:blast-radius` asks what a small-looking change could break elsewhere, finds the one or two facts its safety
depends on, and **proves them by running code** instead of writing a convincing paragraph.

**Worth it when:** money, auth, data migrations, concurrency, or anything you'd be embarrassed to roll back. Skip it
on copy changes. This replaces "trust the green check" with level-4 evidence; it doesn't replace your own L6 review.

### 4.3 Improve a number: the hillclimb loop

For *"make the dashboard load faster"*, *"cut the bundle size"*, *"raise the matcher's accuracy"*: anything with a
number. Type `/pstack:poteto-mode hillclimb <metric>: <target>`. The playbook:

1. Finds a workload that reproduces the complaint, then fixes **one metric, its direction, and a stop predicate with
   a target and an attempt floor** (e.g. *"≥50% faster than baseline and at least 10 attempts"*, so a lucky first win
   can't end it).
2. Builds a measurement harness, proves it can tell a slow case from a fast one, then **freezes** it. One command
   prints the metric (median of N runs) plus an error count. Records the baseline and a green regression gate.
3. Loops, one hypothesis at a time, each naming a mechanism ("defer X off the startup path because it blocks first
   paint"): change, measure, run the regression gate. **Keep only if the number moves past the noise and the gate is
   green; otherwise revert completely.** One commit per kept win. Every try, kept or reverted, is a row in a
   `decision.tsv`.
4. Pushes past the first plateau with new categories of ideas, never relaxes the predicate, and stops when it's met or
   the remaining ideas aren't worth their cost.

Run it in a branch or worktree. For an unattended run, wrap it in `/goal` with the predicate (Posting 01, Track C).
Before you trust any number it reports, `/pstack:benchmark-checklist` vets it (what limits it, errors, repeat runs).

### 4.4 Make mistakes impossible: `/pstack:correct` (weekly, or after you correct Claude twice)

It reads recent commits, reverts and review comments, groups repeated mistakes into classes (a class counts once it
happened twice), and fixes each at the **highest level that works**:

1. **Architecture**: one owner for each piece of state, one supported way per task, delete the old way an agent would copy.
2. **Types**: make the bad state impossible to write. If it still compiles, a **lint or CI check whose error names the fix**.
3. **Tests** that fail when the behavior breaks.
4. **Docs or `CLAUDE.md` rules last**, only for judgment calls, because nothing fails when an agent skips them.

It proves each new check fails on a real past mistake and keeps a table in `CLAUDE.md` pairing each rule with what
enforces it. This is Posting 01's "add a rule, test or hook" habit, done systematically.

### 4.5 Engineering-heavy tasks: `/pstack:poteto-mode`

Use it **instead of** Superpowers for one task when the hard part is engineering rather than deciding what to build:

| Your task | poteto-mode playbook |
|---|---|
| "It's slow" | perf issue (trace, baseline, improve) or hillclimb |
| A live leak, CPU spin, flicker | runtime forensics |
| A bug that needs runtime evidence | bug fix: reproduce on the real surface, binary-search the cause, failing repro committed before the fix |
| Restructure without changing behavior | refactoring |
| Migrate every caller to a new API | multi-phase plan, or figure-it-out for big unattended runs |
| Two designs, can't decide | prototype (build 2–3 throwaway versions and compare) |
| Pick up someone's half-done work | session pickup |

Give it a result and a finish condition, not a list of skills: *"/pstack:poteto-mode the export takes 9 s for 10k
rows. Done means under 2 s on the seeded dataset with all tests green."* For long or unattended work it keeps a
decision log with `/pstack:show-me-your-work`, which you read in the morning (`/pstack:show-me-your-work catch me up`).

Small, cheap helpers you can type any time: `/pstack:how` (walk through a subsystem), `/pstack:why` (why it was built
this way, from commits and PR discussions), `/pstack:teach` (explain a change diagram by diagram),
`/pstack:reframe` (learn the expert terms for what you're building).

---

## 5. Instructions for Claude (when this file is given to you)

1. **Check setup.** Is pstack installed (`/pstack:poteto-mode` available)? If not, show §3.1 and wait. Is there
   `~/.claude/rules/pstack-models.md`? If not, suggest `/pstack:setup-pstack` with the `medium` budget. Does
   `CLAUDE.md` have the §3.3 overrides? If not, propose them and ask before writing.
2. **Read the Claude Code adapter** (`docs/harness/claude-code.md` inside the installed plugin) before running any
   pstack skill, and map Cursor names with it. When a step needs a Cursor-only feature, write one line `Skipped in
   Claude Code: <step>. <reason>.` and continue.
3. **Proof first.** If the project has no `verify-<app>` skill, recommend `/pstack:create-verification-skill` before
   other pstack work, then wire its top journeys into `verify.sh`.
4. **Pick one method per task** (§3.3 rule 1). State the choice in one line; the user can override.
5. **Use pstack's heavy tools only where they pay:** `/pstack:interrogate` and `/pstack:blast-radius` for money,
   auth, data, concurrency or wide changes; hillclimb only with a frozen metric and a target. Say the expected cost
   (number of agents) before starting `/arena`, `/swarm` or `/interrogate`.
6. **Posting 02 wins over pstack's autonomy defaults:** ask before any message outside the session, never merge
   unattended, never push to main, branch or worktree only.
7. **Every claim at level 4.** Quote the command and output for "done", "faster", "safe" and "fixed". Never relax a
   predicate or metric target to finish.

---

## 6. Quick decision guide

```
New project, no way to drive the real app?        → /pstack:create-verification-skill, then wire into verify.sh
Feature where WHAT to build is still open?        → Superpowers (Posting 01), not pstack
"Make it faster / smaller / more accurate"?       → hillclimb: frozen metric, target, keep-or-revert, decision log
Perf, runtime bug, refactor, migration?           → /pstack:poteto-mode with a "done means" condition
Risky PR (money, auth, data, concurrency)?        → /pstack:interrogate + /pstack:blast-radius before merge
Corrected Claude for the same thing twice?        → /pstack:correct (architecture > types > lint > tests > docs)
Leaving it overnight?                             → worktree + predicate + show-me-your-work, autopilot-stack (no merge)
One-line fix or copy change?                      → Neither. Just do it (Posting 01 Track D)
```

---

## Sources (2025–2026)

- pstack plugin, README, skills and playbooks (v0.15.48, Nov 2026): https://github.com/cursor/plugins/tree/main/pstack
- pstack Claude Code adapter and generated harness tree: https://github.com/cursor/plugins/blob/main/pstack/docs/harness/claude-code.md · https://github.com/cursor/plugins/tree/main/harness/pstack
- pstack guide, "Run work while you sleep" (overnight contract, decision log): https://github.com/cursor/plugins/blob/main/pstack/docs/guide/07-overnight.md
- Flavio Copes, "What is poteto-mode in pstack?" (Oct 2026): https://flaviocopes.com/poteto-mode/
- "pstack explained: setup, poteto-mode, costs" (Oct 2026): https://agmazon.com/blog/articles/technology/202610/pstack-poteto-mode-guide-en.html
- "What are the poteto skills for Claude Code?" (Nov 2026): https://skillselion.com/guides/poteto-skills-claude-code-guide
- gstack evals, skill pack head-to-head and correction (includes pstack and the "smallest change" sentence, Nov 2026): https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-19-geva-540-skill-pack-h2h-results/README.md · https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-20-geva-565-skill-pack-h2h-correction/README.md
- Fortune, "The Karpathy loop" (Mar 17, 2026): https://fortune.com/2026/03/17/andrej-karpathy-loop-autonomous-ai-agents-future/
- Davide Gallitelli, "The autoresearch pattern" (Mar 2026): https://dgallitelli.github.io/blog/autoresearch-week/day1-the-autoresearch-pattern/
- SkillsBench (curated, focused skills help; arXiv 2602.12670, 2026): https://arxiv.org/abs/2602.12670
- CodeRabbit, AI vs human pull requests (Dec 2025): https://www.helpnetsecurity.com/2025/12/23/coderabbit-ai-assisted-pull-requests-report/
