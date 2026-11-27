# Daily Coding with Superpowers + Claude Code + Your Own Harness

> **How to use this file:** drop it into a Claude Code session and say:
> *"Use this md file for my project, building <what you're building>."*
> Claude should then follow the workflow below.

Last researched: **November 2026** (Superpowers v7.0.2, Claude Code 2.1.x).

---

## 1. The idea in one paragraph

The latest Claude Code gives you the **mechanisms**: skills, plugins, subagents, hooks,
plan mode, `/loop`, `/goal`, scheduled cloud tasks, and workflows. It does **not** give you a
**method**, meaning an opinionated order of work: design first, plan in small tasks, write the test first,
review, then prove it works before saying "done". **Superpowers** ([obra/superpowers](https://github.com/obra/superpowers))
is that method, packaged as skills that run on those same built-in mechanisms. **Harness engineering**
is the part only you can write: the rules, checks and gates specific to *your* repo. **Loops** are how
you let the agent run unattended once a plan exists.

So you don't choose one of the three. They are three layers, and each does a different job:

| Layer | Who provides it | What it answers | Examples |
|---|---|---|---|
| **Harness (runtime)** | Claude Code itself | *What can the agent do?* | tools, subagents, hooks, skills loader, plan mode, compaction |
| **Method** | Superpowers plugin | *In what order, with what discipline?* | brainstorming → plan → TDD → review → verify → finish branch |
| **Project harness** | **You** (harness engineering) | *What does "correct" mean here?* | `CLAUDE.md`, test/lint commands, hooks that block bad actions, CI gates |
| **Loop** | Claude Code `/goal`, ralph-loop plugin, headless `claude -p` | *How long does it keep going without me?* | "run until all plan tasks pass `make check`" |

The rest of this file shows how to wire them together for daily work.

---

## 2. Is this still worth it on the latest Claude Code? (verdict)

**Yes for real features and bug fixes. No for small edits.**

Why it still adds value:
- **Built-in plan mode is not a process.** Plan mode makes a plan in chat. Superpowers writes a reviewed
  spec (`docs/superpowers/specs/`) and a task-by-task plan (`docs/superpowers/plans/`) to files,
  then forces TDD, review and verification for every task. Those files survive `/clear`, compaction and new sessions.
- **The harness matters as much as the model.** 2026 studies find harness-only changes move benchmark
  scores by double digits: Terminal-Bench 2 went from 69.7% to 77.0% with harness edits alone, and the same model scored 45.9% vs 55.4% on
  SWE-bench Pro depending on scaffold ([summary](https://baeseokjae.github.io/posts/harness-design-coding-agent-performance-2026/);
  [arXiv 2609.00006](https://arxiv.org/abs/2609.00006)). Superpowers is a ready-made, heavily used
  piece of that harness. The author claims 1.1M+ installs, and it's in Anthropic's official plugin marketplace.
- **It targets Anthropic's own named failure modes for long-running agents**: claiming "done" too early,
  one-shotting too much, losing the thread. It does this with `verification-before-completion`, small planned tasks,
  and plan files ([Faros on harness engineering](https://www.faros.ai/blog/harness-engineering)).
- **It keeps up with Claude Code.** Recent releases adapted to Claude Code specifics, such as the 20,000-char skill
  re-attach after compaction, `.git/` write denial, and `AGENTS.md` handling ([release notes](https://raw.githubusercontent.com/obra/superpowers/HEAD/RELEASE-NOTES.md)).

Honest caveats:
- **Token cost.** Subagent-driven execution costs a fresh context per task and per review. The author's
  own tests found *native (inline) execution about 2x faster and half the cost* with frontier models
  ([Superpowers 6.4 post](https://blog.fsck.com/2026/09/21/superpowers-6.4/)). Default to inline.
- **Overkill for small tasks.** Typos, renames, one-file changes, throwaway prototypes, anything under ~15 minutes.
- **Evidence is mostly vendor- or practitioner-level.** One small independent test (12 sessions) found about 9% lower cost,
  more consistent results and better quality on medium and hard tasks, but *higher* cost on simple ones
  ([guide](https://pasqualepillitteri.it/en/news/215/superpowers-claude-code-complete-guide)). No peer-reviewed study yet.
- **Don't stack methodologies.** Two process frameworks (e.g. Superpowers plus a spec-kit or GSD-style plugin)
  in one context perform worse than one.

---

## 3. One-time setup (≈10 minutes per project)

### 3.1 Install Superpowers
```text
/plugin install superpowers@claude-plugins-official
```
Update later with `/plugin update superpowers`, then `/reload-plugins`.
A SessionStart hook loads the `using-superpowers` bootstrap into every session. From then on, skills fire by
description: "build X" triggers brainstorming, and "fix this bug" triggers systematic-debugging. There are no slash commands
any more (`/brainstorm` and the others were removed in v5.1). Ask for a skill by name if you want one explicitly.

### 3.2 Write your project harness (the part only you can do)
Create or extend `CLAUDE.md`. `CLAUDE.md` **overrides** Superpowers skills, so this is where you steer it.

```markdown
# CLAUDE.md

## Verify commands (the single source of truth for "done")
- Fast check:  `make check`      # lint + typecheck + unit tests, < 2 min
- Full check:  `make test-all`   # integration tests
A task is done only when the fast check passes and you have read its output.

## Superpowers settings for this repo
- Execution: use executing-plans (inline) by default. Use subagent-driven-development only
  when I say "use subagents", or when a plan has > 8 independent tasks.
- Skip the full brainstorming/plan flow for changes under ~30 lines or in docs only.
  Still apply verification-before-completion.
- Specs/plans live in docs/superpowers/ and are committed with the feature.

## Hard rules
- Never delete, skip or weaken a test to make it pass.
- Never push to main. Work in a worktree branch; finish with a PR.
- <domain rules: e.g. "all money values are integer cents", "no new dependencies without asking">
```

### 3.3 Add hooks for anything that must *never* happen
Instructions can be ignored. Hooks can't. Add these in `.claude/settings.json`, or ask Claude: "add a hook that ...".
- **PreToolUse**: block `git push` to `main`, and block edits to `migrations/` or to secrets files.
- **PostToolUse** on Edit/Write: run the formatter and linter on the changed file.
- **Stop** (optional): run `make check` and report failures back, so "done" is checked by a machine, not by the model.

Rule of thumb from harness engineering: **every time the agent makes a mistake, add a rule, test or hook so it
can't make that mistake again** (Mitchell Hashimoto's framing, via [Faros](https://www.faros.ai/blog/harness-engineering)).

---

## 4. The daily workflow

### Track A: feature or non-trivial bug (the full Superpowers flow)

| Step | You say | What happens (skill) | Your job |
|---|---|---|---|
| 1. Design | "Build <feature>." | **brainstorming**: reads the repo, asks one question at a time, proposes 2–3 approaches, writes a spec | Answer, pick an approach, **approve the spec**. Say "start right away" to shorten the questions. |
| 2. Isolate | — (automatic) | **using-git-worktrees**: new branch and worktree, runs setup and tests for a clean baseline | Check the baseline is green |
| 3. Plan | — (automatic) | **writing-plans**: plan file with exact files, interfaces and tests, in small tasks | **Read the plan.** This is the cheapest place to fix mistakes. |
| 4. Build | "Execute the plan." | **executing-plans** (inline) or **subagent-driven-development**; **test-driven-development** inside each task: failing test first | Stay nearby, or hand off to a loop (Track C) |
| 5. Review | — (automatic) | **requesting-code-review** between tasks; critical issues block | Skim severity-ranked findings |
| 6. Prove | — (automatic) | **verification-before-completion**: runs your `make check`, reads output, no "done" without evidence | — |
| 7. Finish | — | **finishing-a-development-branch**: tests, then merge / PR / keep / discard, and cleans up the worktree | Choose **PR**, review the diff |

**Your attention goes to three gates: spec approval (1), plan review (3), PR review (7).** Everything between
those gates is what Claude does on its own.

### Track B: bug hunt
Say *"Fix: <symptom + how to reproduce>."* → **systematic-debugging**: reproduce, find the root cause, then fix.
No fix comes before the root cause, and after 3 failed fixes it questions the design. It ends with a regression test
and verification. Use the full Track A only if the fix turns into a redesign.

### Track C: hand a finished plan to a loop (go do something else)
Loops are only safe **after** step 3, once a plan and a machine-checkable definition of done exist.
Pick the lightest loop that fits:

| Need | Use | Example |
|---|---|---|
| Keep working in this session until done | **`/goal`**: a separate evaluator checks the condition after every turn | `/goal All tasks in docs/superpowers/plans/<plan>.md are checked off and make check passes` |
| Fresh context every iteration (long plans, >100K tokens of work) | **ralph-loop** plugin, or headless `claude -p` in a shell loop | `/ralph-loop "Do the next unchecked task in <plan>.md using executing-plans; mark it done" --max-iterations 20 --completion-promise "PLAN COMPLETE"` |
| Recurring check, not iteration | **`/loop`** or a **scheduled cloud task** | "every morning: run the full suite on main, open an issue if red" |

Loop rules. Each one prevents a known failure mode ([agentatlas](https://agentatlas.dev/guides/agent-loops-ralph-loop-ai-coding-agents/),
[loop engineering guide](https://blakecrosley.com/guides/loop-engineering), [paddo](https://paddo.dev/blog/claude-code-loop-ralph-wiggum-evolution/)):
1. **Always cap it**: `--max-iterations`, `--max-turns` or `--max-budget-usd`. Unlimited is the default, and that's a trap.
2. **The plan file is the memory.** Each iteration reads the plan and `git log`, does *one* task, and checks it off.
3. **The doer doesn't grade itself.** "Done" = your verify command exits 0, or `/goal`'s evaluator says so.
4. **Branch only.** Loops write to the worktree branch and finish with a PR, never to `main`.
5. **Tests are read-only to the loop.** Back this with the Hard rules in `CLAUDE.md`, plus a hook if needed.

### Track D: small stuff (skip Superpowers)
Say *"Quick fix, skip the workflow: <change>."* Claude edits, runs the fast check, and shows the output. Done.

---

## 5. Instructions for Claude (when this file is given to you)

When the user gives you this file with a project description:
1. **Check setup.** Is Superpowers installed (is the `using-superpowers` skill available)? If not, tell the user the
   install command in §3.1 and wait. Does `CLAUDE.md` define verify commands? If not, propose a `CLAUDE.md` based on
   §3.2 tailored to this repo (detect its test, lint and build commands) and ask before writing it.
2. **Classify the request**: Track A (feature / multi-file), B (bug), C (an approved plan exists and the user wants it run
   unattended), or D (small edit). State the track in one line. The user can override it.
3. **Track A/B**: follow the Superpowers skills in order. Stop at the three gates (spec, plan, PR) for explicit approval.
   Default to **inline executing-plans**. Use subagent-driven-development only if `CLAUDE.md` or the user says so.
4. **Track C**: propose the exact `/goal` or `/ralph-loop` command with a cap and a machine-checkable condition
   pointing at the plan file. Don't start an uncapped loop.
5. **Always**: never claim done without running the verify command and quoting its result. Never modify tests to
   pass. Never push to main. When you hit a mistake you could have been prevented from making, suggest the `CLAUDE.md`
   rule or hook that would prevent it next time.

---

## 6. Quick decision guide

```
Is it < ~30 lines / docs only?            → Track D (no Superpowers)
Is it a bug with a reproduction?          → Track B (systematic-debugging)
Is it a feature / multi-file change?      → Track A (full flow, 3 gates)
Plan approved & I want to walk away?      → Track C (/goal or ralph-loop, capped)
Did the agent make the same mistake twice? → Add a CLAUDE.md rule, test or hook (harness engineering)
```

---

## Sources (2025–2026)

- Superpowers repo and release notes, v5.0 to v7.0.2 (Mar–Nov 2026): https://github.com/obra/superpowers, https://raw.githubusercontent.com/obra/superpowers/HEAD/RELEASE-NOTES.md
- Jesse Vincent, "Superpowers 6.4" (Sep 2026) and "what a year" (Oct 2026): https://blog.fsck.com/2026/09/21/superpowers-6.4/, https://blog.fsck.com/2026/10/09/what-a-year/
- Superpowers complete guide (updated Oct 2026): https://pasqualepillitteri.it/en/news/215/superpowers-claude-code-complete-guide
- Superpowers skills ranked (Nov 2026): https://claudemarketplaces.com/blog/superpowers-claude-code
- Barbaste et al., "Harness Engineering: Anatomy, Architecture, and Evolution of Coding Agents" (arXiv, Jul 2026): https://arxiv.org/abs/2609.00006
- Faros AI, "Harness Engineering: A Guide to AI Coding Agents" (May/Jul 2026): https://www.faros.ai/blog/harness-engineering
- "How much does the harness actually matter in 2026?" (Sep 2026, summarizes arXiv 2605.23950, 2606.12344, 2609.20804): https://baeseokjae.github.io/posts/harness-design-coding-agent-performance-2026/
- Agent loops: Ralph, /loop, /goal, headless (reviewed Oct 2026): https://agentatlas.dev/guides/agent-loops-ralph-loop-ai-coding-agents/
- "From Ralph Wiggum to /loop" (Mar/Jun 2026): https://paddo.dev/blog/claude-code-loop-ralph-wiggum-evolution/
- Loop engineering guide (Aug/Oct 2026): https://blakecrosley.com/guides/loop-engineering
