# Business Development with gstack: Decide What's Worth Building Before Claude Builds It

> **How to use this file:** drop it into a Claude Code session and say:
> *"Use this md file for my project, building <your product idea or business>."*
> Claude should then run the business side first (demand, wedge, scope, kill line) and only then hand over to the build workflow.

Last researched: **November 2026** (gstack 1.104, Claude Code 2.1.3xx, Claude Opus 5.5). Works with
[Posting 01](01-superpowers-daily-workflow.md) (how to build) and [Posting 02](02-secure-agent-workstation.md) (how to stay safe).

---

## 1. The idea in one paragraph

With an AI coding agent, building got cheap. Garry Tan (President and CEO of Y Combinator) measures his own 2026
output at hundreds of times his 2013 pace. When building is cheap, the expensive mistake is no longer *building it
wrong*. It's **building the wrong thing**, fast and well-tested. [gstack](https://github.com/garrytan/gstack) is
Tan's free, MIT-licensed pack of Claude Code skills. Most of it is a software sprint (plan, review, QA, ship), which
[Posting 01](01-superpowers-daily-workflow.md) already covers with Superpowers. The part that's new for this repo is
the **front end of the sprint**: `/office-hours` (a YC-style office-hours interview with six forcing questions about
demand), `/plan-ceo-review` (a founder's scope review), and a **dated kill line** that gstack checks for you next time.
This posting uses gstack for those business decisions and hands the approved result to Superpowers for the build.

| Stage | Question | Tool | Output |
|---|---|---|---|
| **0. Decide** (new) | *Is this worth building, for whom, and what's the smallest version someone pays for?* | gstack `/office-hours`, `/plan-ceo-review` | `docs/designs/<topic>.md` or `docs/memos/<topic>.md`, with a **KILL/CONTINUE** line |
| 1–3. Build | *Build it correctly* | Superpowers + your harness (Posting 01) | Spec → plan → tested code → PR |
| 4. Learn | *Did real people use it or pay?* | You, plus gstack's kill check and `/retro` | Continue, narrow, pivot or kill |

### 1.1 The idea that ties it together: the same feedback loop, one level up

[README L1](../README.md#l1-the-feedback-loop-the-core-idea) says a coding agent needs a loop with a **target**, a
**sensor**, a **compare** step outside the agent, an **action** and a **safety stop**. A business idea needs the same
loop. The only difference is the sensor: for code it's `./verify.sh`; for a business it's **what real people do**.

| Loop part | Code loop (Posting 01) | Business loop (this posting) |
|---|---|---|
| Target | Acceptance tests pass | A behavior or money test: *"3 of the 10 dentists I demo to pay $49 by Dec 15"* |
| Sensor | `./verify.sh` exit code | Payments, repeat usage, someone upset when it breaks. **Not** praise, waitlists or "that's interesting" |
| Compare (outside the agent) | Stop hook runs `verify.sh` | gstack's kill check reopens the question on the date: *"Dec 15 passed: did anyone pay?"* |
| Action | Claude edits code | The cheapest experiment: a conversation quota, a payment link, delivering by hand, a tiny wedge |
| Safety | Same failure 3× → `NEEDS-HUMAN.md` | The dated **KILL/CONTINUE** line, written *before* you act, so you can't move the goalposts later |

The rule "the doer doesn't judge" applies twice here. A founder grades their own idea too kindly. A language model
also grades your idea too kindly: a Stanford study found models **affirm users about 50% more than people do**
([arXiv 2510.01395, Oct 2025](https://arxiv.org/abs/2510.01395)). gstack's office hours fights this with
written anti-flattery rules, and the kill line fights it with a date and a number.

---

## 2. Is this worth it on the latest Claude Code? (verdict)

**Yes for the business stage (`/office-hours`, `/plan-ceo-review`, the kill line). Not as a second build method on
top of Superpowers.**

What the evidence says:

- **gstack doesn't make Claude write better code.** gstack's own November 2026 test on 100 everyday GitHub bugs and
  features: Claude Opus 5.5 fixed 58 with gstack and 61 of 99 without it. On 60 security fixes, both fixed 29. The
  README says it plainly: *use gstack "for the workflow, the reviews, the shipping steps and the safety guards, not for
  a higher fix rate"* ([results](https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-19-geva-540-skill-pack-h2h-results/README.md),
  [correction](https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-20-geva-565-skill-pack-h2h-correction/README.md)).
  That's honest, and it matches research: persona prompts ("you are a CEO") don't make models more accurate
  ([Zheng et al., EMNLP Findings 2024](https://arxiv.org/abs/2311.10054)); they mostly change depth and tone
  ([Xiao et al., May 2026](https://www.alphaxiv.org/abs/2605.29420)). The value is in the **questions and
  gates**, not the role names.
- **The office-hours method is real customer-development practice, encoded.** Its rules are what experienced
  investors and books like *The Mom Test* teach: interest isn't demand; rank evidence *prepay > signed contract >
  paid pilot > priced letter of intent > unpriced LOI > praise*; the status quo is your real competitor; watch
  people use it instead of demoing; narrow beats wide. A skill turns that from advice you read once into a gate you
  pass every time.
- **Focused beats huge.** gstack installs 60+ skills. A 2026 benchmark found curated skills raise agent pass rates from
  33.9% to 50.5% on average, and that *"focused skills with at most three modules outperform larger or exhaustive
  bundles"* ([SkillsBench, arXiv 2602.12670](https://arxiv.org/abs/2602.12670)). So this posting turns off the gstack
  skills you don't need (§3.3).
- **Type the skill's name.** In the same gstack test, installed skill packs that weren't started by name often didn't
  run their procedure at all. Type `/office-hours`; don't hope it triggers.
- **The criticism is mostly about the hype, not the questions.** Reviewers call the "team of twenty" slogan marketing
  and the lines-of-code numbers a red flag, and note the US-startup tone and the overlap with other review and release
  skills. The same reviewers recommend keeping only the commands you need
  ([one-month review, May 2026](https://claude-codex.fr/en/content/garry-tan-stack-claude-code/),
  [andrew.ooo, Mar 2026](https://andrew.ooo/posts/gstack-garry-tan-claude-code-setup/)). That's what §3.3 does.
- **It conflicts with Superpowers if you run both for building.** gstack has its own plan reviews, `/build-check`,
  `/review` and `/gstack-ship`. Two methods with different gates confuse the agent and you. Pick one for the build.
  This repo picks Superpowers (Posting 01) and uses gstack *upstream* of it.

### 2.1 "Can't I just ask Claude 'is this a good idea?'"

You'll get a friendly, balanced answer that flatters you. Office hours is different in three ways:
1. **It asks before it answers**, one question at a time, and pushes twice on vague answers ("Name the actual human.
   Not 'SMBs'.").
2. **It's forbidden to hedge.** Phrases like "that's an interesting approach" and "you might want to consider" are
   banned during the diagnostic. It must take a position and say what evidence would change it.
3. **It writes the decision down with a date** and checks it next session. A chat answer disappears after `/clear`.

### 2.2 What do I actually need to learn?

Only three things:
1. **The six forcing questions** (§4.1), so you recognize a weak answer when you give one.
2. **The evidence ladder:** money and behavior count; compliments and signups don't.
3. **The KILL/CONTINUE line:** before you build, write *"By <date>, if <behavior or money test> isn't true, I change
   direction."*

You don't need to learn the other ~55 gstack skills.

---

## 3. One-time setup (≈15 minutes per machine)

### 3.1 Install gstack, then read what it changed

gstack is code from the internet that registers hooks in your user settings. Treat it like any plugin
([Posting 02 §3.6](02-secure-agent-workstation.md)): read before trusting. Requirements: Git, [Bun](https://bun.sh)
1.4+, and Node.js.

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup          # solo install; do NOT use --team unless a team needs it
```

`./setup` registers these hooks in `~/.claude/settings.json` ([docs/install.md](https://github.com/garrytan/gstack/blob/main/docs/install.md)).
Open the file and check they're what you expect:

| Hook | What it does | Keep? |
|---|---|---|
| `PreToolUse` destructive-command check (`/careful`) | Warns before `rm -rf`, `DROP TABLE`, force-push; hard-denies home-directory deletes and force-push to the default branch | **Yes.** It adds to Posting 02's denies |
| `UserPromptSubmit` skill routing | Suggests the matching gstack skill for your prompt | Optional. Turn off if it fights Superpowers' triggers: `./setup --no-skill-routing` |
| `Stop` skill contracts | Blocks once if a gstack skill ends without its required output (e.g. no design doc) | Yes |
| `Stop` timeline | Closes session-timeline entries; always exits 0 | Harmless |
| `SessionStart` auto-update | **Only in team mode.** Pulls new gstack versions on session start | **Skip for solo use.** Update by hand with `/gstack-upgrade` after reading the changelog |

### 3.2 Privacy and outside reviews (2 minutes)

gstack sends nothing about your usage unless you opt in: **telemetry is off by default**. Two things are on by default:

```bash
G=~/.claude/skills/gstack/bin/gstack-config
$G set codex_reviews disabled      # outside reviews send your code/plans to OpenAI Codex under your login. Turn on per project if you want them
$G set founder_resources false     # optional: skips the closing YC resources / application note in office hours
# $G set update_check false        # optional: stops the version check (2–3 GitHub requests)
~/.claude/skills/gstack/bin/gstack-egress list   # every off-machine send leaves a receipt here
```

Office hours stores named customers and prices **only** in your private `~/.gstack` folder. Copies it writes into
the repo use roles or initials and run through a secret scan first. Keep it that way: don't paste customer names
into `CLAUDE.md` or specs.

The install prompt in gstack's README also asks Claude to add a `CLAUDE.md` rule *"never use mcp__claude-in-chrome
tools"*. That's only needed if you use gstack's browser skills (`/qa`, `/browse`). For this posting you can skip it.

### 3.3 Keep only the business skills loaded

Every installed skill's description loads into every session (README L5: context is a budget). Turn off the gstack
skills that overlap with Superpowers or that you won't use:

```bash
~/.claude/skills/gstack/bin/gstack-config set disabled_skills \
  build-check,gstack-ship,land-and-deploy,modernize,spec,autoplan,deslop-shared-libs,test-audit,write-skill,pair-agent,ubicloud,runaway-stopper,eval-plan,captain,landing-report,scrape,setup-browser-cookies,make-pdf,diagram,design-motion,ios-qa,ios-fix,ios-design-review,ios-clean,ios-sync
```

What stays on, and why:

| Keep | Use it for |
|---|---|
| `/office-hours` | **The core of this posting.** Demand diagnosis, pivot decisions, design doc or decision memo with a kill line |
| `/plan-ceo-review` | Scope decision on the design doc: expand, hold, or cut to the wedge |
| `/plan-design-review`, `/design-consultation`, `/design-shotgun` | If the product's first impression matters (consumer apps, landing pages) |
| `/plan-devex-review`, `/devex-review` | If your customers are developers (API, CLI, SDK) |
| `/review`, `/qa`, `/qa-only`, `/browse` | Optional second opinions after the Superpowers build (`/qa` opens a real browser) |
| `/retro`, `/document-release`, `/canary`, `/benchmark`, `/careful`, `/freeze`, `/guard`, `/learn`, `/codex` | Optional tools; harmless when unused |

`gstack-config set disabled_skills ""` turns everything back on. Run `~/.claude/skills/gstack/bin/gstack-doctor` to
check the install.

### 3.4 Add the business stage to your project harness

Add this to the project's `CLAUDE.md` (next to the Posting 01 rules):

```markdown
## Business stage (Posting 03)
- New product, new customer segment, or a feature that costs more than ~2 days: run /office-hours first.
  Superpowers brainstorming starts FROM the approved docs/designs/<topic>.md, not from scratch.
- Every design doc and memo has a KILL/CONTINUE line: "By <date>, if <behavior or money test> isn't true, change direction."
- Evidence ranking: prepay > signed contract > paid pilot > priced LOI > unpriced LOI > praise. Waitlists and compliments are not demand.
- Customer names and prices stay out of the repo; use roles or initials.
- On or after a kill date, ask me the kill question before starting new feature work for that product.
```

---

## 4. The workflow

### 4.1 Stage 0a: office hours (≈30–60 min, you answer, Claude pushes)

Type `/office-hours` and describe the idea. It first asks your goal. Pick **Building a startup**,
**Intrapreneurship** (an internal project at a company) or **Pivot** to get the hard **Startup mode**. The other
goals (hackathon, learning, fun) get a gentler brainstorming partner.

It opens with a reality check: *"Has it produced one real outcome yet: one paying user, or one person who used it
this week without being asked? Name them."* Then it asks the forcing questions that fit your stage, one at a time:

| # | Question | A strong answer | A weak answer (it will push back) |
|---|---|---|---|
| Q1 | **Demand reality:** who would be genuinely upset if it disappeared tomorrow? | "Two clinics call me when it's down for 20 minutes" | "500 waitlist signups", "people say it's interesting" |
| Q2 | **Status quo:** what do they do today, even badly, and what does it cost? | "A front-desk person spends 6 h/week in a spreadsheet" | "Nothing exists, that's why it's a big opportunity" |
| Q3 | **Desperate specificity:** name the human. Title? What gets them promoted or fired? | "Dr. K, owner of a 3-chair practice; no-shows cost her ~$2k/month" | "Healthcare SMBs" |
| Q4 | **Narrowest wedge:** smallest version someone pays for *this week*? | "An SMS reminder that cuts no-shows, $49/month, no integration" | "We need the full platform first" |
| Q5 | **Observation:** watched someone use it without helping? What surprised you? | "She ignored the dashboard and only read the SMS replies" | "We sent a survey" |
| Q6 | **Future-fit:** in 3 years, is it more essential or less? | A specific claim about how the users' world changes | "The market grows 20% a year" |

Routing by stage: **pre-product** gets Q1–Q3, **has users** gets Q2, Q4, Q5, **has paying customers** gets Q4–Q6.
For an internal project, Q4 becomes *"the smallest demo that gets your sponsor to greenlight it"* and Q6 *"does it
survive a reorg when your champion leaves?"*

Then it challenges your premises (you agree or disagree with each), offers 2–3 approaches (always one "minimal
viable" and one "ideal"), **stops for your choice**, and writes either:

- a **design doc** at `docs/designs/<topic>.md` (when the next step is building a wedge), or
- a **one-page decision memo** at `docs/memos/<topic>.md` (when nobody has used or paid yet, or you're deciding a
  pivot). A memo's "one action" is usually *not* code: a conversation quota ("12 practice owners in 14 days"), a
  real payment link, or delivering the service by hand to the next few customers.

Both end with the **KILL/CONTINUE** line. When you approve, gstack records it, and the next office-hours session on
this repo after the date opens with *"<date> passed: did anyone pay?"* That's the compare step running outside you.

**If you're impatient:** say "skip the questions". It asks the two most important remaining ones, then moves on. If
you push a second time it respects that. The reality check is never skipped.

### 4.2 Stage 0b: scope review (≈20 min)

Run `/plan-ceo-review`. It reads the design doc and asks you to pick a mode:

| Mode | Pick it when |
|---|---|
| **Scope reduction** | No one has paid yet. Strip to the wedge. *This is the right default before demand is proven* |
| **Hold scope** | The scope is right; review it rigorously (failure modes, what's deferred) |
| **Selective expansion** | Real users want something adjacent; cherry-pick 1–2 additions |
| **Scope expansion** | Demand is proven and you want the "10-star" version |

Accepted additions go into the plan; deferred ideas go to `TODOS.md`, so they stop nagging you.

Optional extra reviews at this stage: `/plan-design-review` for consumer products (it scores each design dimension
0–10 and flags "AI slop"), `/plan-devex-review` for developer products (time to hello world, error messages).

### 4.3 Hand-off to the build (Posting 01, Track A)

Now start the normal Superpowers flow, pointing it at the decision you made:

> *"Build the wedge from docs/designs/<topic>.md. Use it as the starting point for brainstorming; don't re-open
> the business questions. Out of scope: everything under 'deferred'."*

Superpowers' spec (Gate 1) and plan (Gate 2) then cover *how* to build it. The acceptance tests in Posting 01 are
the code target; the KILL/CONTINUE line is the business target. Keep both.

### 4.4 Stage 4: measure and decide

After the wedge is in someone's hands:
- **On the kill date**, run `/office-hours` again (or answer Claude's question from your `CLAUDE.md` rule). Report
  behavior, not feelings: who paid, who came back, who complained when it broke.
- **Met the line?** Write a SUCCESS line with a new date and the next target, then `/plan-ceo-review` in *selective
  expansion*.
- **Missed it?** Choose **Pivot** as the goal in office hours. It asks: is the problem *no love* (a fit problem: run
  discovery on a new direction) or *market too small* (do the ceiling math: reachable buyers × a defensible annual
  price)? Then *"starting from zero today, what would you work on?"* and a new kill line before you act.
- **Weekly:** `/retro` gives a shipping and test-health summary across your repos. Pair it with one line of your own:
  *what did customers do this week?*

---

## 5. Instructions for Claude (when this file is given to you)

1. **Check setup.** Is gstack installed (`~/.claude/skills/gstack`, `/office-hours` available)? If not, show §3.1–3.3
   and wait. Report which gstack hooks are registered in `~/.claude/settings.json` and whether `codex_reviews` is on.
   Don't install team mode or change user-level settings without explicit approval.
2. **Classify the request.** New product, new segment, pivot, or a feature that costs more than ~2 days → Stage 0
   (office hours). Clear, small, already-validated work → skip to Posting 01. State the choice in one line; the user
   can override.
3. **Stage 0:** run `/office-hours`, then `/plan-ceo-review`. Don't write product code during Stage 0. Don't soften
   the diagnostic. If the user has no real outcome yet, recommend the memo route and a non-code action.
4. **Every design doc or memo must end with a dated KILL/CONTINUE (or SUCCESS) line** with a behavior or money test.
   Reject lines like "if users like it".
5. **Hand-off:** start Superpowers brainstorming from the approved `docs/designs/<topic>.md` and follow Posting 01 for
   the build. Don't use gstack's `/autoplan`, `/build-check` or `/gstack-ship` on the same task as Superpowers.
6. **Privacy:** never put customer names, prices or deal terms into repo files, commit messages or PRs; use roles or
   initials. Don't send code to outside reviewers unless the user turned `codex_reviews` on.
7. **On or after a kill date**, ask the kill question before new feature work on that product.

---

## 6. Quick decision guide

```
New idea, or "is this worth building?"          → /office-hours (Startup mode)
Nobody has used or paid yet?                     → Memo route: conversation quota, payment link, or deliver by hand
Someone pays; what do I build first?             → Design doc → /plan-ceo-review (scope reduction) → Posting 01
Internal project at work?                        → /office-hours, goal "Intrapreneurship" (sponsor = customer)
Thinking about a pivot?                          → /office-hours, goal "Pivot" (love vs. size, new kill line)
Developer product?                               → add /plan-devex-review before building
Kill date passed?                                → Report behavior and money, not feelings; continue, narrow, pivot or kill
Want gstack to also plan/build/ship?             → Not with Superpowers on the same task. Pick one method
```

---

## Sources (2025–2026)

- gstack repository, README, ETHOS and docs (v1.104, Nov 2026): https://github.com/garrytan/gstack · install and hooks: https://github.com/garrytan/gstack/blob/main/docs/install.md · privacy: https://github.com/garrytan/gstack/blob/main/docs/privacy.md
- gstack `/office-hours` skill (startup diagnostic, pivot flow, design doc and KILL/CONTINUE template): https://github.com/garrytan/gstack/tree/main/office-hours
- gstack evals, skill pack head-to-head results and correction (Nov 19–20, 2026): https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-19-geva-540-skill-pack-h2h-results/README.md · https://github.com/garrytan/gstack-evals/blob/main/wiki/board/2026-11-20-geva-565-skill-pack-h2h-correction/README.md
- Garry Tan, "Why gstack" (productivity numbers and their caveats): https://github.com/garrytan/gstack/blob/main/docs/why-gstack.md
- "Garry Tan's gstack: a critical review after one month of use" (May 2026): https://claude-codex.fr/en/content/garry-tan-stack-claude-code/
- andrew.ooo, "gstack: Y Combinator CEO's Claude Code setup" (Mar 2026): https://andrew.ooo/posts/gstack-garry-tan-claude-code-setup/
- "Why Garry Tan's Claude Code setup is dividing opinions" (Mar 2026): https://wired24.co.za/2026/03/17/why-garry-tans-claude-code-setup-is-dividing-opinions/
- "Garry Tan's gstack split the developer community" (2026): https://newclawtimes.com/articles/garry-tan-gstack-claude-code-ai-coding-developer-debate/
- Cheng et al., "Sycophantic AI decreases prosocial intentions and promotes dependence" (Stanford, arXiv 2510.01395, Oct 2025): https://arxiv.org/abs/2510.01395
- "Founders are asking a helpful machine to judge their ideas. That's the problem" (Inc.): https://www.inc.com/diana-bocco/founders-ai-prompting-business-ideas-confirmation-bias/91394909
- Zheng et al., "When 'A Helpful Assistant' Is Not Really Helpful: Personas in System Prompts Do Not Improve Performances of LLMs" (EMNLP Findings 2024): https://arxiv.org/abs/2311.10054
- Xiao et al., "When Does Persona Prompting Actually Help?" (May 2026): https://www.alphaxiv.org/abs/2605.29420
- SkillsBench: "Benchmarking How Well Agent Skills Work Across Diverse Tasks" (arXiv 2602.12670, Feb–Jun 2026): https://arxiv.org/abs/2602.12670
