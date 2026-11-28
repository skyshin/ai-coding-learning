# Must-Have Project Files: The Smallest Set of Docs an Agent Needs, by Project Size

> **How to use this file:** drop it into a Claude Code session and say:
> *"Use this md file for my project, building <your product>."*
> Claude should then pick the project's tier, list which files exist and which are missing, and draft only the missing
> ones from the templates below, with no fact written in two places.

Last researched: **November 2026** (Claude Code 2.1.3xx, Superpowers v7.0.2, pstack 0.15.48). The evidence and the
step-by-step reasoning behind every choice here are in
[research/05-must-have-project-files-research.md](../research/05-must-have-project-files-research.md).
Builds on [Posting 01](01-superpowers-daily-workflow.md) (harness and build workflow),
[Posting 02](02-secure-agent-workstation.md) (safe workstation), [Posting 03](03-gstack-business-development.md)
(what to build) and [Posting 04](04-pstack-rigorous-product-engineering.md) (proof).

---

## 1. The idea in one paragraph

An agent knows only what is in the repo, and it starts every session with no memory. So the repo's files *are*
the project's memory, its rules and its definition of "done". But more files isn't better. A 2026 ETH Zurich study
found that typical context files didn't raise agents' success rates and raised cost by over 20%, and files an LLM wrote
by itself made results slightly *worse*. What helped were short, human-written instructions the agent couldn't get
from the code. So the goal is the **smallest set of files where each one answers one question nobody can answer by
reading the code**, plus a check that enforces it. A 3-page web app needs four of them. A large system needs about
twelve. Nobody needs thirty.

### 1.1 Five rules that decide every file

| # | Rule | What it means in practice |
|---|---|---|
| 1 | **Only what the code can't say** | No directory trees, dependency lists or "this is a React app" overviews. Write decisions, rules, words, commands and intent |
| 2 | **One fact, one home** | Each fact lives in exactly one file. Other files **link** to it. Never copy |
| 3 | **Never-rules are checks, not prose** | A doc explains the rule. A test, hook or CI check enforces it (README L2, enforcement ladder) |
| 4 | **Add a file when its trigger fires** | Don't create the large-project set on day 1. Each file in §3 has a "create when" line |
| 5 | **A human reviews every doc an agent drafts** | An unreviewed, agent-written glossary or context file is worse than none |

### 1.2 How the files fit the feedback loop

