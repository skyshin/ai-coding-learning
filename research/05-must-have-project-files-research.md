# Research notes: which project files are must-have for agent-driven development

> **What this file is:** the evidence and step-by-step reasoning behind
> [Posting 05](../postings/05-must-have-project-files.md). The posting tells you *what to create*. This file shows
> *why*, with sources, so you can judge the choices yourself. Read it once; you don't hand it to Claude.

Researched: **November 2026**, against Claude Code 2.1.3xx. Sources are 2025–2026 web, industry and academic
writing, in English. Where a source sells a product, it says so.

---

## Step 1. Define the question precisely

"Must-have" is easy to say and hard to pin down. I used this definition:

> A file is **must-have** for a project if leaving it out causes a failure that a production project can't absorb:
> the agent builds the wrong thing, breaks a rule nobody wrote down, can't prove it's done, or nobody can operate or
> recover the system once it's live.

And **production scale, not POC**, means four things a prototype can skip:

1. **Real users and real data.** Auth, data loss and security mistakes now hurt someone.
2. **It must keep running.** Someone has to deploy, roll back, read logs and restore a backup.
3. **It lives for months.** Decisions and vocabulary must survive many sessions and `/clear`s.
4. **More than one mind works on it.** Several sessions, subagents, loops, or people, who share nothing but the repo.

