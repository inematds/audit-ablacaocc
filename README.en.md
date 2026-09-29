# audit-ablacaocc — ablation audit for Claude Code

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

A **diagnostic** (read-only) skill that audits your `CLAUDE.md`, skills, and hooks, classifies each instruction, and proposes a minimal version — without touching any files.

> Every line in your configuration is **guilty of complexity until proven useful**.

## 📖 User Guide

Complete guide (landing page + walkthrough): **https://inematds.github.io/audit-ablacaocc/guia/en/**

- Skill: [`SKILL.md`](SKILL.md)
- Conceptual foundation: [`doc/`](doc/) — summary of Boris Cherny’s talk (Y Combinator, one day after the launch of Opus 5)

---

## Table of Contents

1. [Boris’s perspective: why delete your own config](#1-boris’s-perspective-why-delete-your-own-config)
2. [Why this cleanup is useful in practice](#2-why-this-cleanup-is-useful-in-practice)
3. [What the skill does (and never does)](#3-what-the-skill-does-and-never-does)
4. [Installation](#4-installation)
5. [How to use it](#5-how-to-use-it)
6. [Usage tip: call skills all the time](#6-usage-tip-call-skills-all-the-time)
7. [Recommended cycle](#7-recommended-cycle)
8. [Practical guide: publishing to Git and the portal](#8-practical-guide-publishing-to-git-and-the-portal)

---

## 1. Boris’s perspective: why delete your own config

Boris Cherny created Claude Code at Anthropic. His thesis, in summary:

**Configuration gets stale.** Every instruction you write is a fix for a specific model’s weakness. When the next model no longer has that weakness, the instruction becomes dead weight — and keeps being reread every time, taking up context and limiting the model.

That’s literally what happened in Claude Code: much of the system prompt existed only to correct behaviors the model should have been able to handle on its own. With Opus 5, the team **removed more than 80% of the system prompt**. What remains is almost entirely safety, permissions, static analysis, and interface.

**Ablation** is the method. It’s a research term: you remove something to measure its impact. Applied to prompts, it means deleting everything and bringing it back one line at a time, discovering what each line actually does. Boris’s advice for anyone who only *uses* Claude Code:

> Every ~6 months, especially with a major model release: delete your `CLAUDE.md`, delete your skills, delete your hooks — and see what the model does without them.

Rebuilding has four steps, and the fourth is the one that matters:

1. Delete the config.
2. Use the tool on **real work**, not a hypothetical test.
3. Observe where it does well and where it stumbles.
4. Only bring back an instruction **after you’ve seen the model repeatedly fail at the same thing**.

There are two reasons for this discipline: you’re a poor predictor of which instructions the model needs, and every retained line costs context on every run.

The other pillars of his perspective:

- **Over-specification is the most common mistake** — and it’s more common among people with years of engineering experience. Scripting “do A, then B, then C” worked with older models; today it keeps the model from finding a better way. Write **task + guardrails + completion criteria**, then come back to check.
- **Verification is the lever almost everyone gets wrong.** A short prompt with a real way for the model to check its own work beats a huge prompt without verification. The classic example: rewrite the Electron app in Swift, run the original in a VM, take a screenshot, compare pixel by pixel, and don’t stop until it’s done. Produce → observe → compare → stopping condition.
- **Aim above the ceiling.** Give it a task slightly harder than you think the model can handle, and retest your old failures with every new version — the reason for the failure may have disappeared.
- **Working with models has become an empirical science**, not a theoretical one. It’s more like getting to know a living creature than configuring a system. The right level of instruction is the same as what you’d use to guide a coworker.
- **There’s no secret trick.** The cycle is: hard task → ways to verify → observe where it gets stuck → fix *that* (better prompt, or a skill, or an MCP) → repeat.

To test this yourself, Boris mentioned two levers: the system-prompt flag at startup, and `CLAUDE_CODE_SIMPLE=1` (removes all system prompts, including those for tools — the ablation baseline used internally). Counterintuitive detail: without the prompts, the model gets *slightly smarter*; they’re kept because they make the product behave as people expect.

## 2. Why this cleanup is useful in practice

Anyone who has used Claude Code for a while accumulates the same layer of sediment:

| Symptom | What it usually is |
|---|---|
| A 300-line `CLAUDE.md` with dated exceptions | fixes for models that are no longer around |
| The same rule in `CLAUDE.md` and in 3 skills | redundancy that causes conflict when one copy changes |
| A skill that teaches “how to think” in 400 lines | generic reasoning the model already does on its own |
| A rigid 12-step walkthrough | micromanagement that blocks a better route |
| Two rules that contradict each other | the model picks one, and you don’t know which |
| Nothing saying *how to check* that it’s correct | the most costly gap of all |

The cost is real and compounded: context burned on every call, reduced autonomy, inconsistent behavior, and zombie rules no one dares delete because no one knows what they’re holding together anymore.

The benefits of cleanup:

- **Context returned** to the real work.
- **Less contradiction** — a rule that exists in only one place can’t conflict with anything.
- **More autonomy** — completion criteria let the model find a better path than yours.
- **Auditable config** — you know *why* each line is there because each one survived a question.
- **Verification is part of the equation** — the audit exposes where there’s no objective way to prove the work is done correctly.

The goal is **not** to maximize reduction. It’s to maximize:

**quality + autonomy + verifiability ÷ complexity**

That’s why the skill explicitly preserves what the model can’t infer: project identity, paths and sources of truth, branding, security, compliance, integrations, interface contracts, and internal conventions.

## 3. What the skill does (and never does)

**It does:**

- read `CLAUDE.md` (project + global), skills, hooks, and `settings.json`;
- classify each instruction — `CONTEXT`, `GUARDRAIL`, `QUALITY CRITERION`, `VERIFICATION`, `INTEGRATION/TOOL`, `REPEATABLE PROCEDURE`, `MICROMANAGEMENT`, `REDUNDANCY`, `LEGACY/OBSOLETE`, `AMBIGUOUS/UNPROVEN`;
- decide `KEEP` / `SIMPLIFY` / `MOVE` / `MERGE` / `TEST` / `REMOVE` (when in doubt, `TEST`);
- audit each skill — `KEEP` / `SIMPLIFY` / `MERGE` / `SPLIT` / `LOAD-ON-DEMAND` / `CONVERT-TO-CONTEXT` / `DELETE-CANDIDATE`;
- deliver a report in 10 sections: executive summary, metrics, issues by file, removal candidates, redundancies and conflicts, skills, proposed minimal `CLAUDE.md`, proposed skills, ablation test plan (versions A/B/C), and top 10 changes by impact ÷ risk.

**It never does:** edit, move, delete, overwrite config, or commit. It’s diagnostic. Applying changes is your decision, in a separate request — by design: an audit that starts changing things is one you can’t trust.

Nearby distinction: `memory-audit` handles session **memory**. This one handles **agent configuration**.

## 4. Installation

```bash
git clone https://github.com/inematds/audit-ablacaocc.git
mkdir -p ~/.claude/skills/audit-ablacao
cp audit-ablacaocc/SKILL.md ~/.claude/skills/audit-ablacao/SKILL.md
```

Or, to install only in the current project:

```bash
mkdir -p .claude/skills/audit-ablacao
cp /caminho/audit-ablacaocc/SKILL.md .claude/skills/audit-ablacao/SKILL.md
```

Restart the Claude Code session so it loads the skill.

## 5. How to use it

Direct invocation:

```
/audit-ablacao
```

Or trigger it using natural language:

- “audit my global CLAUDE.md for ablation”
- “audit this project’s skills, my prompt is bloated”
- “what can I remove from my config without breaking anything?”

Scopes worth auditing:

| Scope | When |
|---|---|
| Global (`~/.claude/`) | every ~6 months or with a major model release |
| One project | when its `CLAUDE.md` exceeds ~150 lines |
| A set of skills | when 3+ skills compete for the same trigger |

After the report, a healthy workflow is: choose the **Top 10 by impact ÷ risk**, apply them in another session, and run the **ablation plan** (versions A / B / C) on real project tasks before deciding the change is good.

## 6. Usage tip: call skills all the time

This is a practice I’ve adopted that changes the result more than any prompt tweak: **use skills as much as possible — by calling them directly or within projects.**

Why:

- **A skill is a procedure, not a global instruction.** A rule in `CLAUDE.md` is read on *every* run, including the 90% that have nothing to do with it. The same rule inside a skill costs context only when the task calls for it. Moving a procedure from `CLAUDE.md` into a skill is the cheapest way to slim down the config without losing anything — that’s exactly what `MOVE` and `LOAD-ON-DEMAND` mean in the report.
- **Calling it directly (`/skill-name`) eliminates trigger roulette.** You stop depending on the description matching your phrasing. When several skills compete on the same subject, explicit invocation breaks the tie — and removes the need for a routing rule in `CLAUDE.md`.
- **A skill inside the project (`.claude/skills/`) is versioned with the code.** The procedure travels with the repo, goes into the PR, is reviewable, and doesn’t leak into other projects.
- **A skill is the right unit for a fix.** In Boris’s cycle, when the model stumbles, you have three remedies: a better prompt (the instruction was unclear), a **skill** (a repeatable procedure is missing), or an MCP (context is missing that it can’t reach). Choosing the right remedy prevents the reflex to dump another rule into `CLAUDE.md`.
- **A skill is easy to audit and retire.** It has a boundary, name, and scope. You can disable a skill and measure; disabling “that paragraph in the middle of `CLAUDE.md`” is much harder.

In practice: keep `CLAUDE.md` only for what is true **always** (identity, guardrails, sources of truth, security). Everything else — procedure, format, recipe, integration — becomes a skill, invoked directly when you know what you want.

## 7. Recommended cycle

1. **Run** `/audit-ablacao` on the chosen scope.
2. **Cut** the Top 10 by impact ÷ risk (in a separate session — the skill doesn’t apply them).
3. **Use it on real work** for a few days. Not on a hypothetical test.
4. **Record failures** and bring back an instruction only when the same failure repeats — in the shortest form possible.
5. **Repeat** every ~6 months or with every major model release.

## 8. Practical guide: publishing to Git and the portal

The process followed by this repository itself. It applies to any INEMA skill/project.

### 8.1 Repo structure

```
audit-ablacaocc/
├── SKILL.md        # the skill (root = installable with one cp)
├── README.md       # this file
├── capa/capa.png   # 1280x720 cover (capa-inema skill)
├── guia/index.html # self-contained landing page + guide (projetos-landing-guia skill)
├── doc/            # source material
└── .github/workflows/pages.yml   # GitHub Pages deployment via Actions
```

A rule that prevents rework: **the guide goes in `guia/`, never in the root** — the root is for code/skills. And it’s **one page inside the repo itself**, never a separate repository. Remote repo name = local folder name.

### 8.2 Git — from scratch to push

```bash
cd ~/projetos/audit-ablacaocc

git init -b main
git config user.name  "inematds"
git config user.email "inematds@gmail.com"   # author matches the destination account
git add -A
git commit -m "feat: skill audit-ablacao + guia"

gh repo create inematds/audit-ablacaocc --public --source=. --remote=origin --push
```

Time-saving check: confirm `git config user.email` **before** committing. An author that doesn’t match the destination account is the #1 reason deployments get blocked.

If the push fails because of `.github/workflows/*.yml` (missing `workflow` scope on the HTTPS token), push that commit over SSH:

```bash
git push git@github.com:inematds/audit-ablacaocc.git main
```

### 8.3 GitHub Pages

Publish **via GitHub Actions**, not the legacy build (“deploy from a branch”): the legacy queue gets stuck, with builds stuck at `building` for hours and deployment returning `Deployment failed, try again later`.

Create `.github/workflows/pages.yml` serving from the root (the guide remains at `/guia/`):

```yaml
name: Deploy static site to Pages
on:
  push: { branches: [main, master] }
  workflow_dispatch:
permissions: { contents: read, pages: write, id-token: write }
concurrency: { group: pages, cancel-in-progress: false }
jobs:
  deploy:
    environment: { name: github-pages, url: "${{ steps.deployment.outputs.page_url }}" }
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with: { path: '.' }
      - id: deployment
        uses: actions/deploy-pages@v4
```

Then point Pages to Actions and watch the run:

```bash
gh api -X POST repos/inematds/audit-ablacaocc/pages -f build_type=workflow \
  || gh api -X PUT repos/inematds/audit-ablacaocc/pages -f build_type=workflow
gh run watch <run-id> --repo inematds/audit-ablacaocc --exit-status
```

Confirm before saying it’s published:

```bash
curl -sI https://inematds.github.io/audit-ablacaocc/guia/en/ | head -1   # expects 200
```

The first run takes ~1 min. Since the guide is in `guia/`, the public URL ends in `/guia/`.

### 8.4 Portal (inema.club)

With the Pages URL responding, publish to the 3 INEMA surfaces (portal, inemabuscas, and PRO catalog):

```
/atualiza-portal https://inematds.github.io/audit-ablacaocc/guia/en/
```

The skill handles the card, cover, and commit + push to all three repositories.

### 8.5 Definition of “published”

Publishing = **commit + push to `origin`**. Deployment is automatic via git webhook → Vercel (and GitHub Pages, when applicable) and isn’t under our control: don’t open the dashboard, check the status, or trigger another deployment. It’s done when the push reaches `origin` and, for Pages, when the file responds.

---

## Credits

Method and perspective: Boris Cherny (Anthropic), talk at Y Combinator about cutting 80% of Claude Code’s system prompt. Summaries and source material in [`doc/`](doc/). Packaged as a skill by INEMA.