The files aren't a pile of paperwork. Each one feeds a part of the loop from
[README L1](../README.md#l1-the-feedback-loop-the-core-idea):

| Loop part | Files | Question |
|---|---|---|
| **Intent** (why build it) | `PRD.md` | What, for whom, why, and what's out of scope? |
| **Target** (what "done" means) | feature specs, `tests/acceptance/` | What exactly must be true? |
| **Sensor** (is it done?) | `verify.sh`, verification skill | One command that proves it |
| **Rules** (what the action may do) | `AGENTS.md`, `ARCHITECTURE.md`, `GLOSSARY.md`, `SECURITY.md`, ADRs | What must never be crossed, and what our words mean |
| **Operate** (keep it running) | `README.md`, `RUNBOOK.md` | How to run, deploy, roll back and recover |

---

## 2. Is this worth it on the latest Claude Code? (verdict)

**Yes. It's the cheapest posting in this repo to apply, and it removes waste rather than adding work.**

- Claude Code still starts each session from your files: `CLAUDE.md` (or `AGENTS.md`), rules and skills.
  Better models didn't remove that; they made it *more* important that those files are short and correct, because
  agents now follow them closely (the ETH study saw agents obey the files even when that cost extra steps).
- **New in Sep 2026:** Claude Code reads `AGENTS.md` (v2.1.277+), but only when there's no `CLAUDE.md`. That makes
  duplication easy to get wrong. §3.2 gives the one setup that works everywhere.
- **What it doesn't do:** it doesn't replace Superpowers' spec and plan files, `verify.sh`, or Posting 02's security.
  It tells you which of those you need at your size and where each fact lives.
- **Not worth it for:** throwaway prototypes and one-day builds. Use a `README.md` and Posting 01 Track D.

### 2.1 Pick your tier (2 minutes)

Tier by **risk and the number of minds working on it**, not by lines of code.

| Tier | Typical shape | Example |
|---|---|---|
| **S: Small** | One person, one deployable, a few screens, one database, months of life ahead but low stakes | A 3-page booking site, an internal dashboard, a small mobile app with a simple API |
| **M: Medium** | Several modules or an external integration, 2–5 people *or* several agents in parallel | A SaaS app with billing, a marketplace, a B2B tool with SSO |
| **L: Large** | Several services or bounded contexts, several teams, long-running agent loops, or regulated | A payments platform, a health-records system, a monorepo with 10+ packages |

**Move up one tier if any of these is true:** it handles money · it stores health, financial or other sensitive
personal data · it's regulated · it has more than one deployable · several agents or people change it in parallel.
A 3-page app that takes payments is **M**, not S.

---

## 3. The files

### 3.0 The map: what you need at each size

| File | S | M | L | Answers | Create when |
|---|:-:|:-:|:-:|---|---|
| [`README.md`](#31-readmemd-for-humans) | ● | ● | ● | What is it, how do I run, test and operate it? | Day 1 |
| [`AGENTS.md` + `CLAUDE.md` shim](#32-agentsmd-and-a-one-line-claudemd) | ● | ● | ● | What must an agent know that the code can't tell it? | Day 1 |
| [`docs/PRD.md`](#33-docsprdmd-what-and-why) | ● 1 page | ● | ● | What are we building, for whom, why, what's out? | Before the first feature |
| [`verify.sh` + `tests/acceptance/` + hooks](#34-verifysh-and-the-harness) | ● | ● | ● + CI | How do we prove it's done? | Day 1 (Posting 01 §3.3) |
| [Feature specs + plans](#35-feature-specs-and-plans-sdd) | ◐ | ● | ● | How will this feature work, step by step? | Per feature, when §3.5's test says so |
| [`ARCHITECTURE.md`](#36-architecturemd) | — | ● | ● | Where do things live; which lines must never be crossed? | 2nd module, or 2nd person/agent |
| [`docs/adr/`](#37-docsadr-decision-records) | — | ◐ | ● | Why is it built this way? | First hard-to-reverse decision |
| [`GLOSSARY.md`](#38-glossarymd-the-ontology-done-simply) | — | ◐ | ● | What exactly do our words mean? | The agent or a person uses two names for one thing |
| [`SECURITY.md`](#39-securitymd-and-threat_modelmd) | ◐ | ● | ● + `THREAT_MODEL.md` | What must we protect, from whom, and how? | Real user data or money |
| [`RUNBOOK.md`](#310-runbookmd) | ◐ | ● | ● | How do we deploy, roll back, recover? | Before first production deploy |
| [`.claude/rules/*.md`](#311-clauderules-for-one-area-only) | — | ◐ | ● | Conventions that apply to one folder only | `AGENTS.md` passes ~150 lines |
| [Verification skill](#312-the-verification-skill-pstack) | — | ● | ● | How does an agent drive the real app as proof? | Once the app has a UI or API worth driving |
| [`docs/progress.md`](#313-docsprogressmd-only-during-long-loops) | ◐ | ◐ | ◐ | Where did the last session stop? | Only during multi-session loops |

● must-have · ◐ conditional (see "create when") · — skip at this size

**In words:**
- **S (a 3-page app):** `README.md` (with an *Operate* section), `AGENTS.md` (with 5 security lines) + the
  `CLAUDE.md` shim, a one-page `PRD.md`, and `verify.sh` with locked acceptance tests and hooks. Four things.
- **M:** add `ARCHITECTURE.md`, `SECURITY.md`, `RUNBOOK.md`, feature specs for real features, a verification skill,
  and ADRs or a glossary when their triggers fire.
- **L:** all of it, plus `THREAT_MODEL.md`, per-area rules, a glossary map, `verify.sh` as a required CI check,
  `CODEOWNERS` on agent config, and a scheduled doc-freshness run (§4.3).

### Files you don't need (they duplicate something above)

| Skip | Because | Put it here instead |
|---|---|---|
| `SDD.md`, `constitution.md` | SDD is a *process*, not a file. It produces per-feature specs | Specs: §3.5. Project-wide rules: `AGENTS.md` |
| `Ontology.md` (OWL/RDF) | No 2026 coding-agent source recommends a formal ontology for agent context | `GLOSSARY.md` (§3.8) |
| `DESIGN.md`, per-feature `design.md` | System design = architecture; feature design = part of the spec | `ARCHITECTURE.md` + the feature spec |
| Both a `CLAUDE.md` *and* an `AGENTS.md` with content | Two copies drift, and Claude Code silently ignores `AGENTS.md` when `CLAUDE.md` exists | §3.2 |
| A second PRD after gstack | gstack's design doc (Posting 03) already answers "what and why" | Move it to `docs/PRD.md` |
| `init.sh`, a JSON feature list, `claude-progress.txt` | Anthropic's long-running pattern; here `verify.sh`, `tests/acceptance/` and the plan file do the same jobs, and the tests are hook-protected | §3.4, §3.5, §3.13 |
| `TODO.md`, `PLANS.md` | The Superpowers plan (or your issue tracker) is the task list | Plan file |
| `CONTRIBUTING.md`, `CHANGELOG.md` | No unique content until you have outside contributors or published releases | README / git history |
| `/init`-generated `CLAUDE.md` you didn't edit | LLM-written context files slightly *lower* success rates | Write it from §3.2 and review it |
| `.cursorrules`, `copilot-instructions.md` copies | `AGENTS.md` is read by Codex, Cursor, Copilot, Gemini CLI and others | `AGENTS.md` |

Suggested layout (M tier; S has only the starred lines):

```
README.md            *
AGENTS.md            *    shared agent instructions (the content)
CLAUDE.md            *    @AGENTS.md + Claude-only lines
ARCHITECTURE.md
GLOSSARY.md               (when triggered)
SECURITY.md
RUNBOOK.md
verify.sh            *
tests/acceptance/    *    locked target
.claude/settings.json *   hooks (Posting 01 §3.3, Posting 02 §3.3–3.4)
.claude/rules/            (when AGENTS.md gets long)
.claude/skills/verify-<app>/   (pstack verification skill)
docs/PRD.md          *
docs/adr/0001-*.md        (when triggered)
docs/superpowers/specs/   feature specs (Superpowers writes these)
docs/superpowers/plans/   plans (Superpowers writes these)
```

---

### 3.1 `README.md`: for humans

**Answers:** what this is, how to run it, how to test it, how to operate it. Written for a person landing in the
repo. The agent reads it too, so `AGENTS.md` doesn't repeat it.
**At S, the *Operate* section replaces `RUNBOOK.md`.** At M+, it's one line linking to `RUNBOOK.md`.

```markdown
# <Product name>

<One sentence: what it does and for whom.> Details: [docs/PRD.md](docs/PRD.md).

## Run it locally
<prerequisites with versions>
<3–5 commands from clone to running app>

## Test it
`./verify.sh`  (lint, types, unit and acceptance tests; exit 0 = pass)

## Operate it            <!-- S tier only; at M+ replace with: see RUNBOOK.md -->
- Deploy: <command or "merge to main deploys via <CI>">
- Roll back: <command>
- Logs / errors: <where>
- Backups: <what, how often, last restore test date>

## Where things are documented
AGENTS.md (agent rules) · docs/PRD.md (what and why) · ARCHITECTURE.md (M+) · SECURITY.md (M+)
```

### 3.2 `AGENTS.md`, and a one-line `CLAUDE.md`

**Answers:** what an agent must know that it can't learn from the code: exact commands, hard rules, project
conventions it would otherwise guess wrong, and a map of where the other docs are.
**Why `AGENTS.md` holds the content:** it's the vendor-neutral file other agents (Codex, Cursor, Copilot, Gemini CLI)
read, and Claude Code reads it too. But Claude Code reads `AGENTS.md` **only if there's no `CLAUDE.md`**, and the
setting to load both can't be committed to a repo. So use the pattern Claude Code's docs recommend: content in
`AGENTS.md`, and a tiny `CLAUDE.md` that imports it. This works on every Claude Code version and setting.
(Only ever use Claude Code? A single `CLAUDE.md` with the same content is fine. Just never put content in both.)

**Size:** aim for under ~100 lines, hard cap 200. It's a table of contents, not an encyclopedia (OpenAI's harness
team). Every line must pass Claude Code's test: *"Could the agent find this by reading the code, and would it
actually look?"* If yes, delete the line.

`AGENTS.md`:

```markdown
# AGENTS.md

## Commands
- Verify (the definition of done): `./verify.sh`   # exit 0 = done
- One test file: `<command> path/to/test`
- Run the app: `<command>`

## Done means
A task is done only when ./verify.sh exits 0. tests/acceptance/ and verify.sh are the locked target:
never edit them. If the target looks wrong, stop and ask.

## Hard rules
- Never push to main. Branch or worktree, then a PR.
- Never delete, skip or weaken a test to make it pass.
- No new dependency without asking.
- <2–5 domain rules the code can't show: "money is integer cents", "all dates in the user's time zone">

## Security (S tier: these lines are your security doc; M+: see SECURITY.md)
- Never read or print .env*, keys or tokens. Secrets come from <secret store>, never from code.
- Every query on user data filters by the signed-in user (or uses row-level security).
- Validate all input at the API boundary. Never build SQL or shell commands from strings.

## Where to look (read when relevant, don't preload)
- What and why: docs/PRD.md · Feature specs/plans: docs/superpowers/
- Module map and boundaries: ARCHITECTURE.md · Decisions: docs/adr/ (accepted = binding; superseded = ignore)
- Domain words: GLOSSARY.md (use these names exactly) · Security: SECURITY.md · Operations: RUNBOOK.md
```

`CLAUDE.md`:

```markdown
@AGENTS.md

## Claude Code only
- Superpowers: <settings from Posting 01 §3.2, e.g. "inline execution by default; skip the flow for < 30 lines">
- Hooks enforce the locked target and the Stop gate (.claude/settings.json). If NEEDS-HUMAN.md appears, stop.
```

**Protect both files.** OWASP's 2026 AI cheat sheet says to treat agent rules files as security-critical
configuration that the agent may not change without approval. Add them to the `ASK` list of Posting 02's
`protect-config.py` hook: `r"(^|/)(AGENTS|CLAUDE)\.md$"` and `r"(^|/)\.claude/rules/"`.

**Gotchas:** a stray `CLAUDE.local.md` or a `CLAUDE.md` in a parent folder makes Claude Code skip a directly read
`AGENTS.md` without warning; the `@AGENTS.md` import avoids this. Claude Code's Explore and Plan subagents start
without these files, so a rule a subagent must obey belongs in a hook or in the skill that starts it.

### 3.3 `docs/PRD.md`: what and why

**Answers:** what we're building, for whom, why it matters, how we'll know it worked, and what's out of scope.
A PRD is about *intent*. A spec (§3.5) is about *how one feature works*. The PRD changes rarely; specs come and go.
**If you ran gstack (Posting 03):** its design doc *is* this file. Move it to `docs/PRD.md`, keep its KILL/CONTINUE line,
and don't write a second one.
**Size:** S = one page. M/L = 2–5 pages; split big areas into linked sub-PRDs (`docs/prd/<area>.md`) rather than
one long file.

```markdown
# PRD: <product>          Status: draft | approved (<date>, <who>)

## Problem and user
<Who has the problem, how they solve it today, why that's bad. Evidence: interviews, data.>

## Goal and success metric
<One measurable outcome and a date. "20 paying teams by Mar 31", not "users love it".>
KILL/CONTINUE: <dated test from Posting 03, if you have one>

## Scope: what v1 does (user stories)
1. As a <user>, I can <action> so that <value>.
   - Acceptance: WHEN <event> THE SYSTEM SHALL <result>.   (becomes a test in tests/acceptance/)
2. ...

## Non-goals (v1 will NOT)
- <things people will assume are included>

## Constraints
- Stack/platform: <...> · Budget/performance: <p95 < 300 ms, < $50/month hosting>
- Data and privacy: <what personal data, where it may live, retention>
- Must not change: <existing behavior, APIs or files that are off limits>

## Open questions
- <question> (owner, due date)
```

The **non-goals** and **must not change** lines do more for an agent than any other part. Agents otherwise
"helpfully" build or change things nobody asked for (Crosley, 30+ agent PRDs, 2026).

### 3.4 `verify.sh` and the harness

**Answers:** how do we prove it's done? This is the sensor from README L1, and it's already fully specified in
[Posting 01 §3.3](01-superpowers-daily-workflow.md) (sensor, locked `tests/acceptance/`, target-lock hook, Stop gate,
3-strikes `NEEDS-HUMAN.md`). This posting doesn't repeat it. What changes by tier:

| Tier | `verify.sh` runs | Also |
|---|---|---|
| S | lint, typecheck, unit, acceptance tests, `check-docs` (below) | Posting 01 hooks |
| M | + the top 1–3 journeys of the verification skill (§3.12), dependency audit | `verify.sh` runs in CI on every PR |
| L | + security scan (e.g. Semgrep), contract tests between services | `verify.sh` is a **required** CI check; `CODEOWNERS` on `AGENTS.md`, `CLAUDE.md`, `.claude/`, CI and deploy files |

Add this to `verify.sh` at every tier, so docs can't silently point at things that no longer exist:

```bash
# check-docs: fail if AGENTS.md grows past its budget or docs reference missing paths
lines=$(wc -l < AGENTS.md); [ "$lines" -le 200 ] || { echo "AGENTS.md is $lines lines (max 200)"; exit 1; }
for f in AGENTS.md CLAUDE.md README.md ARCHITECTURE.md SECURITY.md RUNBOOK.md; do
  [ -f "$f" ] || continue
  grep -oE '\(([A-Za-z0-9_./-]+\.(md|sh|json|ts|py))\)|`([A-Za-z0-9_-]+/[A-Za-z0-9_./-]+)`' "$f" \
    | tr -d '()`' | sort -u | while read -r p; do
      [ -e "$p" ] || { echo "$f references missing path: $p"; exit 1; }
    done || exit 1
done
```

### 3.5 Feature specs and plans (SDD)

**Answers:** for one feature, what exactly must be true and how we'll build it, step by step.
**No separate SDD file or framework.** Superpowers (Posting 01) already runs spec-driven development: it writes the
spec to `docs/superpowers/specs/` and the plan to `docs/superpowers/plans/`, and you approve each. Adding Spec Kit,
Kiro or OpenSpec on top would give you two methods for one job.

**When to write a spec** (from the only dated 2026 SDD benchmark, which found specs paid off on features and large
tickets but not on small fixes): write one when **3 or more** of these hold:
3+ files change · 4+ acceptance criteria · a module or service boundary is crossed · a schema or public API changes ·
more than one person or agent works on it · the code is regulated, or touches money or auth.
Otherwise use Posting 01 Track D (just do it; the Stop gate still runs `verify.sh`).

| Tier | Specs |
|---|---|
| S | Only for the 1–2 risky features (auth, payments, data deletion). Everything else is Track D |
| M | Every real feature (Track A) |
| L | Every feature, plus a spec for any cross-service contract change, reviewed by the owning team |

Make sure every spec has these, whatever tool writes it (Superpowers' template covers most):

```markdown
## Acceptance criteria        (each one becomes a test in tests/acceptance/)
- WHEN <event> THE SYSTEM SHALL <result>.
## Out of scope
## Files NOT to modify
## Links: PRD story #<n> · ADRs: <ids> · Glossary terms used: <terms>
```

### 3.6 `ARCHITECTURE.md`

**Answers:** where the main parts live, how data flows between them, and which lines must never be crossed.
**Not** a directory listing or a list of dependencies (the agent can read those, and Claude Code's `/doctor`
flags them). Keep it to what a newcomer can't see in an hour: boundaries, invariants, and where things are *not*.
**Create when:** the second module or service appears, or a second person or parallel agent joins. **Size:** 1–3 pages.

```markdown
# Architecture          Last checked against the code: <date>

## The big picture
<5–10 lines or one small diagram: the main parts and how a request flows through them.>

## Parts and boundaries
| Part | Lives in | Owns | May call | Must NOT |
|---|---|---|---|---|
| web | apps/web | UI, routing | api only | talk to the database directly |
| api | apps/api | business rules, auth checks | db, payments adapter | import from web |
| payments adapter | packages/payments | Stripe calls | Stripe | be called from anywhere but api |

## Invariants (true everywhere, always)
- <"Every table with user data has row-level security on.">  Enforced by: <test/lint/hook>
- <"Money is integer cents end to end.">  Enforced by: <type Money + lint rule>

## Where things are NOT
- <"There is no cache layer; don't add one without an ADR.">

## Decisions behind this: docs/adr/ (0001, 0004, 0007)
```

Each invariant names what **enforces** it (rule 3). If nothing does, that's a task: add a test or lint rule.

### 3.7 `docs/adr/`: decision records

**Answers:** why it's built this way, so an agent (or you, in six months) doesn't "fix" a deliberate choice.
**Create one only when all three are true:** the decision is hard to reverse, surprising without context, and the
result of a real trade-off. Most choices don't qualify.
**Two failure modes to avoid** (O'Reilly Radar, Oct 2026): agents rigidly follow outdated "accepted" ADRs, so mark
replaced ones `Superseded`; and agents over-edit ADRs into long logs, so keep them short and let git hold the history.

`docs/adr/0003-use-postgres-row-level-security.md`:

```markdown
# 0003: Enforce tenant isolation with Postgres row-level security

Status: accepted | superseded by 00NN       Date: 2026-11-28
Supersedes: —

Context: <2–3 sentences: the forces and constraints.>
Decision: <1–2 sentences: what we chose.>
Why: <the trade-off; what we gave up.>
Consequences for agents: <the rule this creates, e.g. "never query user tables with the service role key".>
Revisit if: <condition, e.g. "we move off Postgres".>
```

`AGENTS.md` already says "accepted = binding; superseded = ignore". That one line is what makes ADRs work for agents.

### 3.8 `GLOSSARY.md`: the ontology, done simply

**Answers:** what exactly our words mean, so the agent doesn't invent a new name each session (`Customer`, `Client`,
`Account` for one thing) or merge two things that only sound alike.
**Why not a formal ontology:** no 2026 coding-agent source recommends OWL/RDF for agent context; all recommend a
plain glossary. A formal ontology is a *product feature* (knowledge graphs, data integration), not a project doc.
**Create when:** the domain has words that mean something specific here (billing, healthcare, logistics, finance), or
you catch two names for one concept. Skip it for a one-day build or a domain with no special vocabulary.
**L tier:** one `GLOSSARY.md` per bounded context (e.g. `services/billing/GLOSSARY.md`) plus a root `GLOSSARY-MAP.md`
listing the contexts and the words that mean different things in each.
**Rule:** update the glossary *before* the code when you rename or ban a term. You review every entry an agent adds.

```markdown
# Glossary: <product>

## Terms (use these names exactly in code, tests, UI and docs)
**Subscription**: a customer's paid plan for one workspace, billed monthly or yearly.
  _Avoid_: plan (that's the price tier), membership.
**Workspace**: the unit of tenancy. All data belongs to exactly one workspace.
  _Avoid_: account, org, team.
**Seat**: one user's right to use a workspace under a subscription.

## Relationships and invariants
- A workspace has 0 or 1 active subscription. A subscription has 1..n seats.
- A user may belong to many workspaces but holds at most one seat in each.

## Flagged ambiguities
- "Account" means *login identity* in auth code and *workspace* in old billing code. New code: never use "account" for workspace.
```

### 3.9 `SECURITY.md` and `THREAT_MODEL.md`

**Answers:** what we must protect, from whom, and the rules and checks that do it. Your *machine's* security is
[Posting 02](02-secure-agent-workstation.md); this file is your *product's*.
**S:** the five security lines in `AGENTS.md` (§3.2) are enough. **M:** a `SECURITY.md` with a short threat-model
section. **L:** split the threat model into `THREAT_MODEL.md` and give each major control its own section.
Write it for a machine reader: each rule specific enough for an agent to check (VirtusLab, Nov 2026).

```markdown
# Security

## Reporting
Email <address>. Don't open a public issue.

## What we protect (data classes)
| Data | Example | Where it may live | Who may read it |
|---|---|---|---|
| Secrets | API keys | <secret store> only, never in the repo | the running service |
| Sensitive personal | email, payment refs | db, encrypted at rest | the owning user, support role (logged) |

## Threat model (M: keep here · L: move to THREAT_MODEL.md)
| Threat | Example | Control | Enforced by |
|---|---|---|---|
| Cross-tenant data leak | user A reads workspace B | row-level security, tenant filter | acceptance test `tenant-isolation` |
| Injection | SQL built from input | parameterized queries only | lint rule + Semgrep in verify.sh |
| Prompt injection (if the product uses an LLM) | user text steers the model to call tools | no tools with write access on untrusted input (Rule of Two, README L8) | design review + test |
| Supply chain | malicious package | Posting 02 install gate, lockfile, minimum release age | hook + CI audit |

## Rules for agents and humans
- Auth and authorization checks live in <one place>; never re-implement them.
- Schema migrations: the agent writes them; a human applies them to production.
- The agent never holds production credentials.

## Not vulnerabilities (save reviewers time)
- <known-safe patterns that look scary>
```

### 3.10 `RUNBOOK.md`

**Answers:** how to deploy, roll back, see what's wrong, and recover. The file that separates a production system
from a POC: it's what you (or an agent you supervise) read at 2 a.m.
**S:** the *Operate* section of `README.md`. **M/L:** this file. **Create before:** the first production deploy.

```markdown
# Runbook

## Deploy
<how a change reaches production; who/what approves it>
## Roll back
<exact command or button; how long it takes>
## Health and alerts
- Dashboard: <link> · Error tracker: <link> · Uptime check: <link>
- Alert → first thing to check: <...>
## Backups and restore
- What: <db, files> · How often: <...> · Kept for: <...>
- Last restore test: <date>, took <minutes>. Test again every <quarter>.
## Common incidents
| Symptom | Likely cause | Fix |
|---|---|---|
## Who to call
<on-call / vendor support links>
```

### 3.11 `.claude/rules/`: for one area only

**Answers:** conventions that only matter in one part of the repo (frontend style, migrations, a legacy module).
Claude Code loads a rule with a `paths:` header only when it touches a matching file, so it costs nothing elsewhere.
**Create when:** `AGENTS.md` passes ~150 lines, or a rule only applies to one folder. In a monorepo, a package's own
`AGENTS.md` (with a `CLAUDE.md` shim) works the same way.
(Other agents don't read `.claude/rules/`. If several tools work on the repo, prefer a folder-level `AGENTS.md`.)

`.claude/rules/migrations.md`:

```markdown
---
paths:
  - "db/migrations/**"
---
- One change per migration; every migration has a down step.
- Never edit a migration that has run in production; add a new one.
- Large tables: add columns nullable first, backfill in batches, then add constraints.
```

### 3.12 The verification skill (pstack)

**Answers:** how an agent drives the real app like a user and saves proof (screenshots, responses).
Fully covered in [Posting 04 §4.1](04-pstack-rigorous-product-engineering.md): run
`/pstack:create-verification-skill` once; it writes `.claude/skills/verify-<app>/`. Then wire the top journeys into
`verify.sh`. **M and L tiers.** At S, Playwright tests in `tests/acceptance/` are enough.

### 3.13 `docs/progress.md`: only during long loops

**Answers:** where the last session stopped, for loops that restart with a fresh context (ralph, `claude -p`; README L4).
The plan file is the task list; `progress.md` holds only what the plan can't: the current state, what was tried, and
what's blocked. Create it when a loop starts; archive or delete it when the plan is done.

```markdown
# Progress: <plan file>          Updated: <date time>
Current task: <plan task #>
State: <what works now; last verify.sh result>
Tried and failed: <approach → why it failed>   (so the next session doesn't repeat it)
Blocked on: <nothing | question for the human>
```

---

## 4. Tools, not files: what to use at each size

Files hold the knowledge. Tools are the methods that use it. Each tool is covered in its own posting; this is only
the "which size" view.

| Tool | S | M | L | Posting |
|---|:-:|:-:|:-:|---|
| Secure workstation (sandbox, secrets, install gate) | ● | ● | ● | [02](02-secure-agent-workstation.md) |
| gstack office hours → becomes `PRD.md` | ◐ if it's a business | ◐ | ◐ new product lines | [03](03-gstack-business-development.md) |
| Superpowers (spec → plan → TDD → review → verify) | ◐ risky features | ● | ● | [01](01-superpowers-daily-workflow.md) |
| Harness: `verify.sh`, locked target, Stop gate, 3 strikes | ● | ● | ● + required CI | [01 §3.3](01-superpowers-daily-workflow.md) |
| `/goal` loop | ● | ● | ● | README L4 |
| Fresh-context loops (ralph, `claude -p`) + `progress.md` | — | ◐ long plans | ● | README L4 |
| pstack verification skill, `/interrogate`, `/blast-radius` | — | ● risky PRs | ● | [04](04-pstack-rigorous-product-engineering.md) |
| pstack hillclimb (improve a number) | — | ◐ | ◐ | [04 §4.3](04-pstack-rigorous-product-engineering.md) |
| pstack `/correct` (repeated mistake → check) | ◐ | ● | ● | [04 §4.4](04-pstack-rigorous-product-engineering.md) |
| Scheduled doc-freshness run (§4.3) | — | ◐ | ● | this posting |

### 4.1 Small: a 3-page web app, step by step

1. Posting 02 once per machine. 2. Write `docs/PRD.md` (one page) yourself, or have Claude interview you and draft
it, then **you** edit it. 3. Ask Claude for `README.md`, `AGENTS.md` + `CLAUDE.md` shim, and the Posting 01 harness
from this posting's templates. 4. Each PRD story's acceptance line becomes a test in `tests/acceptance/`.
5. Build: Track A only for login/payments/data deletion, Track D for the rest, `/goal ./verify.sh exits 0` to finish.
6. Before launch, fill the README *Operate* section and do one backup restore.

### 4.2 Medium and large: what changes

- **M:** after the PRD, have Claude draft `ARCHITECTURE.md`, `SECURITY.md` and `RUNBOOK.md`; you correct them. Every
  real feature goes through Superpowers. Build the verification skill. Run `/pstack:interrogate` before merging risky
  PRs. Write an ADR or glossary entry the moment a trigger fires, not later.
- **L:** also `THREAT_MODEL.md`, per-context glossaries with a map, per-package `AGENTS.md` or path rules,
  `verify.sh` as a required CI check, `CODEOWNERS` on agent config, CI and deploy files, and fresh-context loops with
  `progress.md` for long plans.

### 4.3 Keep the docs true (M optional, L must)

Docs rot, and a wrong doc is worse than none because the agent obeys it. Three layers, cheapest first:
1. **`check-docs` in `verify.sh`** (§3.4): catches broken paths and an oversized `AGENTS.md` on every run.
2. **Update docs in the same PR as the code.** `AGENTS.md`: *"If your change makes a line in ARCHITECTURE.md,
   GLOSSARY.md or an ADR false, update it in the same PR."* Reviewers check docs alongside tests.
3. **A weekly doc-freshness run** (L): a scheduled Claude Code task (`/loop` or a scheduled trigger) with the prompt
   *"Compare ARCHITECTURE.md, GLOSSARY.md, SECURITY.md and accepted ADRs with the code. Open one PR that fixes stale
   lines. Don't change code."* OpenAI's harness team runs this kind of "doc-gardening" agent. You review the PR.

---

## 5. Instructions for Claude (when this file is given to you)

1. **Pick the tier.** Ask at most three questions (people and agents working on it; money, sensitive data or
   regulation; number of deployables), apply §2.1 including the move-up triggers, and state the tier in one line.
   The user can override.
2. **Inventory before writing.** List which §3.0 files exist, which are missing for this tier, and any file from the
   "you don't need" table that already exists. For each duplicated fact you find, propose which single file keeps it
   and replace the other copy with a link. Show this list and wait for approval before creating files.
3. **Draft only what's missing, from §3's templates.** Fill in only what you can confirm from the code or the user.
   Leave `<placeholders>` and `Open questions` rather than guessing. Never add directory trees, dependency lists or
   generic overviews.
4. **`AGENTS.md` holds the content; `CLAUDE.md` is the shim** (§3.2). If the repo already has a long `CLAUDE.md`,
   propose moving its shared content into `AGENTS.md`; don't copy it.
5. **The human writes intent.** For `PRD.md` and `GLOSSARY.md`, interview the user and draft, then ask them to edit.
   Mark these `Status: draft` until they approve.
6. **Pair rules with checks.** For every invariant or hard rule you write, name what enforces it, or propose the test,
   lint rule or hook that would. Add `check-docs` to `verify.sh`. Propose the `AGENTS.md`/`CLAUDE.md` patterns for
   Posting 02's protect-config hook.
7. **Create later files on their trigger.** When work hits a §3.0 "create when" condition (a second name for one
   concept, a hard-to-reverse decision, the first deploy), say so in one line and offer to create that file then.
8. **Keep it small.** `AGENTS.md` under ~100 lines (cap 200). If a section only applies to one folder, move it to a
   path rule or a folder-level `AGENTS.md`.

---

## 6. Quick decision guide

```
Throwaway prototype or one-day build?                 → README.md only. Posting 01 Track D
Small app (one person, few screens, low stakes)?      → README (+Operate), AGENTS.md + CLAUDE.md shim, 1-page PRD, verify.sh + hooks
...but it takes payments or stores sensitive data?    → It's Medium
Medium?                                               → Small set + ARCHITECTURE, SECURITY, RUNBOOK, specs per feature, verification skill
Large / regulated / many teams?                       → Everything in §3.0 + THREAT_MODEL, glossary map, required CI, CODEOWNERS, doc-freshness run
Want both CLAUDE.md and AGENTS.md?                    → Content in AGENTS.md; CLAUDE.md = "@AGENTS.md" + Claude-only lines
Want an SDD.md / Ontology.md / DESIGN.md?             → No: feature specs (Superpowers), GLOSSARY.md, ARCHITECTURE.md + ADRs
Two names for one concept showed up?                  → Create or update GLOSSARY.md before the code
Hard-to-reverse, surprising, real trade-off?          → Write an ADR (all three, or don't)
AGENTS.md over ~150 lines?                            → Move folder-specific parts to .claude/rules/ or a folder AGENTS.md
Doc and code disagree?                                → Fix the doc in the same PR; L tier: weekly doc-freshness run
```

---

## Sources (2025–2026)

Full list with dates, and what each source contributed: [research notes, Sources](../research/05-must-have-project-files-research.md#sources). The main ones:

- Gloaguen et al. (ETH Zurich, LogicStar.ai), "Evaluating AGENTS.md" (arXiv 2602.11988, ICLR 2026): https://arxiv.org/html/2602.11988
- OpenAI, "Harness engineering: leveraging Codex in an agent-first world" (Feb 11 2026): https://openai.com/blog/harness-engineering
- Anthropic, "Effective harnesses for long-running agents" (Nov 2025, updated Oct 2026): https://anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Claude Code docs, Memory and AGENTS.md; Best practices (Nov 2026): https://code.claude.com/docs/en/memory · https://code.claude.com/docs/en/best-practices
- Birgitta Böckeler, "Context Engineering for Coding Agents" (Feb 2026): https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html
- Uvik, "Spec-Driven Development Benchmark 2026" (vendor): https://uvik.net/blog/spec-driven-development-benchmark/
- Matt Pocock, "/domain-modeling" skill (Aug 2026): https://aihero.dev/skills-domain-modeling
- Duncan Davidson, "Coding agents love decision records" (O'Reilly Radar, Oct 2026): https://oreilly.com/radar/coding-agents-love-decision-records/
- Blake Crosley, "PRD-driven development" (Feb 2026): https://blakecrosley.com/blog/prd-driven-development
- OWASP, "Secure Coding with AI Cheat Sheet": https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_with_AI_Cheat_Sheet.html
- VirtusLab, "Hardening the loop" (Nov 2026): https://virtuslab.com/blog/ai/hardening-the-loop-security-good-practises-for-agentic-development/