The prototype-to-production gap is well documented. A vendor survey of AI-built apps lists the missing pieces as
database, authentication and authorization, audit trail, code security, cost control and a deployment path
([Superblocks, Aug 2026](https://www.superblocks.com/blog/vibe-coding-production-wall), vendor). Another write-up
lists database configuration, auth flows and deployment, and code "optimized for the happy path" without retries,
failure modes or monitoring ([8080.ai, 2026](https://dev.to/8080_ai/ai-built-apps-and-the-production-gap-what-the-60-failure-rate-is-actually-telling-us-584j), vendor; its statistics are second-hand).

## Step 2. Collect every candidate file the sources recommend

| Source | Files it recommends |
|---|---|
| OpenAI, *Harness engineering* ([Feb 11 2026](https://openai.com/blog/harness-engineering)) | ~100-line `AGENTS.md` as a table of contents; `ARCHITECTURE.md`; `docs/` with `design-docs/`, `exec-plans/{active,completed}/`, `product-specs/`, `references/`, `generated/db-schema.md`, plus `DESIGN.md`, `FRONTEND.md`, `PLANS.md`, `PRODUCT_SENSE.md`, `QUALITY_SCORE.md`, `RELIABILITY.md`, `SECURITY.md` |
| Anthropic, *Effective harnesses for long-running agents* ([Nov 2025, updated Oct 2026](https://anthropic.com/engineering/effective-harnesses-for-long-running-agents)) | `init.sh` (start app + smoke test), `claude-progress.txt`, a feature list in **JSON**, git history |
| Claude Code docs, [memory](https://code.claude.com/docs/en/memory) and [best practices](https://code.claude.com/docs/en/best-practices) | `CLAUDE.md` (< 200 lines), `CLAUDE.local.md`, `.claude/rules/` (path-scoped), skills in `.claude/skills/`; `AGENTS.md` read since v2.1.277 |
| SDD tools ([Spec Kit, Kiro, OpenSpec, BMAD](https://codemyspec.com/blog/spec-driven-development), Jun 2026) | `constitution.md`, per-feature `spec.md`/`requirements.md` (EARS), `design.md`, `plan.md`, `tasks.md`; BMAD: PRD + architecture + story files |
| PRD writers ([Haberlah, Jan 2026](https://medium.com/@haberlah/how-to-write-prds-for-ai-coding-agents-d60d72efb797); [ChatPRD](https://www.chatprd.ai/resources/prd-for-ai-codegen), vendor; [Crosley, Feb 2026](https://blakecrosley.com/blog/prd-driven-development)) | `docs/PRD.md` with goals, users, features, constraints, non-goals, acceptance criteria, "files not to modify" |
| Domain modeling ([Pocock's skill, Aug 2026](https://aihero.dev/skills-domain-modeling); [Phoenix, 2026](https://understandingdata.com/posts/domain-glossary-as-agent-constraint/)) | `GLOSSARY.md` (formerly `CONTEXT.md`), `GLOSSARY-MAP.md` for several bounded contexts, `docs/adr/NNNN-*.md` |
| Decision records ([Davidson, O'Reilly Radar, Oct 2 2026](https://oreilly.com/radar/coding-agents-love-decision-records/)) | `docs/decisions/` ADRs with `Date:` / `Supersedes:` headers |
| Security ([OWASP Secure Coding with AI](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_with_AI_Cheat_Sheet.html); [VirtusLab, Nov 9 2026](https://virtuslab.com/blog/ai/hardening-the-loop-security-good-practises-for-agentic-development/)) | `SECURITY.md`, `THREAT_MODEL.md`, `CODEOWNERS` for agent config, CI, Docker and deploy files |
| This repo's Postings 01–04 | `verify.sh`, `tests/acceptance/`, hooks in `.claude/settings.json`, `NEEDS-HUMAN.md`, Superpowers specs and plans, gstack design doc, pstack verification skill |

That is **more than 30 candidate files**. Nobody needs all of them. Steps 3 and 4 cut the list.

## Step 3. Apply five filters

### Filter 1: Does it hold something the agent can't work out from the code?

This is the strongest evidence in the whole topic.

- **ETH Zurich + LogicStar, *Evaluating AGENTS.md*** ([arXiv 2602.11988](https://arxiv.org/html/2602.11988), ICLR 2026;
  [InfoQ summary, Mar 2026](https://infoq.com/news/2026/03/agents-context-file-value-review/)). Agents including Claude
  Code were tested with no context file, an LLM-generated one and a developer-written one. Abstract: context files
  "do not generally improve task success rates, while increasing inference cost by over 20% on average".
  **LLM-generated** files slightly *lowered* success; **developer-written** files slightly raised it (a few points,
  not statistically significant). Agents *did* follow the instructions (used the named tools, ran more tests).
  "Repository overviews, although popular and recommended by model providers, are not helpful."
  Recommendation: only write what isn't already in the code or README, such as specific tooling and conventions.
- **Claude Code docs** give the same test for every line of `CLAUDE.md`: *"Could it find this by reading the code?
  Would it actually go and look?"* If yes, cut it. `/doctor` suggestions also cut directory layouts, dependency lists
  and architecture overviews.
- **Böckeler** (Thoughtworks, [Feb 2026](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)):
  build context files up gradually, and watch for the tendency to "overengineer the context with unnecessary,
  copied & pasted instructions".

**Consequence:** no `/init`-generated file goes in unreviewed. No directory tree, no dependency list, no
"this repo is a web app" overview. Files hold **decisions, rules, vocabulary, commands and intent**: things code
can't say.

### Filter 2: One fact, one home (no duplication)

Duplicated facts drift apart, and then the agent gets two answers. OpenAI lists "it rots instantly" and "hard to
verify" as two of four ways a big instruction file fails. Davidson says each rule should be stated once and
cross-referenced. So each candidate was merged into the one file that owns its question:

| Candidate | Merged into | Why |
|---|---|---|
| `SDD.md`, `constitution.md` | Not a file. SDD is a *process* that writes per-feature specs. Project-wide rules go in `AGENTS.md` | Superpowers already writes `docs/superpowers/specs/` and `plans/` (Posting 01). A second SDD tool would be a second method |
| Kiro-style `design.md` per feature | The feature's spec (Superpowers specs include the design) | Same question, "how will this feature work?" |
| `DESIGN.md` / system design | `ARCHITECTURE.md` + ADRs | Architecture = the current map; ADRs = why it looks this way |
| `Ontology.md` | `GLOSSARY.md` | See filter 5 |
| `claude-progress.txt`, `PLANS.md`, `exec-plans/` | The Superpowers plan file (checkboxes) + `docs/progress.md` only during long loops | The plan already tracks tasks |
| JSON feature list | The PRD's acceptance criteria → `tests/acceptance/` | The locked tests *are* the machine-checkable feature list, and they're protected by a hook (stronger than "JSON is less likely to be edited") |
| `init.sh` | `verify.sh` (and the pstack verification skill for driving the app) | One sensor, not two |
| gstack design doc (Posting 03) | **Becomes** `docs/PRD.md` | Same question, "what and why?" Don't write both |
| `CLAUDE.md` *and* `AGENTS.md` with content | `AGENTS.md` holds content; `CLAUDE.md` is a one-line `@AGENTS.md` import plus Claude-only lines | See step 3b |
| `QUALITY_SCORE.md`, `RELIABILITY.md`, `PRODUCT_SENSE.md` | Not adopted | OpenAI's scale (a product built by agents, staffed for it). Their own post says it "should not be assumed to generalize without similar investment" |
| `CONTRIBUTING.md`, `CHANGELOG.md`, `TODO.md` | README / git history / the plan or issues | No unique content until you have outside contributors or published releases |

### Step 3b: `CLAUDE.md` vs `AGENTS.md`, decided

Facts (from the [Claude Code memory docs](https://code.claude.com/docs/en/memory#agents-md) and
[InfoWorld, Sep 21 2026](https://infoworld.com/article/4224410/claude-code-now-also-accepts-instructions-in-openais-agents-md-format.html)):

- Since **v2.1.277 (Sep 18 2026)** Claude Code reads `AGENTS.md`, **but only when there is no `CLAUDE.md`**
  (or `.claude/CLAUDE.md` or `CLAUDE.local.md`) in the folder or above. If both exist, `CLAUDE.md` wins and
  `AGENTS.md` is ignored, silently.
- The "Project instructions" setting (`/config`) can load both, but it **can't be committed** to a repo; it's ignored
  in project settings files.
- Sessions that don't fetch feature flags (telemetry off, first session after an upgrade) don't read `AGENTS.md`
  directly. An `@AGENTS.md` import from `CLAUDE.md` always works.
- The docs' recommended pattern: keep `AGENTS.md` as the shared file, create `CLAUDE.md` with `@AGENTS.md` at the
  top, then Claude-only content below. A symlink also works, but Claude's Edit/Write tools refuse to write through it,
  and Windows git may check it out as a plain file.

**Decision:** `AGENTS.md` holds all shared content; `CLAUDE.md` is a shim of a few lines. It costs one tiny file,
works on every Claude Code version and setting, and the same instructions work in Codex, Cursor, Copilot and others.
If you're certain only Claude Code will ever touch the repo, a single `CLAUDE.md` is fine too. What you must never
do is put content in both.

### Filter 3: Is it a rule that must never bend? Then it's not a doc

Claude Code's docs say `CLAUDE.md` is "not a hard enforcement layer"; never-rules belong in hooks, deny rules or
managed settings. pstack's enforcement ladder (Posting 04): architecture > types/lint > tests > docs last. VirtusLab:
"AI doesn't know your security rules. It knows only what is written down, and follows only what is enforced." So
each doc file is paired with the check that enforces it (`verify.sh`, a hook, a CI-required check, `CODEOWNERS`).

### Filter 4: Does it pay for itself at this project size?

- **Specs:** the only dated SDD benchmark ([Uvik, Q4 2026](https://uvik.net/blog/spec-driven-development-benchmark/),
  vendor; 50 production Python tickets, Claude Sonnet 5, ±10-point noise) found spec frameworks raised merge rates on
  **features and data pipelines** (90% vs 60% control) and **large tickets** (80% vs 50%), and cut review defects
  (0.46 vs 0.86 per merged ticket). On **small 1–2 file tickets, bug fixes, refactors and test-writing**, no gain,
  and more cost. Specs that covered 6+ of 8 checklist items merged 89% vs 60%. Its rule: write a spec when **3 or
  more** hold: 3+ files, 4+ acceptance criteria, a module boundary crossed, a schema or public API change, more than
  one engineer or agent, regulated code.
- A field log ([devmoment, Aug 2026](https://www.devmoment.dev/journal/spec-kit-vs-kiro-field-log-2026)) found Spec
  Kit's spec "longer than the diff" on a two-file fix, and a plain plan "most of the benefit at a tenth of the setup".
- **Glossary:** Pocock: "On a one-day build, skip it", and an unreviewed agent-written glossary is "worse than none".
  agentpatterns.ai: it backfires on greenfield code that refactors weekly, and earns its keep with non-trivial domain
  terms and several bounded contexts.
- **ADRs:** write one only when the decision is **hard to reverse, surprising without context, and a real
  trade-off** (all three) ([agentpatterns.ai, Jun 2026](https://www.agentpatterns.ai/instructions/ubiquitous-language-for-ai-plans/)).
  Davidson's warnings: agents follow obsolete "accepted" ADRs rigidly, so mark superseded ones; keep them short, not
  "courtroom transcripts".

### Filter 5: "Ontology" — formal or plain?

None of the 2026 coding-agent sources recommends a formal ontology (OWL/RDF) for agent context. All of them
recommend a plain markdown glossary: one name per concept, banned synonyms, relationships and invariants, flagged
ambiguities. Phoenix: a root glossary is "the cheapest way I have found to stop an agent loop from quietly inventing
a new vocabulary every iteration." **Decision:** call the file `GLOSSARY.md` (the word "ontology" overpromises).
A formal ontology is a *product* feature (knowledge graphs, data integration), not a project doc.

## Step 4. Tier the survivors by project size

I tiered by **risk and number of minds**, not lines of code, because the failure modes in step 1 scale with those.
Any single *risk trigger* moves a project up one tier: handles money; stores health, financial or other sensitive
personal data; regulated; more than one deployable; several agents or people working in parallel.

| File | Small | Medium | Large | The question it answers |
|---|---|---|---|---|
| `README.md` | ● | ● | ● | What is this, how do I run, test and operate it? (humans) |
| `AGENTS.md` + `CLAUDE.md` shim | ● | ● | ● | What must an agent know that the code can't tell it? |
| `docs/PRD.md` | ● (1 page) | ● | ● (+ per-area specs) | What are we building, for whom, why, what's out? |
| `verify.sh` + `tests/acceptance/` + hooks | ● | ● | ● (+ CI-required) | How do we prove it's done? |
| Feature specs + plans (Superpowers) | ◐ risky features | ● | ● | How will this feature work, step by step? |
| `ARCHITECTURE.md` | — | ● | ● | Where do things live; what must never be crossed? |
| `docs/adr/` | — | ◐ as needed | ● | Why is it built this way? |
| `GLOSSARY.md` | — | ◐ if domain-heavy | ● (+ map) | What exactly do our words mean? |
| `SECURITY.md` | ◐ 5 lines in AGENTS.md | ● | ● (+ `THREAT_MODEL.md`) | What must we protect, and from whom? |
| `RUNBOOK.md` | ◐ "Operate" section in README | ● | ● | How do we deploy, roll back, recover? |
| `.claude/rules/` path-scoped | — | ◐ | ● | Conventions for one area only |
| Verification skill (pstack) | — | ● | ● | How does an agent drive the real app as proof? |
| `docs/progress.md` | only during long loops | only during long loops | ● during loops | Where did the last session stop? |

● must-have · ◐ conditional · — skip

## Step 5. Map the tools (not files) onto the tiers

| Tool | Small | Medium | Large | Evidence / reason |
|---|---|---|---|---|
| Posting 02 workstation | ● | ● | ● | Once per machine; prerequisite for any real data |
| Superpowers (Posting 01) | ◐ Track A for auth/payments/data features, Track D for the rest | ● | ● | Uvik: specs pay on features and large tickets, not small fixes |
| `/goal` loop | ● | ● | ● | Cheapest way to keep the sensor in charge |
| Fresh-context loops (ralph / `claude -p`) | — | ◐ | ● | Anthropic's long-running harness pattern: progress file + git |
| gstack office hours (Posting 03) | ◐ if it's a business | ◐ | ◐ | Output becomes the PRD |
| pstack verification skill, `/interrogate`, `/blast-radius` (Posting 04) | — | ● (risky PRs) | ● | Proof ladder level 4 |
| pstack hillclimb | — | ◐ | ◐ | Only with a frozen metric |
| Scheduled doc-gardening | — | ◐ | ● | OpenAI: docs rot; recurring agent opens fix-up PRs |
| `CODEOWNERS` on agent config/CI/deploy | — | ● | ● | OWASP: treat rules files as security-critical config |

## Step 6. Check for things I might have missed

- **Docs rot.** Every source warns about it. Mitigation in the posting: a `check-docs` step in `verify.sh` (paths
  referenced by docs exist, `AGENTS.md` stays short) and, for large projects, a scheduled doc-gardening run.
- **Agents writing their own rules.** OWASP: don't let the agent modify its own rules files without approval.
  Posting 02's config-protection hook already covers `CLAUDE.md`; the posting extends it to `AGENTS.md`.
- **Over-documentation.** Uvik measured 2.4–12.5% spec drift; devmoment: "The heavier the tool, the more doc there is
  to rot." So the posting creates files **lazily**, on a trigger, not on day 1.
- **Subagents don't see everything.** Claude Code's Explore and Plan subagents start without `CLAUDE.md`; anything
  a subagent must obey belongs in the skill or prompt that starts it, or in a hook.
- **Monorepos.** Claude Code loads subdirectory `CLAUDE.md`/`AGENTS.md` and path-scoped rules on demand, so large
  projects put package-specific rules next to the package instead of growing the root file.

## Sources

- OpenAI, Ryan Lopopolo, "Harness engineering: leveraging Codex in an agent-first world" (Feb 11 2026): https://openai.com/blog/harness-engineering
- Anthropic, "Effective harnesses for long-running agents" (Nov 26 2025, updated Oct 2026): https://anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Gloaguen et al. (ETH Zurich, LogicStar.ai), "Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?" (arXiv 2602.11988, ICLR 2026): https://arxiv.org/html/2602.11988 · InfoQ (Mar 6 2026): https://infoq.com/news/2026/03/agents-context-file-value-review/
- Claude Code docs, Memory (incl. AGENTS.md) and Best practices (Nov 2026): https://code.claude.com/docs/en/memory · https://code.claude.com/docs/en/best-practices
- InfoWorld, "Claude Code now also accepts instructions in OpenAI's AGENTS.md format" (Sep 21 2026): https://infoworld.com/article/4224410/claude-code-now-also-accepts-instructions-in-openais-agents-md-format.html
- Developers Digest, "Claude Code now reads AGENTS.md: setup, modes and gotchas" (Oct 17 2026): https://www.developersdigest.tech/blog/claude-code-agents-md-support
- Birgitta Böckeler, "Context Engineering for Coding Agents" (martinfowler.com, Feb 5 2026): https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html
- Uvik, "Spec-Driven Development Benchmark 2026" (vendor, Sep–Nov 2026): https://uvik.net/blog/spec-driven-development-benchmark/
- devmoment, "Spec Kit vs Kiro vs plan.md: a 2026 field log" (Aug 2026): https://www.devmoment.dev/journal/spec-kit-vs-kiro-field-log-2026
- CodeMySpec, "Spec-driven development in 2026" (vendor, Jun 2026): https://codemyspec.com/blog/spec-driven-development
- Matt Pocock, "/domain-modeling skill" (Aug 24 2026): https://aihero.dev/skills-domain-modeling
- James Phoenix, "The domain glossary is a constraint, not documentation" (2026): https://understandingdata.com/posts/domain-glossary-as-agent-constraint/
- agentpatterns.ai, "Ubiquitous language for AI plans" (reviewed Jun 2 2026): https://www.agentpatterns.ai/instructions/ubiquitous-language-for-ai-plans/
- Duncan Davidson, "Coding agents love decision records" (O'Reilly Radar, Oct 2 2026): https://oreilly.com/radar/coding-agents-love-decision-records/
- Michael Haberlah, "How to write PRDs for AI coding agents" (Jan 12 2026): https://medium.com/@haberlah/how-to-write-prds-for-ai-coding-agents-d60d72efb797
- Blake Crosley, "PRD-driven development" (Feb 8 2026): https://blakecrosley.com/blog/prd-driven-development
- ChatPRD, "Writing PRDs for AI code generation tools" (vendor, 2026): https://www.chatprd.ai/resources/prd-for-ai-codegen
- OWASP, "Secure Coding with AI Cheat Sheet" (2026): https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_with_AI_Cheat_Sheet.html
- Łukasz Lenart, "Hardening the loop: security good practices for agentic development" (VirtusLab, Nov 9 2026): https://virtuslab.com/blog/ai/hardening-the-loop-security-good-practises-for-agentic-development/
- Superblocks, "The vibe coding production wall" (vendor, Aug 21 2026): https://www.superblocks.com/blog/vibe-coding-production-wall
- 8080.ai, "AI-built apps and the production gap" (vendor, 2026): https://dev.to/8080_ai/ai-built-apps-and-the-production-gap-what-the-60-failure-rate-is-actually-telling-us-584j
