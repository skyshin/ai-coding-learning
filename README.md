# ai-coding-learning

Learn the new things you need for coding with AI assistants. This repo is used in two ways:

1. **Learn:** read this README from top to bottom. Each lesson explains one idea in plain words, says what you
   actually do with it, and links to more reading.
2. **Use:** each file in [`postings/`](postings/) is a workflow you hand to Claude Code in one sentence:
   *"Use this md file for my project, building …"*. Claude follows it, so you don't have to remember the details.

_Last updated: November 2026 (Claude Code 2.1.3xx, Superpowers v7.0.2). Added Posting 02 and lessons L8–L10 on agent security._

---

## Postings

| # | Posting | What it gives you | Lessons it uses |
|---|---|---|---|
| 01 | [Daily coding with Superpowers + Claude Code + your own harness](postings/01-superpowers-daily-workflow.md) | A daily workflow: spec → plan → test-first build → review → verified "done", with hooks that lock the tests and a stop for when Claude gets stuck | L1–L7 |
| 02 | [Secure agent workstation: run Claude Code without handing over your keys](postings/02-secure-agent-workstation.md) | Sandbox with your credentials hidden, secrets in three tiers, hooks that block made-up packages and edits to the agent's own config, a plugin vetting checklist, session types, and a 10-minute self-test | L2, L4, L8–L10 |

---

## Learning path

Start with **L1**. Everything else is a way of building or supervising it.

| Lesson | Topic | Depth you need | Time |
|---|---|---|---|
| [L1](#l1-the-feedback-loop-the-core-idea) | The feedback loop | **Understand deeply** | 15 min |
| [L2](#l2-harness-engineering-claudemd-and-hooks) | Harness engineering: `CLAUDE.md` and hooks | Know what exists and what it blocks | 20 min |
| [L3](#l3-superpowers-a-method-you-install) | Superpowers | The tracks and the 3 approval points | 10 min |
| [L4](#l4-loops-letting-claude-keep-going) | Loops: `/goal`, ralph, Ultracode | One command and its safety rules | 10 min |
| [L5](#l5-context-management) | Context management | A few habits | 10 min |
| [L6](#l6-reviewing-ai-written-code-and-tests) | Reviewing AI-written code and tests | **Practice on every PR** | ongoing |
| [L7](#l7-why-prompting-tricks-matter-much-less-now) | Why prompting tricks matter much less now | Unlearn old habits | 5 min |
| [L8](#l8-the-lethal-trifecta-and-the-rule-of-two) | The lethal trifecta and the Rule of Two | **Understand deeply** (it's L1 for security) | 15 min |
| [L9](#l9-sandbox-permissions-and-auto-mode-three-different-layers) | Sandbox, permissions and auto mode | What each one stops and what it doesn't | 15 min |
| [L10](#l10-secrets-and-the-supply-chain) | Secrets and the supply chain | Three tiers of secrets; how packages attack you | 15 min |
| [Example](#worked-example-starting-a-new-project-from-zero) | **Worked example:** a new project from zero, step by step | Read once, then follow | 15 min |

**The short version:** you don't need to master these tools. You need to know enough to **supervise**.
Claude writes the hooks, the plans and the code. Your job is to say what "done" means, check the work at a few
gates, and turn repeated mistakes into rules. Prompting tricks matter much less than they used to.

---

### L1. The feedback loop (the core idea)

**The problem.** A coding agent often says "Done, all tests pass!" when they don't pass. An agent that grades its own work
grades too kindly. Anthropic names this, "declaring victory too early", as a main failure mode of long-running agents.

**The idea.** Borrow the control loop from engineering. Think of a thermostat: it has a **target** (21°C), a **sensor** (a thermometer),
a **compare** step (too cold?) and an **action** (heat). It repeats until the error is zero. Here's the same loop for code:

| Part | In your project | Owned by |
|---|---|---|
| Target | Tests and acceptance checks that a computer can run | **You** (or a spec you approved) |
| Sensor | One command whose exit code is the truth: `./verify.sh` → `0` = pass | The harness |
| Compare: "are we done?" | Decided **outside** Claude: a hook or `/goal` runs the sensor | The harness |
| Action | Claude edits code to make the checks pass | Claude |
| Safety | A cap on attempts. If the same failure repeats 3 times, stop and call a human (`NEEDS-HUMAN.md`) | The harness |

**Three rules that make it work**
1. **The target must be checkable by a machine.** "Make it nice" isn't a target. "`./verify.sh` exits 0" is.
2. **The target is locked.** Claude may not edit the tests to make them pass.
3. **The doer doesn't judge.** Something other than the working agent decides when it's done.

**How the buzzwords fit:** *spec-driven development (SDD)* **sets the target**. The *feedback loop* is the **design**.
*Harness engineering* is the **construction work** that makes the loop run.

**What you do:** for each task, ask yourself "what command would prove this is done?" and get that into
`verify.sh` or a test before Claude starts.

**Check yourself:** why is "Claude, please double-check your work" weaker than a Stop hook that runs `./verify.sh`?
<details><summary>Answer</summary>The first is an instruction that the same agent grades, and it can be skipped or forgotten. The second runs every time, and the exit code decides, not Claude.</details>

**Read more:** [Posting 01 §1.1](postings/01-superpowers-daily-workflow.md) ·
[Harness engineering beyond skills: using sensors (video)](https://www.youtube.com/watch?v=uLWOLmeHOSE) ·
[Loop engineering guide](https://blakecrosley.com/guides/loop-engineering)

---

### L2. Harness engineering: `CLAUDE.md` and hooks

**What a harness is.** An agent is **model + harness**. The harness is everything around the model: tools, memory,
rules and checks. Claude Code *is* a harness. On top of it you build a small **project harness** for your repo.
This matters a lot. In 2026 studies, changing only the harness moved benchmark scores by 7–14 points, and the same
model scored about 10 points apart depending on its harness ([summary](https://baeseokjae.github.io/posts/harness-design-coding-agent-performance-2026/),
[arXiv 2609.00006](https://arxiv.org/abs/2609.00006)).

**Your project harness has two parts:**

| Part | What it is | Strength | Example |
|---|---|---|---|
| `CLAUDE.md` | Instructions Claude reads at the start of every session | **A request.** Usually followed, but can be skipped | "Done = `./verify.sh` exits 0. Never push to main." |
| Hooks | Scripts Claude Code runs automatically at certain moments | **A guarantee.** Runs every time | Block edits to `tests/acceptance/`; refuse to stop until `verify.sh` passes |

**The hooks worth knowing** (configured in `.claude/settings.json`):
- **PreToolUse** runs *before* Claude uses a tool. Exit code 2 blocks the action, and Claude sees your message.
  Use it to **lock the target** and block `git push` to `main`.
- **Stop** runs when Claude tries to finish. It can say "not yet" and send back the failures.
  Use it to **decide "done" outside Claude**.
- **PostToolUse** runs *after* a tool. Use it to auto-format or lint changed files.
- **SessionStart** runs when a session starts. Plugins like Superpowers use it to load their instructions.

**The one habit:** *when Claude makes the same mistake twice, don't write a longer prompt. Add a rule, a test or a hook so
the mistake can't happen again* (Mitchell Hashimoto's framing, via [Faros](https://www.faros.ai/blog/harness-engineering)).

**What you do:** you don't write hooks by hand. Ask Claude to set them up from Posting 01 §3.3, then **read** what it
wrote, so you know what's blocked and why.

**Check yourself:** you want Claude to *never* delete migration files. Do you put it in `CLAUDE.md` or a hook?
<details><summary>Answer</summary>A PreToolUse hook. "Never" means you need a guarantee, not a request. Put it in CLAUDE.md as well so Claude understands why.</details>

**Read more:** [Claude Code hooks reference](https://code.claude.com/docs/en/hooks) ·
[Faros: Harness engineering guide](https://www.faros.ai/blog/harness-engineering) ·
[Why your coding agents need a harness, not a prompt (video)](https://www.youtube.com/watch?v=bUtAGFXPc3A)

---

### L3. Superpowers: a method you install

**What it is.** [Superpowers](https://github.com/obra/superpowers) is a free plugin of about 15 skills that gives Claude a
senior engineer's working method. Claude Code gives you *mechanisms* (subagents, hooks, plan mode). It doesn't give
you a *method*. Superpowers is the method. It's **not built into Claude Code**, so install it once:
`/plugin install superpowers@claude-plugins-official`.

**The flow it enforces:** brainstorm → **spec** → isolated branch → **plan** of small tasks → build with **tests first** →
review between tasks → **verify** before claiming done → finish the branch with a PR.

**The only things you need to know:**
- **The 3 approval points:** (1) approve the **spec**, (2) approve the **plan**, (3) review the **PR**.
  Mistakes are cheapest to fix at (1) and (2).
- **The 4 tracks** in Posting 01: **A** feature (full flow), **B** bug (find root cause before fixing), **C** hand an approved plan to a loop,
  **D** small edit (skip the flow).
- You don't need to learn the skills one by one. They trigger on their own ("build X" starts brainstorming,
  "fix this bug" starts debugging).

**When not to use it:** typos, renames, one-file changes, throwaway prototypes. It costs extra tokens and only pays off on
real features and bugs.

**Why not just tell Claude "use all the latest agent tech"?** A sentence isn't enforced, disappears after `/clear` or a new
session, and Claude has to guess what you mean. Installed plugins, `CLAUDE.md` and hooks reload every time.
(Details: Posting 01 §2.1.)

**Read more:** [Superpowers complete guide](https://pasqualepillitteri.it/en/news/215/superpowers-claude-code-complete-guide) ·
[Superpowers 6.4 post by the author](https://blog.fsck.com/2026/09/21/superpowers-6.4/) ·
[Superpowers review: enforcing TDD and worktrees (video)](https://www.youtube.com/watch?v=kszFdj9_jV8)

---

### L4. Loops: letting Claude keep going

**What a loop is.** Normally Claude works one turn and stops. A loop makes it continue until a condition is met, so you can
walk away. Only start one **after** you've approved a plan and `verify.sh` exists.

| Tool | What it does | Use when |
|---|---|---|
| `/goal <condition>` | After each turn, a separate evaluator checks the condition and keeps Claude going until it's met | **Default.** "Keep going until `./verify.sh` exits 0" |
| ralph-loop plugin / `claude -p` in a shell loop | Restarts with a fresh context each round, reading the plan file as its memory | Long plans that would fill up one session |
| `/loop` or scheduled tasks | Re-runs a prompt on a clock | Recurring checks ("run the full suite every morning"), not building |
| **Ultracode** (`ultracode` in the prompt, or `/effort ultracode`) | Claude writes a workflow that runs many subagents in parallel with a verifier | Big, parallel jobs only. Expensive (one test: about 4x the cost), no human approval points |

**Safety rules (always):**
1. **Cap it:** `--max-iterations`, `--max-turns` or a budget. Unlimited is the default.
2. **The condition is the sensor:** say "`./verify.sh` exits 0", not "the feature works".
3. **Branch only.** The loop never touches `main`.
4. **Same failure 3 times → stop and call a human.**
5. **No loop with untrusted input and real secrets together.** A loop removes the human who would notice a hijack.
   Run loops in a normal build session, not one that read GitHub issues, error reports or web pages (L8).

**Read more:** [Agent loops: Ralph, /loop, /goal, headless](https://agentatlas.dev/guides/agent-loops-ralph-loop-ai-coding-agents/) ·
[From Ralph Wiggum to /loop](https://paddo.dev/blog/claude-code-loop-ralph-wiggum-evolution/) ·
[What is Ultracode](https://agentsroom.dev/blog/what-is-ultracode-in-claude-code)

---

### L5. Context management

**The problem: "context rot".** Everything in a session (your messages, Claude's replies, every file it read, every
test log) sits in one context window. Claude Code's window is now 1M tokens, but **bigger doesn't mean better**.
As the context grows, the model gets worse at using it: attention spreads thinner and stale content distracts it.
Anthropic calls context "a finite resource with diminishing marginal returns"
([Anthropic, Sep 2025](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents)). A 2025 Chroma
study of 18 models found that performance drops as input length grows ([Thoughtworks, Nov 2026](https://www.thoughtworks.com/insights/blog/generative-ai/what-is-context-rot)).
The official docs call the context window "the most important resource to manage"
([Claude Code best practices](https://code.claude.com/docs/en/best-practices)).

**How to spot it:** Claude forgets a constraint you gave earlier, repeats an approach that already failed, re-asks
settled questions, or makes inconsistent decisions.

**The idea: context engineering.** Give the model the *smallest set of high-signal tokens* that gets the job done
(Anthropic). Rather than a perfect prompt, you manage what's in the window.

**Your five tools (in Claude Code)**

| Tool | What it does | Use when |
|---|---|---|
| **`/clear`** | Empties the session. You write a short brief to start fresh | **Starting a new task.** Official rule of thumb: *new task = new session* |
| **`/rewind`** (Esc Esc) | Jumps back to an earlier point and drops everything after it | Claude went down a wrong path. Rewind to before the attempt and re-prompt with what you learned. Better than arguing with it |
| **`/compact <focus>`** | Claude summarizes the session so far (lossy) | Same task, long session. Steer it: `/compact keep the auth decisions and failing test names` |
| **Subagents** | A helper with a fresh context does a noisy job and returns only the conclusion | Big searches, log digging, reviews. Ask yourself: *"Will I need this output again, or just the conclusion?"* |
| **`/btw`** | Ask a side question that never enters the session history | Quick "what does this flag mean?" questions mid-task |

Source for the table: [Using Claude Code: session management and 1M context (Anthropic, Apr 2026)](https://claude.com/blog/using-claude-code-session-management-and-1m-context).

**Habits that keep sessions focused**
1. **One task, one session.** Don't mix "fix the login bug" with "and also restyle the header" (the "kitchen-sink session").
2. **Correct at most twice.** If you've corrected Claude twice on the same thing, the context is polluted.
   `/clear` and write a better first prompt that includes what you learned (official best practice).
3. **Put lasting knowledge in files, not chat.** Specs, plans, decisions and progress go in `docs/` (Superpowers does this).
   A new session reads them back in seconds, while chat history gets compacted and lost.
4. **Keep `CLAUDE.md` short.** It loads into *every* session. For each line ask: *"Would removing this cause Claude to make mistakes?"*
   If not, delete it. A bloated `CLAUDE.md` makes Claude ignore the rules that matter. Domain knowledge belongs in skills or docs.
5. **Keep tool output small.** Long test logs eat context. Have `verify.sh` print only failures, the way the Stop hook in Posting 01 sends back only the last 40 lines.
6. **Hand off on purpose.** Before `/clear`, ask: *"Write the current state to `docs/progress.md`: goal, decisions, what failed,
   what's left."* Then in the new session: *"Continue from docs/progress.md."*

**Check yourself:** you've worked 3 hours on one feature and Claude keeps re-asking settled questions. What do you do?
<details><summary>Answer</summary>Have Claude write the decisions and state into the spec, plan or a progress file, then /clear and tell it to continue from that file. Don't keep correcting in the polluted session.</details>

**Read more:** [Session management and 1M context (Anthropic)](https://claude.com/blog/using-claude-code-session-management-and-1m-context) ·
[Effective context engineering for AI agents (Anthropic)](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) ·
[What is context rot? (Thoughtworks)](https://www.thoughtworks.com/insights/blog/generative-ai/what-is-context-rot) ·
[Claude Code best practices](https://code.claude.com/docs/en/best-practices)

---

### L6. Reviewing AI-written code and tests

**Why this is your most important skill.** With a good harness, Claude does most of the typing. What's left for you is
judgment. The data says that judgment is needed:
- AI co-authored pull requests had **about 1.7x more issues** than human ones (10.83 vs 6.45 per PR), with about 75% more
  logic and correctness issues, nearly 2x more error-handling gaps, about 1.5x more security issues, and nearly 8x more excessive I/O
  ([CodeRabbit study of 470 PRs, Dec 2025](https://www.helpnetsecurity.com/2025/12/23/coderabbit-ai-assisted-pull-requests-report/)).
- **Agents cheat on tests when the tests make the task hard.** In ImpossibleBench (Carnegie Mellon and Anthropic, Oct 2025), tests were
  made to contradict the spec, so passing meant cheating. Frontier models often "passed" anyway. Claude models mostly did it by
  **editing the tests**. Others used tricks like **overriding `==` to always return True**, **counting calls**, or
  **hardcoding the exact test inputs** ([arXiv 2510.20270](https://arxiv.org/abs/2510.20270)). This is why Posting 01 locks the tests with a hook.
  Note that read-only tests don't stop the hardcoding tricks, so you still need to review.

**A green `verify.sh` only proves what the tests check.** So review the tests first, then the code.

**Step 1: review the tests (the target)**
- [ ] Do they test **behaviour a user cares about**, or just that the code runs without crashing?
- [ ] Would they **fail if the feature were broken**? A good habit: ask Claude to break the feature on purpose and confirm a test goes red.
- [ ] **Gaming red flags:**
  - assertions loosened (`== 42` became `is not None`)
  - `skip`/`xfail`/`.only` added
  - expected values copied from current output
  - heavy mocking of the very thing under test
  - a custom `__eq__` or `equals`
  - code that checks for test-specific inputs
- [ ] What's **missing**? Edge cases, empty input, errors, permissions, concurrency, large data.

**Step 2: review the code**
- [ ] **Scope:** does the diff match the plan? Unexpected files are a warning sign.
- [ ] **Logic:** walk one real input through the main path by hand. This is where AI code is weakest (CodeRabbit: logic and business-rule errors more than 2x).
- [ ] **Error handling:** swallowed exceptions, broad `try/except`, silent fallbacks, missing null checks.
- [ ] **Made-up things:** APIs, config keys or packages that don't exist or aren't in your versions.
- [ ] **Duplication:** a new helper that already exists elsewhere in the repo.
- [ ] **Security:** secrets, unchecked input, passwords or tokens handled carelessly, new dependencies.
- [ ] **Performance:** I/O or queries in loops (CodeRabbit: about 8x more excessive I/O in AI PRs).

**Step 3: use AI to help you review, then judge it yourself**
- Ask Claude: *"Explain this diff in plain words. What are you least sure about? What did you not test?"*
- Run a **fresh** reviewer (`/code-review` or a new subagent) with the plan. Tell it: *"Report gaps, not style preferences."*
  A reviewer that didn't write the code isn't biased toward it.
- **Run the feature yourself once.** Click through it; don't only trust the green check.
- Judge the findings yourself, since AI reviewers also miss things and over-report.

**Turn findings into harness:** each real problem you catch should become a test, a `CLAUDE.md` rule or a hook,
so you never have to catch it again (L2).

**Check yourself:** `verify.sh` is green, but the diff adds `@pytest.mark.skip` to two old tests and a new `__eq__` that returns `True`. Merge?
<details><summary>Answer</summary>No. Both are classic test gaming. Ask why, fix the real problem, and add a hook or rule that blocks new skips and suspicious equality overrides.</details>

**Read more:** [ImpossibleBench: measuring how often LLMs exploit test cases](https://arxiv.org/abs/2510.20270) ·
[CodeRabbit: AI vs human code report](https://www.helpnetsecurity.com/2025/12/23/coderabbit-ai-assisted-pull-requests-report/) ·
[Why AI coding agents fail on real codebases (video)](https://www.youtube.com/watch?v=VlCeTC6N9N8)

---

### L7. Why prompting tricks matter much less now

**What changed.** In 2023–2024, people collected "magic" prompt tricks: role-play ("You are a world-class engineer"),
threats and rewards, ALL-CAPS warnings, long templates. Frontier models in 2026 follow plain instructions well, and in agents
the prompt you type is a small part of what the model sees. Most of the context is files, tool output and history.
Anthropic describes the shift as **from prompt engineering to context engineering**: curating *everything* the model sees,
not wording one message ([Anthropic, Sep 2025](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents)).

**What matters much less**
- Persona lines, flattery, threats, "take a deep breath", tips.
- ALL-CAPS and stacking "IMPORTANT". The official docs advise using emphasis on single lines only, because overuse makes rules blur together.
- Huge prompt templates you paste every time. Put stable rules in `CLAUDE.md` or skills once instead.

**What still matters (and is not a trick)**
1. **Be specific about the task.** Scope, files, the existing pattern to follow, the symptom. Compare *"write a test for foo.py
   covering the case where the user is logged out; avoid mocks"* with *"add tests"* (from the official best practices).
2. **Give a way to verify.** *"Give Claude a check it can run: tests, a build, a screenshot to compare"* (best practices). This is L1.
3. **Ask for honesty and an exit.** Telling the agent it may stop and flag a problem instead of forcing a pass reduced test
   cheating a lot for some models in ImpossibleBench. That's the idea behind `NEEDS-HUMAN.md`.
4. **Give context, not commands.** Say *why* (users are on mobile, this runs every minute), and Claude makes better trade-offs.

**The takeaway:** spend your effort on the **target** (L1), the **harness** (L2), **context** (L5) and **review** (L6).
Those compound across every session. A clever prompt helps once.

**Check yourself:** your prompt starts with "You are a 10x senior engineer. CRITICAL: NEVER write bugs!!!" What would help more?
<details><summary>Answer</summary>Delete that line. Say exactly what to build and where, point to an existing pattern, and give the command that proves it works (./verify.sh). Put lasting rules in CLAUDE.md.</details>

**Read more:** [Effective context engineering for AI agents (Anthropic)](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) ·
[Claude Code best practices](https://code.claude.com/docs/en/best-practices)

---

### L8. The lethal trifecta and the Rule of Two

**The problem.** A language model can't reliably tell *your* instructions from instructions hidden in text it reads.
So any text an attacker can write, like a GitHub issue, an error report, a web page or a README inside a dependency,
can steer your agent. This is **prompt injection**. It's not hypothetical: in 2026 fake Sentry error reports made
Claude Code, Cursor and Codex leak credentials in 85% of tests ([CSA](https://labs.cloudsecurityalliance.org/research/csa-research-note-agentjacking-mcp-sentry-injection-20260612/)),
and PR titles made AI agents in CI print their API keys.

**Why filters don't fix it.** In 2025, researchers from OpenAI, Anthropic and Google DeepMind attacked 12 published
defenses. Adaptive attacks beat most of them more than 90% of the time, and human red-teamers beat all of them
([summary](https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/)). In web security, 95% blocked is a
failing grade.

**The idea.** Stop asking "can the agent be tricked?" (yes) and ask "what could a tricked agent *do*?"
Simon Willison's **lethal trifecta** ([Jun 2025](https://simonwillison.net/2025/jun/16/the-lethal-trifecta/)): trouble needs all three of

| | Capability | Coding example |
|---|---|---|
| **A** | Reads untrusted input | Issues, error reports, web pages, MCP tool output |
| **B** | Has sensitive access | `~/.ssh`, cloud keys, `.env`, production database |
| **C** | Can change things or send data out | Shell commands, `git push`, network access |

Meta's **Rule of Two** turns it into a rule you can follow: *one session gets at most two of A, B and C; if it truly
needs all three, a human approves the risky steps* ([Meta, Oct 2025](https://ai.meta.com/blog/practical-ai-agent-security/)).
A coding agent almost always has C. So in practice: **keep secrets out of its reach** (Posting 02's default), and
read outside text in a **separate triage session with no secrets** that writes a notes file you read.

**What you do:** before a session that reads outside text, ask "does this session hold anything worth stealing?"
If yes, split the work (Posting 02 §4). Production changes are never an agent job: Claude writes the script, you run it.

**Check yourself:** you want Claude to read 20 new GitHub issues and fix the easy ones, in one session, using a
GitHub token that can push. Which of A, B, C does that session have, and what would you change?
<details><summary>Answer</summary>All three: issues are untrusted input (A), the token is sensitive access (B), pushing changes things (C). Split it: a triage session with a read-only token summarizes the issues into a file; you read it; a build session without outside text does the fixes and you push.</details>

**Read more:** [The lethal trifecta (Willison)](https://simonwillison.net/2025/jun/16/the-lethal-trifecta/) ·
[Agents Rule of Two (Meta)](https://ai.meta.com/blog/practical-ai-agent-security/) ·
[Agentic AI and Security (Martin Fowler's site)](https://martinfowler.com/articles/agentic-ai-security.html) ·
[OWASP Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026)

---

### L9. Sandbox, permissions and auto mode: three different layers

**The problem.** People hear "Claude Code has a sandbox and auto mode" and assume they're protected. Each layer
stops something different, and each has a gap.

| Layer | What it is | What it stops | What it doesn't stop |
|---|---|---|---|
| **Permissions** (`allow` / `ask` / `deny` in settings) | Rules about which tools and commands Claude may use | Exactly what you list. **Deny rules hold in every mode** | Anything you didn't list; a `Read` deny can be dodged by a Python one-liner in Bash |
| **Sandbox** (`/sandbox`, off until you enable it) | The OS limits what Bash commands can touch: files, network | Commands reaching hosts you didn't allow, writing outside the project | By default it can **read almost your whole machine**, including `~/.aws` and `~/.ssh`, until you add `sandbox.credentials` denies. It covers Bash only, not MCP servers |
| **Auto mode** (default since Aug 2026 on Pro, Max, Team) | A classifier judges risky actions before they run | Most overeager mistakes; humans approve 93–97% of prompts, so it beats tired humans | Its own miss rate is 17% on real overeager actions; **file edits inside the project skip it**, so insecure code changes aren't checked ([Anthropic](https://www.anthropic.com/engineering/claude-code-auto-mode), [Backslash](https://www.backslash.security/blog/claude-code-auto-mode-catches-a-lot-but-not-everything)) |

**The idea: defense in depth.** Use all three, plus hooks (L2) for the things you need guaranteed:
- **Sandbox** with your credential files and token env vars denied, and a short network allowlist.
- **Permissions:** deny reads of `.env` files and secret folders, deny force-push and publishing, ask before `git push`.
  Allow only narrow commands: `Bash(git *)` also allows `git push --force`.
- **Hooks** for what auto mode skips: block edits to `.claude/`, `.mcp.json` and CI files; check packages before install.
- **Auto mode** on top, to catch the mistakes you didn't think of.

**What you do:** ask Claude to apply Posting 02 §3, read the diff, then run the **self-test** (Posting 02 §5). If Claude
can print your `~/.aws/credentials`, a layer is missing.

**Check yourself:** auto mode is on. Is that enough to stop Claude from disabling certificate checks in your HTTP client code?
<details><summary>Answer</summary>No. File edits inside the project skip the classifier. Insecure code changes are caught by review and scanners (L6, /security-review, Semgrep), not by auto mode.</details>

**Read more:** [Claude Code sandboxing docs](https://code.claude.com/docs/en/sandboxing) ·
[Auto mode, how it works (Anthropic)](https://www.anthropic.com/engineering/claude-code-auto-mode) ·
[Sandbox ladder and credentials (Krupa)](https://bartlomiejkrupa.dev/articles/claude-code-security-sandboxing-2026/)

---

### L10. Secrets and the supply chain

**The problem, part 1: secrets.** 28.65 million secrets leaked on public GitHub in 2025, up 34%. Commits co-written by
Claude Code leaked at 3.2% versus a 1.5% baseline, and MCP config files alone exposed 24,008 secrets because setup
guides tell you to paste keys into config ([GitGuardian, Mar 2026](https://blog.gitguardian.com/the-state-of-secrets-sprawl-2026/)).
And an agent that *finds* a powerful key may use it: in April 2026 an agent used a token it found to delete a
production volume and its backups in about 9 seconds.

**The idea: three tiers.** Decide for each secret where it lives:
1. **Never on your machine:** production database, live payment keys, cloud admin, package publish tokens. They live in
   your hosting provider's or CI's secret store. The agent can't misuse what isn't there.
2. **On the machine but unreadable by the agent:** SSH keys, cloud CLI logins, personal `.env.local`. Denied by sandbox and permissions.
3. **OK for the agent:** dev database, payment test-mode keys, an API key with a spending cap. Assume it could leak; keep it cheap to rotate.

Plus: keys in environment variables, never in committed files; expiry dates on every token; push protection and `gitleaks`
to catch mistakes. **When in doubt, rotate first and investigate second.**

**The problem, part 2: packages.** Two attacks hit AI-assisted developers:
- **Slopsquatting:** models invent package names (4–6% of the time for frontier models in a May 2026 study), and attackers
  register those names with malware inside ([CSA](https://labs.cloudsecurityalliance.org/research/csa-research-note-slopsquatting-ai-supply-chain-20260419-csa/)).
- **Worms:** Shai-Hulud (2025) and Mini Shai-Hulud (May 2026) used **install scripts** to steal tokens from developer
  machines and CI, then republished themselves into more packages ([CSA](https://labs.cloudsecurityalliance.org/research/csa-research-note-mini-shai-hulud-supply-chain-sigstore-2026/)).

**The idea: slow down and don't run strangers' code at install time.**
- An **install gate** hook checks that a package exists and isn't brand new before Claude installs it.
- A **minimum release age** (pnpm `minimumReleaseAge`, Bun, Yarn): new versions wait a day or more, by which time most
  poisoned versions have been pulled.
- **Block dependency install scripts** (pnpm and Bun do this by default; npm needs `ignore-scripts`).
- **Commit the lockfile** and look at lockfile changes in PRs.
- **Plugins, skills and MCP servers are packages too.** 76 malicious skills were found in one public marketplace in
  Feb 2026 ([Snyk](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)). Read a skill before installing it.

**What you do:** sort your project's secrets into the three tiers once (Posting 02 §3.2), switch to pnpm or Bun with a
release-age setting, and keep the install gate hook on.

**Check yourself:** Claude suggests `npm install fast-json-sanitizer-pro` to fix a bug. The install gate says the package
was published 3 days ago. What do you do?
<details><summary>Answer</summary>Don't approve it yet. Ask Claude which library's official docs recommend it. A 3-day-old package with a plausible name is exactly what slopsquatting looks like. Use a well-known package, or write the small function yourself.</details>

**Read more:** [State of Secrets Sprawl 2026 (GitGuardian)](https://blog.gitguardian.com/the-state-of-secrets-sprawl-2026/) ·
[pnpm on npm supply chain security](https://pnpm.io/blog/2025/12/05/newsroom-npm-supply-chain-security) ·
[GitHub: preparing for the next malware campaign](https://github.blog/2025-12-20-strengthening-supply-chain-security-preparing-for-the-next-malware-campaign/) ·
[ToxicSkills (Snyk)](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)

---

## Worked example: starting a new project from zero

Say you start a new **habit-tracker web app** (TypeScript, a small API plus a web UI). Here is how every lesson applies,
in order. You type about ten short messages. Claude does the rest.

**Day 0, once per machine: secure the workstation (≈1 hour).** Give Claude
[Posting 02](postings/02-secure-agent-workstation.md) and say *"Use this md file to secure my machine."* Approve the
sandbox and credential settings, then run its self-test. Every later project inherits this (L8–L10).

**Day 1, morning: kickoff and harness (≈20 min)**

| # | You do or type | What happens | Lesson |
|---|---|---|---|
| 1 | `mkdir habit-tracker && cd habit-tracker && git init && claude` | New repo, new session | L5 (fresh session) |
| 2 | `/plugin install superpowers@claude-plugins-official` | Installs the method | L3 |
| 3 | Copy `postings/01-superpowers-daily-workflow.md` into the repo and type: *"Use this md file for my project, building a habit tracker web app: users log daily habits and see streaks. TypeScript, simple API + web UI."* | Claude reads the posting, sees there's no harness yet, and proposes one | Postings |
| 4 | Claude proposes the stack, `CLAUDE.md`, `verify.sh` (lint + typecheck + unit + Playwright acceptance tests), the target-lock hook, the Stop gate and the no-push-to-main hook. **You read it and say yes** | The project harness exists before any feature code | L1, L2 |
| 5 | Check: `verify.sh` runs and passes on the empty skeleton (the "clean baseline") | The sensor works | L1 |

**Day 1, afternoon: first feature, "log a habit and see a streak" (Track A)**

| # | You do or type | What happens | Lesson |
|---|---|---|---|
| 6 | *"Build: a user can create a habit, check it off for today, and see the current streak."* | **brainstorming** asks you a few questions, one at a time (Do streaks break on a missed day? Time zones?) and offers 2–3 approaches | L3 |
| 7 | **Gate 1:** read the spec in `docs/superpowers/specs/`. Fix anything wrong now; it's the cheapest moment | The target is now agreed | L1, L3 |
| 8 | **Write the target:** make sure the spec's acceptance criteria became tests in `tests/acceptance/` (e.g. "missing a day resets the streak to 0") | The locked target. Claude can't edit it later | L1, L2, L6 |
| 9 | **Gate 2:** read the plan in `docs/superpowers/plans/`. Check the **test list** first. Approve | Small tasks, each with its tests | L3, L6 |
| 10 | *"Execute the plan"*, or hand it to a loop: `/goal ./verify.sh exits 0 and all tasks in the plan are checked off` | Claude builds test-first and task by task. The Stop gate re-runs `verify.sh` whenever Claude tries to finish | L1, L4 |
| 11 | Go do something else | If the same failure repeats 3 times, `NEEDS-HUMAN.md` appears and Claude stops | L1, L4 |

**Day 1, evening: review**

| # | You do or type | What happens | Lesson |
|---|---|---|---|
| 12 | **Gate 3:** Claude opens a PR. You review **tests first** (any loosened assertions? skips?), then the code (scope, logic, errors, security) | You catch what green checks can't | L6 |
| 13 | *"Explain this diff. What are you least sure about?"* plus `/code-review` | A second opinion from a fresh context | L6 |
| 14 | Click through the feature yourself once, then merge | Done means verified *and* checked by you | L6 |
| 15 | You noticed Claude used local time instead of the user's time zone. *"Add a CLAUDE.md rule and a test so dates always use the user's time zone."* | The mistake can't happen again | L2 |

**Day 2: next feature and a bug**

| # | You do or type | What happens | Lesson |
|---|---|---|---|
| 16 | `/clear` (new task = new session) | Clean context. The spec, plan and `CLAUDE.md` carry the memory | L5 |
| 17 | *"Fix: streak shows 0 after midnight for users in Asia/Seoul. Steps: …"* | Track B: **systematic-debugging** finds the root cause, adds a regression test, then fixes | L3, L6 |
| 18 | *"Quick fix, skip the workflow: change the button text to 'Done today'."* | Track D: small edit, but the Stop gate still runs `verify.sh` | L3 |
| 19 | `NEEDS-HUMAN.md` appeared on the export feature: "Same failure 3x: CSV test expects UTF-8 BOM" | You decide: is the code stuck, or is the **target wrong**? Here the test was wrong, so **you** fix the test (it's locked to Claude, not to you) and restart | L1, L6 |
| 20 | Long session getting slow or forgetful? *"Write state to docs/progress.md"*, then `/clear`, then *"continue from docs/progress.md"* | Fresh, focused context without losing decisions | L5 |

**What you never did:** write hooks by hand, memorize Superpowers skills, or craft clever prompts (L7).
**What you always did:** define the target, approve at the gates, review tests before code, and turn mistakes into harness.

---

## Glossary

- **Agent:** a model plus a harness, working in a loop (think → use a tool → look at the result → repeat).
- **Harness:** everything around the model: tools, rules, memory, checks. Claude Code is one. Your `CLAUDE.md` and hooks extend it.
- **Hook:** a script Claude Code runs automatically at a set moment (before a tool, on stop, etc.).
- **Skill:** a markdown file of instructions Claude loads when relevant. **Plugin:** a package of skills, hooks and commands.
- **Subagent:** a helper Claude starts with a fresh context for one job. It returns only the result.
- **SDD (spec-driven development):** write and approve *what* to build before building it.
- **TDD (test-driven development):** write a failing test first, then the code that makes it pass.
- **Sensor / `verify.sh`:** the one command whose exit code says whether the work is done.
- **NEEDS-HUMAN:** the file the Stop hook writes when Claude is stuck on the same failure 3 times.
- **Compaction:** Claude Code summarizing old conversation when the context window is full.
- **Context rot:** the model getting worse at using its context as the context grows or fills with stale content.
- **Context engineering:** choosing what goes into the model's context (files, tool output, history), not just wording the prompt.
- **Test gaming / reward hacking:** making checks pass without solving the task (editing tests, hardcoding expected inputs, overriding `==`).
- **Ultracode / dynamic workflows:** Claude Code mode that runs many subagents in parallel through a generated script.
- **Prompt injection:** hidden instructions in text the agent reads (an issue, a web page, an error report) that steer it.
- **Lethal trifecta:** untrusted input + sensitive access + a way to change things or send data out, in one agent. Together they let an attacker steal data.
- **Rule of Two:** give one agent session at most two of those three; if it needs all three, a human approves the risky steps.
- **Sandbox:** the operating system limiting which files and network hosts a command can reach. In Claude Code it covers Bash commands.
- **Auto mode:** Claude Code's default permission mode (Pro, Max, Team) where a classifier, not you, approves most actions. Helpful, not a security boundary.
- **Slopsquatting:** registering a package name that AI models tend to invent, so that installs of the made-up name get malware.
- **Install script:** code a package runs automatically when it's installed. The main way npm worms spread.
- **Minimum release age:** a package manager setting that refuses versions published less than N days ago.
- **MCP server:** a plugin that gives the agent new tools (GitHub, a browser, a database). Code running with your permissions; its output is untrusted input.

---

## How this README grows

Every new posting must also update this README. That's how this repo stays a learning guide:

1. Add a row to the **Postings** table.
2. For each new idea the posting relies on, add a **lesson** (`L7`, `L8`, …) in the same format:
   *the problem → the idea → what you do → check yourself → read more*. Add it to the **Learning path** table too.
3. If the posting changes an existing idea (for example, a new Claude Code feature replaces a step), **update that lesson**
   instead of adding a duplicate, and note what changed.
4. Add new terms to the **Glossary**, and bump the **Last updated** line.
