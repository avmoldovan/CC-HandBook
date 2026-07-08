# Vibe Coding at Scale

## A Handbook for Building Autonomous Workflows in Claude Code

*Memory, skills, plugins, MCP servers, agents, hooks — and how to debug them when they collide.*

---

## Preface

This handbook is the systematic version of a conversation. It started as a series of questions about Claude Code's extension stack — memory and CLAUDE.md, then skills, then plugins, then MCP servers, then agents, then hooks, then debugging — and grew into a complete operating model for getting Claude Code to do real engineering work autonomously, with you in the loop only when judgment is required.

It targets people who have already used Claude Code enough to feel the pain points: rules that get ignored mid-session, skills that don't trigger when they should, agents that drift, parallel work that clashes, sessions that end with no record of what happened. The book's claim is that all of these are fixable, that the fix lives at a specific layer in the system, and that picking the right layer is the difference between vibe coding that feels chaotic and vibe coding that feels like a well-run pipeline.

The book assumes you're working on real code, not toys — codebases with conventions, multiple teams, real CI, production deployments, security review, documentation in Confluence, tickets in Jira. The examples reflect that. If you're using Claude Code purely for personal experiments, much of this is overkill; if you're using it to ship work that other people depend on, every chapter earns its place.

### How to read this book

The chapters build on each other, but every chapter stands alone. Three reading paths:

- **Linear** — start at chapter 1 and go through. Best if you're setting up Claude Code from scratch or formalizing an ad-hoc setup.
- **Reference** — jump to the chapter that matches the layer you're working on. Each part is self-contained.
- **Debug-first** — if something is broken right now, skip to Part VII and work backwards. The triage tree in chapter 19 answers most "Claude isn't doing X" questions in about two minutes.

Throughout, code blocks are meant to be read as examples, not copy-paste templates. The companion starter kit (see appendix D) has working versions of everything; this book explains why each piece looks the way it does.

### What's in scope, what isn't

In scope: every Claude Code feature that shapes Claude's behavior outside a single prompt. CLAUDE.md, rules, auto memory, skills, plugins, MCP servers, hooks, settings.json, subagents, agent teams, AskUserQuestion, the inspection commands, the debug flags.

Out of scope: prompt engineering inside a single message; programming the Claude API directly (this is about Claude Code specifically); enterprise admin topics like SSO and audit; non-Anthropic tools.

### A note on velocity

Claude Code changes fast. Features that were experimental six months ago are stable now; features stable today may be deprecated in a year. Where exact version-dependent behavior matters, the book calls it out and gives you the inspection command to verify your version. Where the conceptual model has been stable, the book just states it. The mental model and architectural patterns are durable even when specific commands rename themselves.

This edition is current to **Claude Code 2.1.204 (July 8, 2026)**. It was written around **Claude Opus 4.8** as the working model; note that as of 2.1.197 the *default* model in Claude Code is now **Claude Sonnet 5** (1M context), and **Claude Fable 5** (a Mythos-class model) is selectable since 2.1.170 — pick your model explicitly with `/model` if you depend on Opus. Changes between 2.1.132 and 2.1.158 are folded into the chapters and collected in **Appendix F — Release Delta**; everything from 2.1.159 through 2.1.204, including what has been **deprecated and removed**, is in **Appendix G**. If you're reading this later still, run `claude --version` and skim Appendix G first.

---

# Table of Contents

### Front Matter
- [Preface](#preface)
- [How to read this book](#how-to-read-this-book)
- [What's in scope, what isn't](#whats-in-scope-what-isnt)
- [A note on velocity](#a-note-on-velocity)

### Part I — Foundations
- [Chapter 1. The Mental Model](#chapter-1-the-mental-model)
- [Chapter 2. The Five Layers](#chapter-2-the-five-layers)

### Part II — Memory and Instructions
- [Chapter 3. CLAUDE.md and the Memory Hierarchy](#chapter-3-claudemd-and-the-memory-hierarchy)
- [Chapter 4. Path-Scoped Rules](#chapter-4-path-scoped-rules)
- [Chapter 5. Auto Memory](#chapter-5-auto-memory)
- [Chapter 6. Compaction and What Survives It](#chapter-6-compaction-and-what-survives-it)

### Part III — Procedures and Capabilities
- [Chapter 7. Skills](#chapter-7-skills)
- [Chapter 8. Where Skills Are Specified, Loaded, and Ignored](#chapter-8-where-skills-are-specified-loaded-and-ignored)
- [Chapter 9. The Skill-Creator Workflow](#chapter-9-the-skill-creator-workflow)
- [Chapter 10. Plugins](#chapter-10-plugins)
- [Chapter 11. MCP Servers](#chapter-11-mcp-servers)
- [Chapter 12. Combining Skills with MCP](#chapter-12-combining-skills-with-mcp)

### Part IV — Control Flow
- [Chapter 13. Hooks and the Deterministic Layer](#chapter-13-hooks-and-the-deterministic-layer)
- [Chapter 14. Preventing Loops](#chapter-14-preventing-loops)
- [Chapter 15. Automating with settings.json](#chapter-15-automating-with-settingsjson)
- [Chapter 16. AskUserQuestion and Workflows That Pause](#chapter-16-askuserquestion-and-workflows-that-pause)

### Part V — Agents
- [Chapter 17. Where Agents Fit in the Stack](#chapter-17-where-agents-fit-in-the-stack)
- [Chapter 18. Subagents, Forks, and Agent Teams](#chapter-18-subagents-forks-and-agent-teams)
- [Chapter 19. The Trust Model for Autonomous Agents](#chapter-19-the-trust-model-for-autonomous-agents)
- [Chapter 20. When to Use Agents — and When Not To](#chapter-20-when-to-use-agents-and-when-not-to)

### Part VI — Building the Pipeline
- [Chapter 21. The Complete Cast of Agents](#chapter-21-the-complete-cast-of-agents)
- [Chapter 22. The Main Flow: Jira to Deploy](#chapter-22-the-main-flow-jira-to-deploy)
- [Chapter 23. The CVE-Remediation Variant](#chapter-23-the-cve-remediation-variant)
- [Chapter 24. The Atlassian Integration Layer](#chapter-24-the-atlassian-integration-layer)
- [Chapter 25. Everything Together: The Concentric Picture](#chapter-25-everything-together-the-concentric-picture)

### Part VII — Greenfield, Brownfield, and Scale
- [Chapter 26. Greenfield from Day One](#chapter-26-greenfield-from-day-one)
- [Chapter 27. Brownfield and Large Codebases](#chapter-27-brownfield-and-large-codebases)
- [Chapter 28. Established Teams](#chapter-28-established-teams)
- [Chapter 29. The Where-Does-This-Go Cheat Sheet](#chapter-29-the-where-does-this-go-cheat-sheet)

### Part VIII — Debugging the Stack
- [Chapter 30. The Triage Tree](#chapter-30-the-triage-tree)
- [Chapter 31. The Inspection Commands](#chapter-31-the-inspection-commands)
- [Chapter 32. Verbose Mode vs. Debug Mode](#chapter-32-verbose-mode-vs-debug-mode)
- [Chapter 33. Debugging Each Layer](#chapter-33-debugging-each-layer)
- [Chapter 34. The Clean-Room Technique](#chapter-34-the-clean-room-technique)
- [Chapter 35. A Worked Debugging Example](#chapter-35-a-worked-debugging-example)
- [Chapter 36. Building Debug Instrumentation](#chapter-36-building-debug-instrumentation)
- [Chapter 37. Common Failure Patterns Reference](#chapter-37-common-failure-patterns-reference)

### Appendices
- [Appendix A. The Layer Decision Matrix](#appendix-a-the-layer-decision-matrix)
- [Appendix B. Command and Flag Reference](#appendix-b-command-and-flag-reference)
- [Appendix C. Frontmatter Field Reference](#appendix-c-frontmatter-field-reference)
- [Appendix D. The Companion Starter Kit](#appendix-d-the-companion-starter-kit)
- [Appendix E. Further Reading](#appendix-e-further-reading)
- [Appendix F. Release Delta — Changes Since 2.1.132](#appendix-f-release-delta--changes-since-2-1-132)
- [Appendix G. Release Delta — 2.1.159 to 2.1.204 (Deprecations, Removals, and New Capabilities)](#appendix-g-release-delta--2-1-159-to-2-1-204-deprecations-removals-and-new-capabilities)

---

# Part I — Foundations

The pieces of Claude Code that extend its behavior beyond a single prompt — memory, skills, plugins, MCP, hooks, agents — feel like a menu of equivalent options when you first encounter them. They're not. They form a stack, with each layer answering a different question, and most "Claude is behaving strangely" complaints trace to putting something in the wrong layer. Part I lays out the mental model. The rest of the book is its consequences.

---

## Chapter 1. The Mental Model

Every Claude Code session starts with a fresh context window. To get persistence and capability across that boundary, you write four kinds of artifacts and one kind of guardrail. **CLAUDE.md** is *always-on context* — facts Claude must hold in every turn. **Auto memory** (`~/.claude/projects/<repo>/memory/MEMORY.md`) is the same idea, but Claude writes it for itself based on what it learns from you. **Skills** are *on-demand procedures* — multi-step playbooks Claude loads only when relevant, so they cost almost nothing until invoked. **Plugins** are the *packaging format* for skills + agents + hooks + MCP configs, so you can version and share them. **MCP servers** are the *external capability layer* — they expose tools (GitHub issues, Postgres queries, Sentry errors) that Claude calls during a session. **Hooks** are *deterministic gates* — shell commands that fire at lifecycle events whether Claude wants them to or not.

Anything you put in the wrong layer either burns context unnecessarily or fails to load when you need it.

The single decision rule that resolves 90% of "where does this go" questions: **if the information must be true in every session, it's CLAUDE.md or a `.claude/rules/` file; if it's a procedure invoked sometimes, it's a skill; if it's a capability against an external system, it's an MCP server; if you want to ship any of the above to others, wrap it in a plugin; if it must happen at a fixed lifecycle event no matter what, it's a hook.**

The rest of this book is the long form of that paragraph.

---

## Chapter 2. The Five Layers

Stepping back from the details: your full Claude Code setup is five concentric layers, each enforcing different guarantees.

The **innermost layer is CLAUDE.md and skill descriptions** — the standing instructions Claude reads every turn. Specific, concise, well-routed. This is where Claude *learns what to do*.

Around that, **skills** are the procedures Claude invokes when the description matches. Loaded on demand, cheap, support dynamic context injection. This is where Claude *learns how to do things*.

Around that, **agents** are the specialists who actually do the work. Own context, own tools, own permission posture, optionally own worktree. This is *who does the work*.

Around that, **MCP servers** are the external systems agents reach into — Jira, Confluence, GitHub, Sentry, Postgres. Tool Search defers schemas so context cost stays flat regardless of how many you connect. This is *what they can touch*.

Wrapping it all, **hooks** are the deterministic gates that fire regardless of what Claude decides. SessionStart for context injection, PreToolUse for blocking, PostToolUse for formatting, Stop and SubagentStop for quality gates, Notification for pulling you back in. This is *what must happen no matter what Claude wants*.

When you're staring at "Claude did the wrong thing again," the layer to fix is almost always the outer one. Wrong content? Skill body. Wrong procedure selection? Skill description. Wrong agent picked? CLAUDE.md routing. Wrong external action? MCP scope or agent's tool allowlist. Action that should never have happened? Missing hook. Each layer has a different lever, and most failures point at exactly one of them.

The implicit hierarchy of trust matters too. **Hooks are policy** — non-negotiable, automatic, programmable. **Permissions** (allow/deny rules) are also policy, applied at the tool-call boundary. **CLAUDE.md** is guidance — Claude tries to follow it but the model has discretion. **Skill descriptions** are routing hints — Claude picks among them. The first two enforce; the latter two inform. When you need a guarantee, you reach for the enforcement layer; when you need behavior shaping, you reach for the guidance layer. Most "Claude ignored my rule" complaints are guidance dressed up as if it were enforcement: a behavior in CLAUDE.md that should have been a hook.

# Part II — Memory and Instructions

Memory is where most teams' Claude Code setup either succeeds or quietly fails. CLAUDE.md is the contract Claude reads every session; auto memory is what Claude learns about you while it works; path-scoped rules let large codebases keep that contract small. This part covers all three, plus the subtle but consequential mechanics of what survives compaction.

---

## Chapter 3. CLAUDE.md and the Memory Hierarchy

Claude Code has two memory mechanisms that load at the start of every session: **CLAUDE.md files** that you write, and **auto memory** that Claude writes for itself (v2.1.59+). Both arrive as a user message after the system prompt — so they're context, not enforcement. The model "tries to follow" CLAUDE.md, but specific, concise, well-structured rules get followed more reliably than vague, sprawling ones.

CLAUDE.md lives at four scopes, loaded in this order from broadest to most specific (later files appear after earlier ones in context, which gives later instructions soft priority when they conflict):

| Scope | Location | Purpose |
|---|---|---|
| Managed policy | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS), `/etc/claude-code/CLAUDE.md` (Linux/WSL), `C:\Program Files\ClaudeCode\CLAUDE.md` (Windows) | Org-wide rules deployed via MDM/Group Policy/Ansible; cannot be excluded by users |
| User | `~/.claude/CLAUDE.md` | Personal preferences across all your projects |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team-shared, committed to git |
| Local | `./CLAUDE.local.md` | Personal, gitignored, project-specific |

Claude also walks up the directory tree from where you launched and concatenates every `CLAUDE.md` it encounters along the way, so a monorepo with `packages/api/CLAUDE.md` will get both the root and the package file when you're in `packages/api/`. Subdirectory CLAUDE.md files below your launch directory are *not* preloaded — they load on demand when Claude reads files inside them. This is exactly the behavior you want in a large monorepo: ambient context for the top-level rules, lazy load for the team-specific ones.

There's a hard rule worth memorizing: **target under 200 lines per CLAUDE.md, and 60–80 lines is what production teams converge on**. Everything in CLAUDE.md is reloaded every session, so each line is a recurring token cost. Production-tested teams (HumanLayer is one frequently cited example) keep their root file around 60 lines. The instinct to "document everything" here is wrong — you're not writing a wiki, you're writing a working contract.

### What belongs in CLAUDE.md vs. somewhere else

A clean test: **if a code reviewer would raise an eyebrow at a violation, the rule belongs in CLAUDE.md. If a violation would fail CI, the rule belongs in CI, not CLAUDE.md.** Never use CLAUDE.md as a substitute for automated enforcement.

Concretely, CLAUDE.md is the right home for build/test/lint commands, naming conventions, "always do X" rules, project layout facts, and the architectural decisions Claude can't infer from the code. It is the *wrong* home for multi-step procedures (those become skills), path-specific rules (those become `.claude/rules/` files), and things that must run at fixed lifecycle events (those become hooks). A useful concrete CLAUDE.md skeleton looks like this:

```markdown
# Project: <name>

## Build & test
- Install: `pnpm install`
- Test: `pnpm test` (run before any commit)
- Lint: `pnpm lint --fix`
- Type-check: `pnpm tsc --noEmit`

## Architecture (one-liner per area)
- `src/api/` — Express handlers, one file per resource
- `src/services/` — pure business logic, no I/O
- `src/db/` — Drizzle ORM, migrations in `src/db/migrations/`

## Conventions
- Named exports only (no default exports)
- 2-space indentation
- All API endpoints validate input with Zod
- Tests sit next to source as `*.test.ts`

## Workflow
- Read existing code before editing
- Prefer targeted edits over rewrites
- Never commit unless explicitly asked

## Skills to prefer for this repo
- Use the `spec-checker` skill before implementing any item from `prompt_plan.md`
- Use the `bug-investigator` skill for any test failure that doesn't reproduce in one read
```

That last section is a routing hint — it teaches Claude *which skill to pick* without spending lines on the skill content itself. This is the pattern that solves the "Claude doesn't auto-select the right skill" pain point: a one-line routing rule in CLAUDE.md, with the heavy content in the skill where it belongs.

### The `@import` syntax

CLAUDE.md supports `@path/to/file` imports up to five hops deep. Imports load *at session start* alongside the host file, so they don't save context — they're for *organization*, not lazy loading. The right use is to factor a long file into focused pieces while keeping them all preloaded:

```markdown
# Project Instructions

@docs/architecture-overview.md
@docs/git-workflow.md
@~/.claude/preferences/typescript.md

## Project-specific
- Use feature flags from `flagsmith`, not environment variables
- Frontend uses Tanstack Query; never write raw fetch calls
```

If you want lazy loading, that's a skill or a path-scoped rule — not an import.

---

## Chapter 4. Path-Scoped Rules

For larger projects, splitting instructions across `.claude/rules/*.md` files is the right move. Each file covers one topic. Rules without frontmatter load unconditionally; rules with a `paths:` field only load when Claude reads matching files:

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/handlers/**/*.ts"
---

# API Development Rules

- All endpoints validate input with Zod schemas in `src/schemas/`
- Return errors via `respondWithError(res, code, message)` — never throw
- Add an OpenAPI doc comment to every handler
```

This is exactly the right primitive for a brownfield monorepo: you keep the root `CLAUDE.md` lean, and each subteam contributes path-scoped rules that only burn context when Claude is actually working on their code. Rules also support symlinks, which is what enables a "central rules repo" pattern — one shared rules folder symlinked into many projects, so a standards update propagates without copy-paste.

The decision between CLAUDE.md and `.claude/rules/*.md` with `paths:` is straightforward. A rule that applies to the entire codebase ("never use default exports") goes in CLAUDE.md. A rule that applies only to a subtree ("API handlers always validate with Zod") goes in a path-scoped rule. Rules with `paths:` cost zero tokens when Claude isn't touching matching files — that's the whole point.

---

## Chapter 5. Auto Memory

Auto memory shipped in v2.1.59 (late February 2026). It lives at `~/.claude/projects/<repo>/memory/`, scoped per git repository (so all worktrees of the same repo share one memory directory). `MEMORY.md` is the index — its first 200 lines or 25KB are loaded at the start of every session. Topic files (`debugging.md`, `api-conventions.md`, anything Claude creates) are *not* preloaded; Claude reads them on demand using its file tools when needed.

The mental model: **CLAUDE.md is what you teach Claude. Auto memory is what Claude learns.** When you correct Claude ("we use pnpm, not npm"), auto memory captures that without you typing `#`. When Claude figures out a non-obvious build flag while debugging, it writes that down. You see "Writing memory" and "Recalled memory" in the UI when this happens.

The right division of labor: **routing rules and standing instructions go in CLAUDE.md, because they need to fire every session. Learned facts and incidental discoveries go in auto memory, because they're discovered, not designed.** A common mistake is putting routing rules in auto memory — auto memory is preloaded too, but only its first 200 lines, and topic files don't preload at all. Putting "always check for `MEMORY.md` at session start" in auto memory is a bet that it'll survive in the top 200 lines forever, which it won't.

You can browse, edit, and delete auto memory at any time with `/memory` from inside a session. Disable it per-session with `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` or via `autoMemoryEnabled: false` in settings. For most setups, leave it on — the cost is negligible and the learnings compound.

---

## Chapter 6. Compaction and What Survives It

This catches everyone. When auto-compaction fires (or you trigger `/compact` manually), the *root* CLAUDE.md is re-read from disk and re-injected. Nested CLAUDE.md files in subdirectories are *not* re-injected — they reload only when Claude next reads a file in that subdirectory. Skill descriptions are also re-attached after compaction, but only the first 5,000 tokens of each invoked skill, sharing a 25,000-token combined budget, oldest dropped first.

The practical rule: **anything that must survive compaction lives in the root CLAUDE.md.** Mid-conversation corrections go into auto memory (or, if they're general enough, get promoted to CLAUDE.md). Manually `/compact` before auto-compact fires, because auto-compact tends to run at the lowest-intelligence point of a long session.

The defense against compaction surprises is a SessionStart hook with matcher `compact`. The hook fires *after* compaction completes, on the same session, and its stdout becomes Claude's context. Use it to re-inject anything critical that was dropped:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'REMINDER: Use pnpm not npm. Tests must pass before commit. Current sprint: auth refactor. See CLAUDE.md.'"
          }
        ]
      }
    ]
  }
}
```

This costs almost nothing and makes the difference between "I lost the plan after compaction" and "compaction is invisible." The `InstructionsLoaded` hook is the diagnostic complement: it logs exactly which CLAUDE.md and rules files were loaded and when, so you can verify the re-injection actually happened.

# Part III — Procedures and Capabilities

CLAUDE.md tells Claude *what to do*. Skills tell Claude *how to do specific things*. Plugins package those into shareable units. MCP servers extend what Claude can reach. This part covers the procedure-and-capability layers in detail.

---

## Chapter 7. Skills

Skills replaced custom commands (the old `.claude/commands/*.md` files still work, but skills are the supported primitive going forward). A skill is a directory under `.claude/skills/<name>/` (or `~/.claude/skills/<name>/` for personal) containing a `SKILL.md` with YAML frontmatter and instructions. The key property: **skill content does not load into context until the skill is invoked.** Only the name and description are visible to Claude at session start, so you can have dozens of skills without paying for them.

### The anatomy of a useful skill

A minimal but well-described skill that solves the auto-selection problem looks like this:

```markdown
---
name: spec-checker
description: Validates an implementation against the matching spec in specs/ before code is committed. Use when the user mentions implementing a numbered spec, when reviewing a diff that claims to implement a spec, or when prompt_plan.md references a spec file. Reads specs/NNNN-*.md, lists acceptance criteria, and verifies each one against the working tree.
allowed-tools: Read Grep Glob Bash(git diff *) Bash(git status *)
---

## Inputs
- $ARGUMENTS: the spec number (e.g. `0042`) or filename

## Procedure
1. Locate `specs/${ARGUMENTS}-*.md` (use Glob if exact filename not given)
2. Extract the "Acceptance Criteria" section
3. For each criterion, run a targeted grep/test against the working tree
4. Produce a checklist: ✅ verified, ❌ missing, ⚠️ partial
5. If any ❌ or ⚠️, list the exact file and line where the gap is

## Output format
Always end with a one-line summary: "Spec NNNN: X/Y criteria met"
```

The description field is what makes Claude *auto-select* the skill — it's loaded into context at session start so Claude can match user requests against it. This is where the "Claude doesn't auto-select my skill" pain point gets solved: write descriptions like routing rules, with concrete trigger phrases ("when the user mentions implementing a numbered spec", "when prompt_plan.md references…"). The combined `description + when_to_use` text is capped at 1,536 characters per skill, so make every word count and put the key use case first.

### Invocation control

Two frontmatter fields gate who can invoke a skill:

`disable-model-invocation: true` means only you can invoke it with `/skill-name`. Use this for anything with side effects: `/deploy`, `/commit`, `/send-slack-message`. You don't want Claude deciding to deploy because your code looks ready.

`user-invocable: false` means only Claude can invoke it (hidden from the `/` menu). Use this for background reference knowledge — a `legacy-system-context` skill explains how an old system works, but `/legacy-system-context` isn't a meaningful command for you to type.

### Skills that run in a forked subagent

This is the feature that enables forked-context skills to run in clean contexts. Add `context: fork` and `agent: Explore` (or your custom agent name) and the skill runs in an isolated subagent, returning only its summary to your main conversation:

```markdown
---
name: bug-investigator
description: Investigates a stubborn bug in isolation. Use when a test has failed twice or a fix has been attempted unsuccessfully. Runs in a forked context with read-only tools and worktree isolation to avoid polluting the main session.
context: fork
agent: Explore
allowed-tools: Read Grep Glob Bash(git log *) Bash(git blame *)
---

Investigate the bug described in $ARGUMENTS.

1. Reproduce the failure: identify the failing test or assertion
2. Read the relevant source — start from the failure site, follow imports
3. Use git blame and git log to check whether this code recently changed
4. Form three hypotheses, ranked by likelihood
5. Return: hypothesis ranking, evidence for each, recommended next step

Do not modify files. The main session will apply fixes based on your report.
```

This is the highest-leverage token move in long sessions: push exploration into subagents so their final report is the only thing that returns to main context. Pair `context: fork` with a worktree (configured at the agent level) and you've solved the parallel-subagent file-clash problem too.

### Dynamic context injection

The `` !`<command>` `` syntax in a skill body runs a shell command *before* the skill content is sent to Claude, inlining the output. This is what makes skills feel like "smart commands" rather than static prompts:

```markdown
---
name: review-diff
description: Reviews the current uncommitted diff for risks before commit.
---

## Current diff
!`git diff --staged`

## Changed file list
!`git diff --staged --name-only`

## Test status (last run)
!`cat .test-cache/last-run.txt 2>/dev/null || echo "no recent run"`

## Task
Review the diff above. Flag: missing error handling, hardcoded secrets, missing tests for new code paths, breaking API changes.
```

You can disable this for security-sensitive environments with `disableSkillShellExecution: true` in managed settings, but for personal/team use it's transformative — every invocation gets fresh, real data.

### Skills lifecycle and context budget

When a skill is invoked, its rendered SKILL.md becomes a message in the conversation and *stays there for the rest of the session*. Claude Code does not re-read the skill file on later turns. So write the body as **standing instructions that apply throughout a task**, not as one-time steps. Keep SKILL.md under 500 lines; move detailed reference material into sibling files (`reference.md`, `examples.md`, `scripts/helper.py`) and reference them from SKILL.md so Claude loads them only when needed. This is "progressive disclosure" — the same pattern Anthropic uses for its own built-in skills like `pdf` and `xlsx`.

### Skills vs. CLAUDE.md: a worked example

Suppose you want to enforce "all API handlers validate input with Zod." If this rule is short, universally true for `src/api/`, and just one of dozens of small rules, it goes in `.claude/rules/api.md` with a `paths:` field — it's a fact, always-on for matching files.

But suppose you have a *workflow* for adding a new API endpoint: scaffold the file, write the Zod schema, register the route in the router, write the handler, write the test, update OpenAPI docs. That's six steps with branching logic. It doesn't belong in CLAUDE.md — it'd burn 30 lines on something used once a week. It's a skill: `.claude/skills/new-endpoint/SKILL.md` with a description like "Use when the user asks to add a new API endpoint or implement a new route." Claude auto-loads it when relevant; you don't pay for it the other 90% of the time.

---

## Chapter 8. Where Skills Are Specified, Loaded, and Ignored

This is the most misunderstood corner of the system, so it deserves a careful answer. There are three different things people mean when they say "specify a skill":

**(a) Making a skill *exist* and *discoverable*.** This happens by putting `SKILL.md` in the right directory: `.claude/skills/<name>/`, `~/.claude/skills/<name>/`, a plugin's `skills/` folder, or a managed enterprise location. That's all you need. At session start, Claude Code scans these directories and loads each skill's **name + description (and `when_to_use`)** into context — that's roughly 1,536 characters per skill, with the listing budget defaulting to 1% of the context window. The skill's *body* doesn't load until invoked.

**(b) Telling Claude *when* to auto-pick a skill.** This is what the `description` and `when_to_use` fields do, and it's where the "Claude doesn't auto-select my skill" pain point lives. The model decides which skill to invoke by matching your prompt against these descriptions, so they must read like routing rules. Be "pushy" — the failure mode Anthropic documents is *under*-triggering, not over-triggering. A weak description is "Reviews code." A strong one is "Reviews code for security issues, missing error handling, hardcoded secrets, and missing tests. Use whenever the user asks to review, audit, or check code, even if they don't say 'review' explicitly. Also trigger on diffs, PR descriptions, or any uncommitted changes."

**(c) *Routing* to a skill from another piece of context.** This is the part that confuses people. You can mention a skill name in CLAUDE.md, in auto memory, in another skill, in `.claude/rules/*.md`, or in a system prompt. The mention itself doesn't load the skill — it nudges Claude to invoke it via the Skill tool. So **the right place to write routing rules is CLAUDE.md** (root), because CLAUDE.md reloads in full every session and survives compaction.

The "what's actually loaded vs. ignored" table:

| Location | What loads at session start | What's ignored / lazy |
|---|---|---|
| Root `CLAUDE.md` and `CLAUDE.md` files in directories above CWD | **Full content, every session, every compaction** | Nothing — all of it loads |
| `CLAUDE.md` in subdirectories below CWD | **Nothing** at start | Loads only when Claude reads a file in that subdir |
| `.claude/rules/*.md` without `paths:` frontmatter | **Full content, every session** | Nothing |
| `.claude/rules/*.md` with `paths:` frontmatter | **Nothing** at start | Loads when Claude touches a matching file |
| `MEMORY.md` (auto memory entrypoint) | **First 200 lines or 25KB**, whichever is smaller | Anything past that is ignored at start |
| Auto memory topic files (`debugging.md`, etc.) | **Nothing** at start | Loaded on-demand by Claude using file tools |
| `SKILL.md` frontmatter (name + description + when_to_use) | **Description text**, capped at 1,536 chars per skill | Body content (only loaded on invocation) |
| `SKILL.md` body | **Nothing** at start | Loads only when the skill is invoked, then stays for the session |
| Skill `disable-model-invocation: true` | Description **not** in context, skill is hidden from auto-routing | Body loads only when you type `/skill-name` |
| Skill `user-invocable: false` | Description **is** in context (Claude can invoke), hidden from `/` menu | Body loads when Claude decides to invoke |
| `.claude/agents/*.md` (subagent definitions) | Agent name + description visible to main Claude | Body loads only when delegated to |
| Plugin manifest (`plugin.json`) | Plugin metadata only | Plugin's skills/agents/hooks load like their unpacked equivalents |
| `.mcp.json` and MCP configs | **Server names + tool names** only (Tool Search defers schemas) | Tool schemas load on-demand |
| Imported files via `@path` in CLAUDE.md | **Full content** of imported file, every session | Nothing — `@import` is *not* lazy loading |
| HTML comments (`<!-- ... -->`) in CLAUDE.md | **Ignored** entirely (stripped before context injection) | — |

The most consequential row in that table is the auto-memory one. Auto memory has a hard 200-line/25KB cutoff for what loads from `MEMORY.md` — anything past that is invisible until Claude reads it on demand. This is exactly why **routing rules belong in CLAUDE.md, not in auto memory**: in auto memory they're betting that they stay in the top 200 lines forever; in CLAUDE.md they always load. The same goes for "always check for spec files at session start" — that's a routing rule, not a learning, so it lives in root CLAUDE.md.

A concrete CLAUDE.md routing block that solves the auto-selection problem:

```markdown
## Skill routing (read this every session)

- For any task referring to a numbered spec (e.g. "implement spec 0042",
  "is spec 0017 done") → invoke the `spec-checker` skill first.
- For any test failure that doesn't reproduce in one read, or any bug
  you've attempted to fix once → invoke `bug-investigator` (it runs in
  a forked worktree, so it won't pollute the main context).
- Before producing the end-of-session report → invoke `report-generator`
  (it uses Sonnet, so it's cheap to run).
- Never invoke `deploy-staging` or `rotate-credential` automatically;
  those require an explicit `/deploy-staging` or `/rotate-credential` from me.
```

That single block does what 200 lines of inline instructions used to do — because each skill carries its full procedure in its own SKILL.md, the routing rule just has to be specific enough that Claude picks the right one.

### Newer skill controls (2.1.x)

A few capabilities have been added to the skill surface since this chapter's core was written:

**`disallowed-tools` in frontmatter** (2.1.152): skills *and* slash commands can now list tools to *remove* from the model while the skill is active, the inverse of `allowed-tools`. This is the clean way to sandbox a skill — e.g. a `summarize-incident` skill that should read and write but never run Bash or call a deploy MCP tool. It composes with `allowed-tools`: allow the narrow set you need, disallow the dangerous ones explicitly.

**`/reload-skills` and SessionStart `reloadSkills`** (2.1.152): you no longer have to restart the session to pick up a newly added or edited skill. Type `/reload-skills` interactively, or have a SessionStart hook return `reloadSkills: true` so a hook that *installs* skills makes them available in the same session. This matters for the bootstrap pattern where a SessionStart hook pulls the team plugin and you want its skills live immediately.

**`${CLAUDE_EFFORT}` in skill bodies** (2.1.120) and `effort:` frontmatter: a skill can read the active effort level in its content and branch on it, and skill/agent `effort:` frontmatter can pin the effort for that unit of work. Combined with the hook-side `$CLAUDE_EFFORT` (Chapter 13), effort is now a first-class signal you can thread through the whole pipeline.

**`skillOverrides` setting** (working as of 2.1.129): controls visibility per skill — `off` hides it from both the model and `/`, `user-invocable-only` hides it from the model (so it only runs when you type it), and `name-only` collapses the description to just the name to save listing budget. Use `name-only` to keep a large skill catalog discoverable without paying full description cost for every entry.

One reliability fix worth knowing if you run subagents: through 2.1.132, subagents sometimes failed to discover project/user/plugin skills via the Skill tool; that was fixed in 2.1.133. If you saw "subagent can't find the skill that the main session can," upgrading past 2.1.133 resolves it.

---

## Chapter 9. The Skill-Creator Workflow

The `skill-creator` is itself a skill, distributed by Anthropic. In Claude Code you add it via `/plugin marketplace add anthropics/skills` and then `/plugin install skill-creator` (or clone the GitHub repo and load locally via `--plugin-dir`). It exposes four operating modes that cover the full skill development lifecycle: **Create, Eval, Improve, Benchmark.**

The mental model: you don't write SKILL.md by hand from a blank file. You invoke `skill-creator` and have a conversation. It interviews you about the use case, drafts the SKILL.md, generates triggering test cases, then optionally runs an automated optimization loop on the description.

A typical creation session goes like this. You start with `/skill-creator create`. Claude (now driven by the skill-creator's instructions) asks you to describe two or three concrete use cases — *real* repetitive work, not "a helpful skill in the abstract." It asks what the inputs look like, what Claude should do with them, what the output format must be, what constraints apply. Then it drafts `SKILL.md` with frontmatter and body, picks a name, generates 10–15 trigger evaluation prompts (queries that should activate the skill, plus distractors that should *not*), and offers to test.

The **Eval** mode runs a chosen skill against an eval set, scoring trigger rate (does Claude pick this skill when it should?) and execution quality (does the skill produce the right output when triggered?). Under the hood there are four agents: an **Executor** runs the skill against eval prompts; a **Grader** scores outputs against the expected criteria; a **Comparator** does blind A/B comparisons between two versions of a skill; an **Analyzer** suggests targeted improvements based on failure patterns. The output is an HTML report you can open in a browser to inspect what passed and what didn't.

The **Improve** mode runs an automated optimization loop: split your eval set 60/40 into train/test, run the current description three times per query to get a reliable trigger rate, ask Claude to propose improved descriptions based on what failed, re-evaluate, iterate up to 5 times. It picks the best description by *test* score (not train) to avoid overfitting, and writes the winner back to your SKILL.md.

The **Benchmark** mode aggregates results with variance analysis — useful when you want to know whether a description change is a real improvement or just noise. This is also what you wire into CI to catch regressions when Claude Code updates change triggering behavior.

A practical command sequence:

```text
# Inside Claude Code
/plugin install skill-creator
/skill-creator create
  → describe spec-checker workflow
  → accept the draft SKILL.md
  → confirm the generated eval set
/skill-creator eval spec-checker
  → review the HTML report
/skill-creator improve spec-checker --max-iterations 5
  → review the winning description
  → accept and write to disk
/skill-creator benchmark spec-checker
  → confirm the improvement is real
```

One caveat: AskUserQuestion had a regression in some 2026 versions (2.1.104) where it auto-completes with empty answers when invoked from inside plugin skills with `defaultMode: "bypassPermissions"`. Two related fixes have since landed: 2.1.136 fixed AskUserQuestion discarding multi-select answers supplied as an array, and 2.1.147 fixed auto mode suppressing AskUserQuestion when a user or skill explicitly relies on it. If you've automated heavily around AskUserQuestion-driven skills, check `claude --version` and upgrade past 2.1.147; pin only if you're stuck on an older build.

---

## Chapter 10. Plugins

A plugin is a directory containing `.claude-plugin/plugin.json` plus any combination of `skills/`, `agents/`, `hooks/hooks.json`, `.mcp.json`, `.lsp.json`, `monitors/monitors.json`, `bin/`, and `settings.json`. It's a packaging format. Everything a plugin can contain, you can also put standalone in `.claude/` — the difference is *distribution*.

The decision rule: **use standalone `.claude/` for things only this project needs; wrap into a plugin the moment you want to share, version, or reuse across projects.** Start standalone, convert to plugin when the content stabilizes. Plugin skills are namespaced (`/my-plugin:hello`) which prevents conflicts but also means slightly noisier invocation — that's a worthwhile trade for shared work, not for solo experiments.

A minimal plugin manifest:

```json
{
  "name": "internal-platform",
  "description": "Skills, hooks, and MCP servers for our internal platform",
  "version": "1.2.0",
  "author": { "name": "Platform Team" },
  "homepage": "https://git.internal/platform/claude-plugin",
  "repository": "https://git.internal/platform/claude-plugin"
}
```

Layout for a typical team plugin:

```
internal-platform/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── deploy-staging/SKILL.md
│   ├── rotate-credential/SKILL.md
│   └── new-microservice/
│       ├── SKILL.md
│       ├── templates/service-template/
│       └── scripts/scaffold.py
├── agents/
│   ├── security-reviewer.md
│   └── perf-investigator.md
├── hooks/
│   └── hooks.json
├── .mcp.json
├── settings.json
└── README.md
```

The `settings.json` at the plugin root currently supports `agent` (activate a custom agent as the main thread) and `subagentStatusLine`. Setting `"agent": "security-reviewer"` in a plugin's settings makes the plugin reshape how Claude behaves when enabled — useful for compliance-oriented plugins where you want everyone using the same hardened agent profile.

### Plugin marketplaces and team workflows

For a team, the canonical pattern is a private git repo containing a marketplace manifest pointing at one or more plugins. Teammates run `/plugin marketplace add <repo>` once, then `/plugin install internal-platform`, and they get the whole stack — skills, hooks, MCP configs, agents — versioned and updated through git. For solo work or experimentation, the `--plugin-dir ./my-plugin` flag loads a plugin from a local directory; `--plugin-url https://…/plugin.zip` does it from an archive.

### A critical plugin gotcha

**Plugin-provided subagents silently ignore `hooks`, `mcpServers`, and `permissionMode` fields.** For security reasons, those fields are honored only when the agent is defined in `.claude/agents/` or `~/.claude/agents/`. So for your most-trusted agents (the ones doing real work with full autonomy), define them at project or user scope, not in a shared plugin. Plugins are for distribution; project-scope is for trust.

---

## Chapter 11. MCP Servers

MCP is the protocol Claude Code uses to talk to external systems. An MCP server exposes tools (a tool is "create GitHub issue", "query Postgres", "post Slack message") and Claude Code invokes them mid-session. Two transports matter: **stdio** for local subprocess servers (most npm-distributed servers), and **HTTP** for remote servers (GitHub's, Sentry's, Atlassian's). SSE is deprecated in favor of HTTP; for Atlassian's Rovo MCP, the older `/sse` endpoint stops working June 30, 2026, so make sure your URL is `/mcp`.

### Three scopes, and what each is actually for

Configuration scopes determine where the MCP config lives and who sees it:

**Local** (`--scope local`, the default) writes to `~/.claude.json` keyed by project path. Only you, only this project. Right for personal credentials against a project-specific resource — your local Postgres dev DB, a personal Sentry token.

**User** (`--scope user`) writes to `~/.claude.json` globally. Right for servers you want available in every project — GitHub, a search tool, your personal Notion. The criterion is "would I want this in every repo I open?" If yes, user scope.

**Project** (`--scope project`) writes to `.mcp.json` at the project root, committed to git. Right for team-shared infrastructure: a CI MCP server, a project-specific schema inspector. **Never put credentials in `.mcp.json`** — use `${ENV_VAR}` references and have teammates set their own env vars locally.

Precedence on collisions: local > project > user > plugin-provided.

### Tool Search: why MCP is no longer expensive

This is the most important MCP update in 2026 and the reason "too many MCP servers" used to be a real complaint. Tool Search (default on, controllable via `ENABLE_TOOL_SEARCH`) defers MCP tool definitions until Claude needs them. Only tool *names* load at session start; full schemas load on demand when Claude's search tool surfaces a relevant one. This roughly cuts MCP context cost by 95% versus the old behavior. You can connect 15 MCP servers and pay roughly the cost you used to pay for 1.

The practical implication: stop optimizing MCP setups for context budget. Optimize them for *which credentials you trust to expose*. The bottleneck moved from tokens to security.

### Adding servers, with real examples

The `claude mcp add` command is the canonical interface:

```bash
# GitHub via HTTP, available everywhere
claude mcp add github --scope user --transport http https://api.githubcopilot.com/mcp/

# Postgres via stdio, project-specific
claude mcp add postgres --scope project -e DATABASE_URL='${PG_DSN}' \
  -- npx -y @modelcontextprotocol/server-postgres

# Sentry via HTTP, project-scoped
claude mcp add sentry --scope project --transport http https://mcp.sentry.dev/mcp

# Atlassian Rovo MCP for Jira + Confluence + Compass
claude mcp add atlassian --scope project --transport http https://mcp.atlassian.com/v1/mcp

# Filesystem access to additional directories, personal
claude mcp add filesystem --scope user \
  -- npx -y @modelcontextprotocol/server-filesystem ~/code ~/notes
```

For multi-account setups, separate MCP servers per account at user scope avoids the OAuth juggling — `github-personal`, `github-work`, each with its own token, scoped narrowly. Inside a session, `/mcp` shows server status and handles OAuth. `claude mcp list` shows configuration; `claude mcp doctor` diagnoses connection problems.

### Newer MCP capabilities (2.1.x)

A handful of changes since 2.1.121 are worth folding into how you configure servers:

**`alwaysLoad`** (2.1.121): set `"alwaysLoad": true` on a server in its config and *all* of that server's tools skip Tool Search deferral and are always present in context. Normally deferral is what you want — it's the 95%-context-saving default — but for a small, hot server whose tools you call constantly (a custom project server with three tools), `alwaysLoad` removes the discovery round-trip. Reserve it for low-tool-count servers; turning it on for a 60-tool server defeats the purpose of deferral.

**`CLAUDE_PROJECT_DIR`, `CLAUDE_CODE_SESSION_ID`, and `CLAUDECODE=1` in the MCP environment** (2.1.139 and 2.1.154): stdio MCP server subprocesses now receive these, matching what hooks already got. Plugin MCP configs can reference `${CLAUDE_PROJECT_DIR}` in their `command`/`args`, which finally makes project-relative local servers portable. The session id lets a server correlate its own logs with the audit log from Chapter 36.

**`MCP_TOOL_TIMEOUT` now actually raises the per-request ceiling** (fixed 2.1.142): before, remote HTTP/SSE tool calls were capped at 60 seconds regardless of the configured value. If you have a long-running MCP tool (a report build, a slow query) that was mysteriously dying at one minute, set `MCP_TOOL_TIMEOUT` and upgrade past 2.1.142.

**Paginated `tools/list` is fully consumed** (fixed 2.1.144): servers that returned tools across multiple pages previously had everything past page one silently dropped. If a server's tool was "defined but never callable," this was often why.

Two operational notes: `/mcp` Reconnect now picks up `.mcp.json` edits without a full restart (2.1.139), and `workspace` is a reserved server name as of 2.1.128 — a server named `workspace` is skipped with a warning, so rename it.

### Security non-negotiables

MCP servers run code on your machine with your credentials. Three rules that aren't optional:

**Audit before installing.** Read the source of community MCP servers before running them. Prefer official servers (published by the tool vendor) over forks. A server with file system + network access can exfiltrate.

**Principle of least privilege.** Read-only Postgres users. Fine-grained GitHub PATs (not classic), scoped to the repos you need. A schema-inspection server doesn't need write access; a PR-reader doesn't need write to issues.

**Don't commit `.mcp.json` files containing personal credentials.** Use `${ENV_VAR}` references and a shared `.env.example` documenting which vars teammates need to set.

### When MCP isn't the answer

If the capability is just "run a shell command and inject the output," that's a skill with `` !`<command>` `` injection, not an MCP server. MCP is the right call when there's a real external system with structured I/O — issues, rows, errors, threads — that benefits from a typed tool interface. Don't build an MCP server to wrap `kubectl`. Do build (or use) one to wrap a Kubernetes API client that returns structured pod state.

---

## Chapter 12. Combining Skills with MCP

The clearest way to think about this: **MCP gives Claude the ingredients and the knives. A skill is the recipe that tells it how to use them.** Skills shape behavior; MCP servers provide capability against external systems. Combining them is where the highest-leverage workflows live.

Three patterns:

### Pattern A: skill calls MCP tools inline

The skill body lists steps that invoke specific MCP tools. The `allowed-tools` field pre-approves those tools so the skill runs without per-call permission prompts. Tool names follow the `mcp__<server>__<tool>` pattern, so you can also match by server with wildcards.

```markdown
---
name: triage-sentry-error
description: Investigates a Sentry error, finds the responsible code, opens a draft GitHub issue with the analysis. Use when the user mentions an error URL from Sentry or pastes a Sentry issue ID.
allowed-tools: mcp__sentry__get_issue mcp__sentry__list_events mcp__github__create_issue Read Grep Glob
context: fork
agent: Explore
---

Investigate Sentry issue $ARGUMENTS:

1. Call `mcp__sentry__get_issue` with the issue ID to fetch error details,
   stack trace, and affected users.
2. Identify the source file from the top frame of the stack trace.
3. Read that file and the function indicated by the line number.
4. Use Grep to find recent changes to that function.
5. Form a hypothesis: regression, environmental, edge case, unhandled input.
6. Call `mcp__github__create_issue` with:
   - Title: "Triage: <error summary>"
   - Body: stack trace, hypothesis, suggested fix location, link to Sentry
   - Labels: ["bug", "triage", "claude-generated"]
   - Do NOT assign; leave for human review.
7. Return the new issue URL to the main session.
```

That's a complete cross-system workflow in 15 lines of skill body. The `context: fork` runs it in an isolated subagent so its 5,000-token investigation doesn't bloat your main context — only the issue URL comes back.

### Pattern B: skill does deterministic processing, MCP is for data retrieval only

When the skill does heavy lifting (parsing, computing, validation) that doesn't benefit from a model call, the skill bundles a script (Python, Node, whatever) and uses MCP only to fetch inputs. The `${CLAUDE_SKILL_DIR}` placeholder makes script paths work regardless of where the skill is installed:

```markdown
---
name: weekly-velocity-report
description: Generates the weekly engineering velocity report by pulling Linear issue data and computing throughput metrics.
allowed-tools: mcp__linear__list_issues Bash(python3 *)
---

1. Call `mcp__linear__list_issues` with state=Done, completedAt>={last_monday}
2. Pass the JSON to `python3 ${CLAUDE_SKILL_DIR}/scripts/compute_velocity.py`
3. The script writes velocity-report.html
4. Open the HTML in the user's browser
```

### Pattern C: skill orchestrates multi-system workflows

Read one system, transform, write to another. Useful for "create Linear ticket → cut branch → open draft PR → post Slack." This is where the system feels like real automation rather than coding assistance.

### Notes on combining these

**Tool Search and skill `allowed-tools` interact in a useful way.** Tool Search defers MCP tool *schemas* by default, but when a skill lists a specific MCP tool in `allowed-tools`, that tool's schema gets pulled in when the skill loads. So skill-bundled MCP usage doesn't pay the discovery-search cost — Claude already knows the schema.

**Plugin skills can ship their own MCP server configs.** Drop a `.mcp.json` at the plugin root and any skill in that plugin can rely on those servers. This is how Anthropic's own `pdf` and `xlsx` skills work — they bundle the deps they need. For your team, this means a `internal-platform` plugin can ship both the skills *and* the MCP servers they call against the internal platform, so a new teammate gets the whole stack from a single `/plugin install`.

**The MCP `serverInstructions` field becomes a routing hint.** Newer MCP servers can expose a `serverInstructions` string that Claude reads to learn when to search for tools from that server. If you're building an MCP server, write this field as if it were a skill description — that's exactly how Tool Search uses it.

# Part IV — Control Flow

CLAUDE.md and skills *shape* what Claude does. Hooks *enforce* what must happen. This part covers the deterministic control-flow layer — hooks at lifecycle events, anti-loop machinery, the full settings.json automation surface, and the AskUserQuestion tool that lets workflows pause and come back to you.

---

## Chapter 13. Hooks and the Deterministic Layer

CLAUDE.md is advisory; Claude *tries* to follow it. Hooks are deterministic; they *always* fire. Any rule of the form "this must happen at lifecycle event X" belongs in a hook, not in CLAUDE.md.

Hooks fire at specific points during a Claude Code session. When an event fires and a matcher matches, Claude Code passes JSON context to your hook handler. For command hooks, input arrives on stdin; for HTTP hooks, as the POST body. Your handler inspects, takes action, and optionally returns a decision via exit code or stdout JSON.

There are now more than twenty lifecycle events in current Claude Code, falling into three cadences (plus a display-time event, below):

**Once per session:** `SessionStart` (matchers: `startup`, `resume`, `clear`, `compact`), `SessionEnd`. SessionStart is the highest-leverage event because its stdout becomes Claude's context — anything you echo here Claude can see and act on. SessionEnd is for cleanup and final logging.

**Once per turn:** `UserPromptSubmit` (fires when you press enter), `Stop` (Claude finished responding), `StopFailure`. UserPromptSubmit can inject context, validate, or block; Stop is your quality gate.

**Once per tool call:** `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `SubagentStart`, `SubagentStop`. PreToolUse can block destructive actions; PostToolUse can format/lint/test after edits.

**At display time:** `MessageDisplay` (2.1.152+) fires as each assistant message is about to be shown and can transform or hide the text — see the newer-capabilities note below.

### Hook handler types

Four types, and when to use each:

`type: "command"` is what 90% of hooks should be — a shell command or script. Fast, deterministic, no model call cost.

`type: "http"` POSTs the event JSON to a URL. Use it when your team has a central policy service (compliance auto-approver) or to fan out to multiple tools.

`type: "prompt"` runs an LLM evaluation of whether to proceed. Useful when the decision isn't binary — "evaluate whether this Stop is appropriate given the plan" is a good prompt-hook job. Costs a model call per fire.

`type: "agent"` invokes a custom agent for the decision. Heaviest, slowest, but most powerful — Stop-hook-driven report generators are essentially this pattern.

### Exit codes and JSON output

The exit code from your hook command tells Claude Code whether the action should proceed, be blocked, or be ignored.

**Exit 0** means success. Claude Code parses stdout for JSON output fields. For most events, stdout is written to the debug log but not shown in the transcript. The exceptions are UserPromptSubmit, UserPromptExpansion, and SessionStart, where stdout is added as context that Claude can see and act on.

**Exit 2** means a blocking error. Claude Code ignores stdout and any JSON. Instead, stderr text is fed back to Claude as an error message. The effect depends on the event: PreToolUse blocks the tool call, UserPromptSubmit rejects the prompt, Stop forces continuation.

**Any other exit code** is a non-blocking error for most hook events. The transcript shows a hook-error notice with the first line of stderr.

For more control than exit codes alone, exit 0 and print JSON to stdout. PreToolUse hooks can return `permissionDecision` (allow/deny/ask). PostToolUse and Stop hooks can return `decision: "block"` with a reason:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep for performance"
  }
}
```

### Newer hook capabilities (2.1.x)

Several hook features landed in the 2.1.118–2.1.157 window that change what's worth wiring:

A new **`MessageDisplay`** hook event (2.1.152) fires as assistant message text is about to be shown and lets a hook transform or hide it. This is the clean way to do output redaction (mask secrets, strip internal URLs) or house-style rewriting without touching the model — previously you'd have hacked it through PostToolUse. It's display-only; it doesn't change what the model actually said in context.

**Exec-form `args`** (2.1.139): a command hook can now specify `"args": ["script.sh", "$CLAUDE_PROJECT_DIR/x"]` instead of a single `command` string. The command is spawned directly without a shell, so path placeholders with spaces never need quoting — a real reliability win on Windows paths and `$CLAUDE_PROJECT_DIR` interpolation. Prefer this form for any hook that passes file paths.

**`continueOnBlock` for PostToolUse** (2.1.139): set it `true` and a PostToolUse hook's rejection reason is fed back to Claude and the turn *continues* instead of erroring out. This is what you want for "lint failed, here's what to fix" gates that should nudge rather than halt.

**`terminalSequence` in hook output** (2.1.141): hooks can emit desktop notifications, window-title changes, and bells via a returned `terminalSequence` field even when they have no controlling terminal — which is exactly the background/headless case. This is the modern replacement for shelling out to `osascript`/`notify-send` inside a notify hook, and it works from background sessions where the old approach silently did nothing. (Relatedly, as of 2.1.139 hooks run *without* direct terminal access so a chatty hook can't corrupt an interactive prompt — write to stderr, not the tty.)

**`effort.level` and `$CLAUDE_EFFORT`** (2.1.133): hooks now receive the active effort level in their JSON input (`effort.level`) and as the `$CLAUDE_EFFORT` environment variable, and Bash tool commands can read `$CLAUDE_EFFORT` too. Use it to scale gate strictness — run the full test suite at `high`/`xhigh`, a smoke subset at lower effort.

**`background_tasks` and `session_crons` in Stop/SubagentStop input** (2.1.145): your Stop gate can now see whether background work or scheduled tasks are still in flight and decline to "finish" prematurely. Worth checking in a report-gate so the end-of-session report isn't generated while a backgrounded build is still running.

**SessionStart can now mutate the session** (2.1.152): a SessionStart hook may return `reloadSkills: true` to re-scan skill directories — so a hook that *installs* a skill makes it available in the same session — and may set the session title via `hookSpecificOutput.sessionTitle`. There's also a standalone `/reload-skills` command for the interactive case.

**`type: "mcp_tool"`** (2.1.118) lets a hook invoke an MCP tool directly, and **`hookSpecificOutput.updatedToolOutput`** (generalized to all tools in 2.1.121) lets a PostToolUse hook rewrite tool output before Claude sees it. PostToolUse input also carries `duration_ms` (2.1.119) for timing-based gates.

One configuration caveat surfaced in 2.1.142: a `prompt`- or `agent`-type hook attached to `SessionStart`, `Setup`, or `SubagentStart` is now rejected with a clear "use a command-type hook instead" error. Those early-lifecycle events run before there's a conversation for a model-based hook to evaluate, so keep them `command`-type.

### Concrete examples

The simplest auto-formatter:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit|MultiEdit",
      "hooks": [{
        "type": "command",
        "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\"",
        "async": true
      }]
    }]
  }
}
```

A PreToolUse block-list for dangerous Bash:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|DROP TABLE' && exit 2 || exit 0"
      }]
    }]
  }
}
```

A SessionStart hook that injects git context (stdout becomes Claude's context):

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|resume|compact",
      "hooks": [{
        "type": "command",
        "command": "echo \"Branch: $(git branch --show-current). Uncommitted: $(git status --porcelain | wc -l) files.\""
      }]
    }]
  }
}
```

---

## Chapter 14. Preventing Loops

The "Claude won't stop / keeps running in a loop" failure mode is real, and the system has explicit machinery to prevent it. Three things matter.

**The `stop_hook_active` field.** When your Stop hook fires and exits with code 2 (forcing Claude to continue), Claude continues — and when it tries to stop again, your Stop hook fires *again*. Without a guard, this is an infinite loop that burns tokens until the session times out. The guard is the `stop_hook_active` field in the hook's stdin JSON: when it's `true`, a previous Stop hook is already forcing continuation, and your hook must exit 0 immediately to let Claude actually stop. **Every Stop hook needs this check as its first action:**

```bash
#!/usr/bin/env bash
# .claude/hooks/test_gate.sh
INPUT=$(cat)

# CRITICAL: if a previous Stop hook is already forcing continuation, exit 0.
# This is the single most important line in any Stop hook.
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi

# Otherwise, run the gate
if ! pnpm test --silent; then
  echo "Tests failing — fix before declaring done." >&2
  exit 2
fi
exit 0
```

Without this guard, your Stop hook becomes a token-furnace. With it, the gate fires *once* per Stop attempt: Claude tries to stop, the hook blocks once with feedback, Claude reads the feedback and works on the fix, Claude tries to stop again, and this time `stop_hook_active` is true so the hook exits 0 and Claude actually stops.

The same pattern applies to SubagentStop. **This is the single most important line in any Stop or SubagentStop hook.**

**Default non-blocking semantics for Stop/SubagentStop.** Newer Claude Code versions ship Stop and SubagentStop hooks with `blocking: false` by default in some hook framework wrappers, precisely because the infinite-loop failure mode is so common. If you need blocking behavior (a TDD gate), you explicitly enable it — but with the `stop_hook_active` guard above.

**The runtime block cap (2.1.143+).** As of 2.1.143 there is now a backstop in the runtime itself: a Stop hook that keeps blocking ends the turn with a warning after **8 consecutive blocks**, rather than looping until the session times out. You can tune the ceiling with `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`. Treat this as defense-in-depth, not a substitute for the `stop_hook_active` guard — a hook that blocks eight times in a row is still a bug, and the cap just stops it from burning your whole budget. If you ever see "turn ended after repeated Stop-hook blocks," that's the cap firing, and the fix is almost always a missing or broken `stop_hook_active` check.

**A cleaner feedback path than exit 2 (2.1.163+).** Stop and SubagentStop hooks can now return `hookSpecificOutput.additionalContext` to hand Claude feedback and keep the turn going *without* the interaction being logged as a hook error. Where exit 2 forces continuation by raising a blocking error (noisy in the transcript and in `--debug`), `additionalContext` reads as guidance: "tests are red, here's the tail — keep working." For a TDD gate you now have a choice — exit 2 for a hard stop-the-line block, or `additionalContext` for a softer nudge. The `stop_hook_active` guard still matters either way, because both paths re-enter the hook on the next stop attempt.

**AskUserQuestion no longer auto-continues by default (2.1.200), and cannot be called from subagents.** Older versions returned a default-choice fallback after a 60-second timeout; as of 2.1.200 the dialog waits rather than auto-continuing, and you opt into an idle timeout via `/config` if you want the old unattended behavior back. For overnight autonomous runs this is a meaningful change: if a skill calls AskUserQuestion and nobody answers, the turn now *waits* unless you've explicitly configured an idle timeout — so either configure one or keep AskUserQuestion out of the unattended path and let hooks + auto mode make the call instead.

### The hash-sentinel idempotency pattern

For Stop hooks that should fire *exactly once* per state change (think: "fire the report-generator agent when prompt_plan.md is fully complete"), use a hash sentinel. Hash the relevant file, compare to the last-recorded hash, exit 0 if they match (already reported), otherwise fire and update the hash:

```bash
#!/usr/bin/env bash
# .claude/hooks/report_gate.sh
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi

PLAN="$CLAUDE_PROJECT_DIR/prompt_plan.md"
SENTINEL="$CLAUDE_PROJECT_DIR/.claude/.last-report-hash"

if grep -q '^- \[ \]' "$PLAN"; then exit 0; fi  # work still open

CURRENT=$(sha256sum "$PLAN" | cut -d' ' -f1)
LAST=$(cat "$SENTINEL" 2>/dev/null || echo "")
if [ "$CURRENT" = "$LAST" ]; then exit 0; fi  # already reported

echo "$CURRENT" > "$SENTINEL"  # record before signaling
echo "Plan complete. Invoke the report-generator agent." >&2
exit 2  # signal Claude
```

This makes the hook *idempotent across sessions*, not just within one. It generalizes — any "fire X exactly once when condition Y is reached" workflow uses this shape.

---

## Chapter 15. Automating with settings.json

`settings.json` is the central wiring file for hooks, environment, permissions, and a handful of behavioral settings. It exists at four layers with merge precedence: managed → user → project → local. For automation specifically, the workhorse is the `hooks` object.

### A complete `.claude/settings.json` for a high-automation setup

Here's a single settings file that covers auto-format, dangerous-command blocking, SessionStart context injection, a test gate, async session logging, and a report-generator pattern:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "python \"$CLAUDE_PROJECT_DIR/.claude/hooks/session_context.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python \"$CLAUDE_PROJECT_DIR/.claude/hooks/prompt_validator.py\"",
            "timeout": 5
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/guard_bash.sh\"",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/guard_protected_files.sh\"",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/format_changed.sh\"",
            "timeout": 30,
            "async": true
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/test_gate.sh\"",
            "timeout": 120
          },
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/report_gate.sh\"",
            "timeout": 60
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date -Iseconds) session ended\" >> ~/.claude/work-log.txt",
            "async": true
          }
        ]
      }
    ]
  },
  "autoMemoryEnabled": true,
  "skillListingBudgetFraction": 0.02
}
```

A few important things in there. First, the `SessionStart` hook uses three matchers (`startup|resume|compact`) — that's the trick to also re-inject context after auto-compact fires, which is when most "Claude forgot the plan" complaints happen. Second, the `PostToolUse` format hook runs `async: true`, so it doesn't block Claude's next turn — the file gets formatted in the background. Third, Stop has *two* hooks defined and they run in parallel; one is the test gate, one is the report gate, each with its own `stop_hook_active` guard internally. Fourth, `skillListingBudgetFraction: 0.02` doubles the budget for skill descriptions — useful when you have a lot of skills and want fuller descriptions in context.

### The settings layers you actually use

`~/.claude/settings.json` for personal global rules (auto-format with your preferred tool, your global dangerous-command list).

`.claude/settings.json` at project root, committed to git, for team-shared rules (the test gate everyone needs, the import policy for that repo).

`.claude/settings.local.json` at project root, gitignored, for your personal project overrides (your local DB URL, your `claudeMdExcludes` for monorepo teams you don't work with).

Managed settings at the OS-specific managed path for org-wide rules that individuals can't override.

### One important gotcha

Settings changes don't hot-apply mid-session. Claude Code snapshots hooks at session start and prompts you to review them in `/hooks` if they change. This is a security feature (a compromised repo can't inject malicious hooks mid-session) — restart Claude Code to pick up changes.

---

## Chapter 16. AskUserQuestion and Workflows That Pause

`AskUserQuestion` is a built-in Claude Code tool (since v2.0.21) that pauses execution and presents structured multiple-choice options. It's what makes spec-based development work: instead of guessing and writing the wrong thing, Claude asks. It returns within 60 seconds (timeout) and Claude can ask roughly 4–6 questions per session before it stops using the tool. You don't define AskUserQuestion yourself — it's always available — but you can *steer Claude toward using it* in your CLAUDE.md and skill bodies.

A CLAUDE.md directive that activates this behavior:

```markdown
## Information gathering

Before implementing anything from `specs/`, if any of the following are
unclear, use `AskUserQuestion` to surface the decision rather than guessing:
- Which of multiple existing patterns to follow
- Error handling strategy (fail fast vs. retry vs. degrade)
- Where to put new files when conventions are ambiguous
- Whether a breaking change is intentional

Generate concrete options (with a recommended one marked) rather than
open-ended questions. If the user explicitly says "use your judgment"
or "just pick one," proceed without asking.
```

Inside a skill, the same pattern looks like this:

```markdown
---
name: new-microservice
description: Scaffolds a new microservice in this repo. Asks about runtime, framework, and storage before generating files. Use when the user asks to create a new service or add a new bounded context.
---

Before scaffolding, use `AskUserQuestion` to gather:

1. Runtime — Node 22 (recommended), Bun, or Deno
2. Framework — Hono (recommended), Fastify, or Express
3. Storage — Postgres + Drizzle (recommended), Postgres + Prisma, or none yet
4. Add to monorepo workspaces — Yes (recommended) or No

After answers come back, proceed to scaffold under `services/<name>/`...
```

This is the elicitation pattern. It pairs especially well with **plan mode** — start a session in plan mode (`/plan` or shift-tab toggle), let Claude explore + ask its clarifying questions, accept the plan, then execute. Plan mode is built around AskUserQuestion: it's the model gathering requirements before any code gets written.

### The always-execute-with-escape-hatch composition

Three primitives together cover every "workflow that should run deterministically but pause when ambiguous" case.

**Hooks** are for things that must always run regardless of model judgment — quality gates, format-on-edit, dangerous-command blocking. They fire whether Claude wants them to or not.

**AskUserQuestion** is for moments where the model needs information it cannot infer. It pauses execution and gives you structured options.

**Notification hooks** wake you up when AskUserQuestion fires or a permission prompt appears, so you can be away from the terminal during long runs.

When Claude calls AskUserQuestion and you're not at the keyboard, you want to know. The `Notification` hook fires with matchers `permission_prompt`, `idle_prompt`, `auth_success`, and on some versions `user_question`/`elicitation_dialog`. Wire it to your OS notification system:

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell -Command \"New-BurntToastNotification -Text 'Claude Code', 'Input needed'\""
          }
        ]
      }
    ]
  }
}
```

(On macOS, swap in `terminal-notifier` or `osascript`; on Linux, `notify-send`.) Combined with AskUserQuestion in your skills, this is what makes long autonomous runs viable: Claude works for an hour, gets stuck on a real decision, asks the question, your taskbar lights up, you tap an option, it keeps going.

### The complete pattern

The full shape of a workflow that fires deterministically and yields control when needed:

1. **SessionStart hook** injects the current state (branch, plan step, recent decisions).
2. **CLAUDE.md** contains the routing rules pointing at skills, plus the "use AskUserQuestion when ambiguous" directive.
3. **The skill itself** does the actual work and uses AskUserQuestion at known decision points.
4. **PreToolUse hooks** block destructive actions unconditionally — Claude can't override these by deciding it's fine.
5. **PostToolUse hooks** auto-format and lint changed files asynchronously.
6. **Stop hooks** gate completion: tests must pass, spec must be marked done in `prompt_plan.md`. Failures exit 2 with feedback; the `stop_hook_active` guard prevents loops.
7. **Notification hooks** ping your OS when input is needed.
8. **The report-generator agent** fires once via a Stop hook with hash-sentinel idempotency when the plan is genuinely complete.

That's a workflow that fires the same way every time, completes autonomously when it can, and asks you precisely when it can't. Every step is either a hook (deterministic) or AskUserQuestion (interactive); CLAUDE.md and skills are the *content* of the work, but the *control flow* is enforced by hooks. Claude can't drift past a hook the way it can drift past a CLAUDE.md instruction.

# Part V — Agents

Agents are the layer that closes the loop. The previous parts framed memory, skills, plugins, MCP, hooks, and AskUserQuestion as the *content* and *control flow* of Claude's work. **Agents are who actually does the work.** They are scoped specialists — each with its own context window, system prompt, tool allowlist, model choice, permission mode, and (optionally) git worktree — that the main session delegates to. Once you understand how to constrain them properly, you can let an agent run an entire bug fix, CVE remediation, doc update, or deployment end-to-end and only come back to you when something genuinely needs a human.

---

## Chapter 17. Where Agents Fit in the Stack

Five primitives, five different jobs:

**CLAUDE.md** is what Claude *always knows*. Loaded every session, every compaction.

**Skills** are *procedures Claude can run*. Loaded on invocation.

**Hooks** are *deterministic gates* — code that fires at lifecycle events whether Claude wants it to or not.

**MCP servers** are *external capabilities* — tools against real systems.

**Agents are *who runs the work*.** Skills shape what one Claude instance does in one turn. Agents are entire delegated sessions: own prompt, own tools, own context, own model. The main session keeps the conversation; agents do the heavy or risky work and return only a summary.

A useful test for whether you want an agent or a skill: *would this work pollute the main context if it stayed there?* A skill running inline ("apply this naming convention") doesn't pollute. An investigation that reads 30 files, runs the test suite, and traces a failure to its root cause absolutely does — that's an agent. The agent does the noisy work in isolation and returns a five-line summary.

The right rule: **start as a skill, promote to an agent when the work becomes noisy enough that you don't want it in your main context, or risky enough that you want tool restrictions and worktree isolation around it.**

---

## Chapter 18. Subagents, Forks, and Agent Teams

Three forms of delegation exist and they're worth keeping straight.

A **subagent** is a scoped specialist invoked through the Task tool. It has its own fully-defined system prompt (the markdown body of `.claude/agents/<name>.md`), its own tool list, its own model. It costs context because the system prompt loads fresh.

A **fork** is cheaper: it reuses the parent's system prompt and tool definitions, so the first request hits the parent's prompt cache. Use forks when you want the same setup as the parent but isolated execution — speculative refactors, "try this and report back" tasks. Enable with `CLAUDE_CODE_FORK_SUBAGENT=1` or pass `isolation: "worktree"` when spawning. A fork cannot spawn further forks.

An **agent team** (the experimental Agent Teams feature, behind `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) coordinates multiple long-lived sessions with a team lead. Teammates can message each other directly through a mailbox system, not just report back to the lead. Use it for genuinely parallel work across separate workstreams (refactor the API layer while migrating the database while updating tests) where the workers need to coordinate. **Two changes to know (2.1.178):** the `TeamCreate` and `TeamDelete` tools were *removed* — with the flag set, every session now has one implicit team, so you spawn teammates directly with the Agent tool's `name` parameter and skip the setup step (the old `team_name` parameter is still accepted but ignored). Separately, the broader capability grew: as of 2.1.172 subagents can spawn their *own* subagents up to 5 levels deep, so a lead → teammate → helper chain is now a first-class pattern rather than something you fake.

For 90% of work, subagents are enough. Don't reach for agent teams until you've actually hit "this work needs four agents talking to each other" — that's rarer than it sounds.

### Orchestration features added in 2.1.x: workflows, the agent view, and /goal

Three capabilities landed in the May 2026 releases that change the ceiling on autonomous work and deserve to sit alongside subagents/forks/teams.

**Dynamic workflows (`/workflows`, 2.1.154)** are the headline. You ask Claude in plain language to create a workflow, and it plans and orchestrates work across *tens to hundreds* of background agents — far beyond what you'd hand-wire with subagents or an agent team. Run `/workflows` to see your runs and their live agent counts. This is the right tool when a task fans out massively and uniformly: "migrate every call site of this deprecated API across the monorepo," "triage all 400 open Dependabot alerts." It supersedes hand-rolled fan-out for large, parallelizable jobs; you still use named subagents for the small, role-differentiated cast in your main pipeline. **Note the keyword change:** the explicit trigger keyword was renamed from `workflow` to **`ultracode`** in 2.1.160 (highlighted violet in the prompt) — the bare word "workflow" no longer triggers a run, though asking for one in your own words still works. A "Dynamic workflow size" setting in `/config` (2.1.202) advises how large Claude makes these runs.

As of 2.1.198, **subagents run in the background by default** — Claude keeps working on the main thread while a delegated agent runs, and you're notified (via the `Notification` hook, matchers `agent_needs_input` / `agent_completed`) when it needs input or finishes. Background agents launched from `claude agents` that finish code work in a worktree now commit, push, and open a draft PR on their own instead of stopping to ask.

**The agent view (`claude agents`, Research Preview, 2.1.139)** is a single dashboard of every Claude Code session — running, blocked on you, or done — across your machine. For someone running several background pipelines at once this replaces the "which terminal was that in" problem. It grew real flags quickly: `--add-dir`, `--settings`, `--mcp-config`, `--plugin-dir`, `--permission-mode`, `--model`, `--effort`, and `--dangerously-skip-permissions` (2.1.142–2.1.143) all set defaults for sessions dispatched from the view, `--cwd <path>` scopes the list (2.1.141), and `claude agents --json` (2.1.145) emits the session list for status bars and scripting. As of 2.1.157 the `agent` field in `settings.json` is honored for dispatched sessions, with `--agent <name>` to override. You can also fire a one-off background shell with `! <command>` inside the view, or `claude --bg --exec '<command>'` (2.1.154).

**`/goal` (2.1.139)** sets a completion condition and lets Claude keep working across turns until it's met, showing live elapsed/turns/tokens in an overlay. It works in interactive, `-p`, and Remote Control. This is the lighter-weight cousin of the Stop-hook completion gate from Chapter 14: where the Stop hook enforces *your* deterministic condition (tests green, plan complete), `/goal` lets the model self-evaluate against a natural-language target. Use the Stop hook for hard guarantees; use `/goal` for "keep going until the flaky test is actually fixed" where the condition is judgment, not a check. One gotcha (fixed 2.1.140): if `disableAllHooks` or `allowManagedHooksOnly` is set, older builds let `/goal` hang silently — upgrade past 2.1.140 for the clear message.

### Subagent frontmatter, in full

The full frontmatter surface for `.claude/agents/<name>.md`:

```yaml
name: <unique-name>
description: <when to invoke; be specific and pushy>
model: <sonnet | opus | haiku | inherit | full-model-id>
effort: <low | medium | high | max>
tools: <space-separated list, with Bash(pattern) for specifics>
disallowedTools: <space-separated list>
permissionMode: <default | acceptEdits | plan | bypassPermissions>
maxTurns: <integer cap>
skills: <skills to preload into context>
mcpServers: <MCP servers to expose to this agent only>
hooks: <agent-scoped hooks, same shape as settings.json hooks>
memory: <user | project | local | none>
background: <true | false>
isolation: <worktree | none>
initialPrompt: <auto-submitted first turn>
color: <UI tag color>
```

Only `name` and `description` are required. The rest defaults to inheriting from the main session (which is the wrong default for autonomous agents — narrow each field explicitly).

A note: subagents **don't automatically inherit project CLAUDE.md**. They get their own system prompt (the agent file body) plus basic environment details. If your agent needs to know the project's build commands or conventions, write them into the agent body — don't assume CLAUDE.md is in scope.

---

## Chapter 19. The Trust Model for Autonomous Agents

The fear behind "can I trust an agent to do X end-to-end" is reasonable, because giving a model write access plus shell access plus network access without constraints is the recipe for the bad story you don't want to be the protagonist of. The good news: Claude Code's agent configuration surface is designed for exactly this — narrow tools, narrow permissions, narrow output contracts. Five mechanisms together make autonomous agents safe.

**Worktree isolation is the blast-radius control.** When you set `isolation: worktree` on an agent, it runs in a fresh temporary git worktree — a separate checkout of the same branch in a separate directory. The agent can write whatever it wants there; your main working tree is untouched. When it finishes, it returns the worktree path and branch name; you review and merge (or discard). This is the foundation: if everything else fails, the worktree contains the damage. A CVE-fixing agent should *always* have worktree isolation. So should any refactor agent.

One behavior change to know if you isolate agents (2.1.133): the `worktree.baseRef` setting (`fresh` | `head`) controls whether `--worktree`, `EnterWorktree`, and agent-isolation worktrees branch from `origin/<default>` or your local `HEAD`. The default is `fresh`, which branches from `origin/<default>` — meaning **unpushed local commits do not appear in the agent's worktree.** For the common bug-investigator pattern where you want the agent to see your in-progress work, set `worktree.baseRef: "head"` explicitly. (There's also `worktree.bgIsolation: "none"` in 2.1.143 for repos where worktrees are impractical and you want background sessions to edit the working copy directly — use it sparingly, since it gives up the blast-radius guarantee.) As of 2.1.154 a bug where background-session subagents could bypass the worktree-isolation guard and write to the shared checkout was fixed; if you run isolated agents from background sessions, upgrade past 2.1.154.

**The `tools` / `disallowedTools` fields shape capability.** An agent that reviews code shouldn't have `Edit` or `Write`. An agent that runs the test suite shouldn't have `WebFetch`. A documentation agent shouldn't have `Bash` except for narrow git operations. The narrower the toolset, the smaller the failure space. The convention is `tools: Read Grep Glob Bash(git diff *) Bash(git log *)` — explicit list, parenthesized command patterns where you need to be specific. The opposite convention, `disallowedTools`, is for "use most things but never X" cases.

**The `permissionMode` field controls when humans get pulled in.** Modes are `default` (Claude asks for sensitive operations), `acceptEdits` (file edits auto-approved, commands still ask), `plan` (read-only, surfaces a plan for approval before any action), and `bypassPermissions` (auto-approve everything — only safe inside fully-isolated worktrees with tight tool allowlists). Plan mode is the right default for any agent doing design or analysis. AcceptEdits is the right mode for agents doing well-defined implementation work in a worktree. Bypass mode is the right mode for an agent running in a worktree with `tools: Read Edit Write Bash(npm test*)` — there's nothing destructive it can do.

Naming note (2.1.200): the mode formerly surfaced as "default" is now labeled **"Manual"** across the CLI, `--help`, VS Code, and JetBrains, and a grey ⏸ badge appears in the footer when you're in it (2.1.203). Both `--permission-mode manual` / `"defaultMode": "manual"` and the old `default` spelling are accepted, so existing agent frontmatter and settings keep working — but when you read "Manual" in the UI, that's this mode.

There is now a fifth posture worth knowing: **auto mode.** Rather than the blanket allow/deny of `bypassPermissions`, auto mode runs each proposed action through a safety classifier that allows routine work and stops on genuinely risky operations (notably data exfiltration — the classifier's detection of bulk repository-content transfers was hardened in 2.1.154). It appears in the Shift+Tab permission cycle (2.1.143), no longer requires an opt-in flag or consent prompt as of 2.1.152, and — most relevant for a Bedrock/Vertex/Foundry shop — became available on those providers for Opus 4.7 and 4.8 in 2.1.158 by setting `CLAUDE_CODE_ENABLE_AUTO_MODE=1`. You can tune it with `autoMode.allow`, `autoMode.soft_deny`, `autoMode.hard_deny` (unconditional blocks, 2.1.136), and `autoMode.environment` rules; include `"$defaults"` in those lists to extend the built-in ruleset instead of replacing it (2.1.118). Think of auto mode as the middle ground between `acceptEdits` and `bypassPermissions`: more autonomous than the former, more defensible than the latter, because a classifier — not a blanket rule — is making the call.

**`maxTurns` caps cost.** An agent with `maxTurns: 30` literally cannot run forever. It will hit the cap, summarize, and return. This is the simplest insurance against pathological loops and prompt-injection-induced wandering.

**Skill-level and agent-level hooks enforce policy at the agent boundary.** An agent definition can include its own `hooks` block — PreToolUse to block dangerous Bash, PostToolUse to run a linter, SubagentStop with the test-gate pattern. These run *inside* the agent's session, so the agent cannot bypass them even if its prompt is jailbroken.

### The full-autonomy posture

The combined posture for full autonomy looks like this:

- The agent runs **in a worktree** (blast radius bounded)
- With a **narrow tool allowlist** (capability bounded)
- In **acceptEdits or bypassPermissions mode** (human-in-loop only when the agent explicitly asks via AskUserQuestion)
- With **maxTurns set** (cost bounded)
- With **its own SubagentStop hook** (quality gated)

Inside that envelope, you can sleep through the run.

### One more guardrail

**Plugin subagents don't support `hooks`, `mcpServers`, or `permissionMode` frontmatter** — for security reasons, those fields are honored only when the agent is defined in `.claude/agents/` or `~/.claude/agents/`. So for your most-trusted agents (the ones doing real work), define them at project or user scope, not in a shared plugin. Plugins are for distribution; project-scope is for trust.

---

## Chapter 20. When to Use Agents — and When Not To

Five clear use-yes cases:

**Bounded investigation work** that produces verbose intermediate output. Worktree-isolated, read-only, returns a structured report. A `bug-investigator` agent that reads thirty files and returns three ranked hypotheses is exactly this.

**Risky modification work** where you want filesystem isolation. Worktree-isolated, acceptEdits mode, narrow Bash allowlist, SubagentStop test gate. CVE remediation, large refactors, automated cleanups — anything where "discard and try again" is a reasonable fallback.

**Specialized review work** where you want to enforce a different mental model. Read-only, possibly different model (Opus for security-critical work even if main session is on Sonnet). A `security-reviewer` agent with read-only tools, Opus, and a detailed prompt about injection patterns is the pattern.

**Cross-system orchestration** that touches multiple MCP servers. Sonnet is fine; the work is structured, not creative. A `doc-publisher` agent that reads a git diff, updates Confluence, and adds a Jira comment is in this bucket.

**Long-running async work** that shouldn't block the main session. `background: true` agents for overnight regression sweeps, dependency audit passes, doc-rebuild runs.

Five clear use-no cases:

**Tightly coupled multi-step work** where each step depends on context from the previous. The handoff cost (summarize, re-establish, re-read) outweighs the isolation benefit. Stay in the main session and use skills.

**Quick fact-finding** that fits in one read. An agent costs a fresh context window setup; for a single grep, just do it inline.

**Anything requiring shared mutable state** between workers. Agent Teams partly addresses this with the mailbox system, but it's still experimental — for now, sequence subagents rather than parallelize them when they need to coordinate.

**Trivial edits** to a single known file. Skills cover this better; the agent overhead isn't justified.

**Anything you wouldn't trust to run unattended.** If you wouldn't accept the worst-case output, don't run it as an autonomous agent — keep it in the main session where you can interrupt.

The implicit rule: **agents are for noisy, bounded, summarizable work.** Everything else stays inline.

# Part VI — Building the Pipeline

This part is the worked example. A full software development lifecycle, packaged as agents, with each agent's role, tool allowlist, model, and where it plugs in. The pipeline covers requirements → design → implement → security review → test → docs → deploy, plus parallel paths for bug fixing and CVE remediation, plus Atlassian (Jira + Confluence) plumbing.

If you've read parts I through V, this is where everything ties together.

---

## Chapter 21. The Complete Cast of Agents

Ten agents. Each one tight. The main session is the conductor — it routes work to agents based on the CLAUDE.md rules from Part II and the user's intent.

| Agent | Role | Model | Worktree | Tools (high level) |
|---|---|---|---|---|
| `spec-architect` | Turns a Jira ticket or rough idea into a numbered spec | Opus, plan mode | No | Read, Grep, Atlassian MCP read, AskUserQuestion |
| `spec-checker` | Validates implementation against a spec | Sonnet | No | Read, Grep, Glob |
| `implementer` | Implements next step in `prompt_plan.md` | Opus, acceptEdits | Yes | Read, Edit, Write, Bash(test/lint), Grep, Glob |
| `bug-investigator` | Diagnoses bugs in isolation | Opus | Yes | Read, Grep, Glob, Bash(git/test, read-only) |
| `cve-remediator` | Patches CVEs and validates the fix | Opus | Yes | Read, Edit, Write, Bash(audit/test), WebFetch |
| `security-reviewer` | Reviews diffs for security issues | Opus | No (read-only) | Read, Grep, Glob, Bash(git diff *) |
| `test-runner` | Runs targeted test suites, reports failures | Haiku | No | Read, Bash(test commands only) |
| `doc-publisher` | Updates Confluence and Jira from the diff | Sonnet | No | Read, Grep, Bash(git log/diff *), Atlassian MCP write |
| `deployer` | Stages/promotes, with explicit human gate | Sonnet | No | Read, Bash(deploy scripts), AskUserQuestion |
| `report-generator` | End-of-session summary | Sonnet | No | Read, Bash(git log *), Atlassian MCP write |

A few design choices worth explaining.

**Model selection is deliberate.** Opus for work that needs reasoning (design, implementation, investigation, security review). Sonnet for structured cross-system orchestration (docs, deploys, reports). Haiku for I/O-bound work (running tests, parsing output). This is roughly a 10× cost reduction versus running everything on Opus, with no quality loss on the cheap-model jobs.

**Worktree isolation tracks risk.** Anything that *writes code* gets a worktree — `implementer`, `cve-remediator`, `bug-investigator` (because it might be tempted to fix something it sees). Anything read-only doesn't need one. `deployer` doesn't write code but it does *deploy*; its safety comes from the AskUserQuestion gate, not from worktree isolation.

**Permission modes track autonomy.** `plan` for design and review (surface a plan, no action). `acceptEdits` for implementation (file edits free, commands still ask). `default` for cross-system writes (every external call confirmed). No agent gets `bypassPermissions` in this cast — even worktree-isolated agents still have permission prompts for the genuinely irreversible operations.

### The agent file format

For concreteness, here's what `implementer` looks like as a real file. The other agents follow the same pattern with different bodies and allowlists.

```markdown
---
name: implementer
description: Implements the next open step in prompt_plan.md. Use when the user says "implement", "next step", "continue", "build", or asks to action a numbered spec item. Runs in an isolated worktree; returns the worktree path and a summary of changes.
model: claude-opus-4-7
effort: high
isolation: worktree
permissionMode: acceptEdits
maxTurns: 40
tools:
  - Read
  - Edit
  - Write
  - MultiEdit
  - Grep
  - Glob
  - Bash(pnpm test*)
  - Bash(pnpm lint*)
  - Bash(pnpm tsc*)
  - Bash(git status)
  - Bash(git diff*)
  - Bash(git add*)
  - Bash(git commit -m *)
disallowedTools:
  - WebFetch
  - Bash(git push*)
  - Bash(rm -rf *)
hooks:
  PostToolUse:
    - matcher: "Edit|MultiEdit|Write"
      hooks:
        - type: command
          command: "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/format_changed.sh\""
          async: true
  SubagentStop:
    - hooks:
        - type: command
          command: "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/test_gate.sh\""
          timeout: 120
---

You are the implementer agent. You work in an isolated git worktree on a single
plan item from `prompt_plan.md`. The user has already approved the design.

## Procedure

1. Read `prompt_plan.md` and identify the next open `- [ ]` item.
2. Read the spec file it references in `specs/`.
3. Implement the item. Make targeted edits, not broad rewrites.
4. Write or update the test that covers the new behavior.
5. Run `pnpm test`, `pnpm lint`, and `pnpm tsc --noEmit`. Fix anything that fails.
6. Mark the plan item as `- [x]` in `prompt_plan.md`.
7. Stage and commit with a Conventional Commits message:
   `feat(scope): brief description (refs SPEC-NNNN)`
8. Do not push. Return: the worktree path, the commit SHA, the test status, and
   any decisions you made that weren't fully specified.

## Constraints

- Use AskUserQuestion if you encounter a real ambiguity not covered by the spec.
- Never edit `prompt_plan.md` to add items; only mark existing items done.
- Never bump dependencies unless the spec explicitly requires it.
- Never modify files outside the paths the spec touches.

## Output format

Return exactly this structure:

```
WORKTREE: <path>
BRANCH: <branch>
COMMIT: <sha>
TESTS: <pass|fail with summary>
SPEC: <NNNN status: complete|partial|blocked>
DECISIONS:
- <one-line description of each non-trivial decision>
NOTES:
- <anything the main session should know>
```
```

A few things in that file are worth pointing out. The `disallowedTools` explicitly forbids `git push` and `rm -rf` even though they might match a broader `Bash` allow — disallows win when they conflict with allows. The SubagentStop hook is the test gate; it fires when this agent tries to finish and forces it back to work if tests fail. The output format is rigid because the main session has to parse it — agents return text, and rigid format makes parsing reliable.

---

## Chapter 22. The Main Flow: Jira to Deploy

Here's how a typical incoming bug works end-to-end with this setup.

**Step 1 — Ingest.** You paste a Jira link or say "look at PLATFORM-1247." The main session calls the Atlassian MCP (`mcp__atlassian__getJiraIssue`) to pull the ticket. If the ticket is genuinely ambiguous, AskUserQuestion fires with options ("is this a bug fix, a refactor request, or a new feature?"). For a bug, it routes to `bug-investigator` first.

**Step 2 — Investigate.** `bug-investigator` runs in a worktree with read-only tools. It reproduces the failure, reads the relevant source, runs `git blame` to find recent changes, and forms a hypothesis with ranked evidence. The output is a structured report: failure site, three hypotheses, recommended fix location. **Nothing has been modified yet.** The report comes back to the main session as a single message.

**Step 3 — Implement.** Main session reviews the investigator's report, optionally asks you via AskUserQuestion which hypothesis to act on, then routes to `implementer`. `implementer` runs in a *fresh* worktree (the investigator's worktree was read-only and can be discarded), with `acceptEdits` mode. It writes the fix, adds a test that reproduces the original failure and now passes, and updates `prompt_plan.md` to mark the spec item done. Its SubagentStop hook runs `pnpm test` and exit-2's if anything fails — the agent literally cannot finish with broken tests.

**Step 4 — Review.** With the fix applied in the worktree, main session routes to `security-reviewer`. This agent has only Read/Grep/Glob plus `git diff` — it cannot modify anything. It scans the diff for injection patterns, hardcoded secrets, missing error handling, missing tests for new code paths, and broken invariants. Output is a structured findings list: critical, important, nitpick.

**Step 5 — Tests.** `test-runner` (Haiku, cheap) runs the full suite and reports. If it fails, the loop goes back to `implementer` with the failure context.

**Step 6 — Documentation.** `doc-publisher` reads the diff, updates the relevant Confluence page (release notes, changelog, runbook), and adds a comment to the Jira issue with the PR link and a one-paragraph summary. It uses the Atlassian MCP's write tools (`mcp__atlassian__createConfluencePage`, `mcp__atlassian__addJiraComment`).

**Step 7 — Deploy.** This is the one place the pipeline insists on you. `deployer` is called only when you explicitly say "deploy" or after the report-generator surfaces "all checks green, ready to deploy." Its first action is AskUserQuestion with options: "Deploy to staging," "Deploy to production," "Cancel." Bypass mode is *not* set on this agent — Bash commands still need approval. A Notification hook pings your OS so you can answer from a different desktop.

**Step 8 — Report.** Stop hook fires `report-generator` exactly once per completed plan (using the hash-sentinel pattern from Chapter 14). It pulls the session's git activity, the Jira ticket transitions, the test status, and produces a structured end-of-work report committed to `reports/<date>-<branch>.md` and posted as a Confluence page.

Every step that touches files runs in a worktree. Every step that touches external systems goes through MCP with scoped credentials. Every step that involves judgment has the right model size (Opus where reasoning matters, Sonnet for structured tasks, Haiku for I/O-bound work). Every Stop has a guard. Every deploy has a human. **This is autonomy with rails.**

---

## Chapter 23. The CVE-Remediation Variant

Same architecture, slight reorder. When `npm audit` or a Dependabot alert fires (you can also trigger it manually with `/cve PLATFORM-1342`), the main session routes directly to `cve-remediator`. It runs in a worktree, with the Atlassian MCP available so it can read the CVE description from the linked security advisory ticket. It:

1. Reads the CVE advisory via WebFetch (with `allowed-domains` locked to nvd.nist.gov, github.com/advisories, and your security vendor)
2. Identifies the affected package and the recommended fix version
3. Bumps the dependency, runs `npm audit` to verify the issue is gone
4. Runs the test suite; if anything breaks, makes the minimal fixes
5. Hands off to `security-reviewer` to confirm no new issues introduced
6. Hands off to `doc-publisher` to update the security-fixes Confluence page and comment on the Jira ticket
7. Routes back to main session for human-gated deploy

The whole cycle for a simple CVE bump runs in under five minutes, fully isolated. Your role is review-and-merge.

The cve-remediator definition matters here because it's where WebFetch domain restriction shows up:

```markdown
---
name: cve-remediator
description: Patches a CVE in a dependency and validates the fix. Use when the user mentions a CVE, security advisory, Dependabot alert, or asks to remediate a vulnerability. Runs in an isolated worktree.
model: claude-opus-4-7
isolation: worktree
permissionMode: acceptEdits
maxTurns: 30
tools:
  - Read
  - Edit
  - Write
  - Grep
  - Glob
  - Bash(pnpm audit*)
  - Bash(pnpm install*)
  - Bash(pnpm test*)
  - Bash(git status)
  - Bash(git diff*)
  - Bash(git add*)
  - Bash(git commit -m *)
  - WebFetch
  - mcp__atlassian__getJiraIssue
  - mcp__atlassian__addJiraComment
disallowedTools:
  - Bash(git push*)
  - Bash(rm -rf *)
---

You patch CVEs. You do not introduce other changes.

## Procedure

1. Identify the CVE. If a Jira ticket is given, fetch it.
2. Read the advisory (WebFetch is allowed only for these domains:
   nvd.nist.gov, github.com/advisories, your-security-vendor.com).
3. Identify the affected package and the fix version.
4. Bump in `package.json`. Run `pnpm install`.
5. Run `pnpm audit` to verify the issue is gone.
6. Run `pnpm test`. If something breaks because of the bump:
   - Identify the minimum surface to patch
   - Make the change, re-run tests
   - If you can't fix it in 5 turns, stop and ask via AskUserQuestion
7. Commit: `fix(deps): bump <pkg> to <ver> for CVE-YYYY-NNNNN`
8. Add a Jira comment summarizing the fix.

## Constraints

- WebFetch is restricted to security-advisory domains. Any other URL: stop.
- Do not bump packages other than the one named in the CVE.
- Do not make non-security changes in the same commit.
```

The WebFetch domain restriction is enforced by your hooks (a PreToolUse hook on WebFetch that exits 2 if the URL isn't in the allowed list), not just by prompt — because prompts are guidance, hooks are policy.

---

## Chapter 24. The Atlassian Integration Layer

The Atlassian MCP (formally the Rovo MCP server) went GA in February 2026 and is the most underused MCP server in most teams' Claude Code setups. Once authenticated via OAuth (or API token for headless), the same server gives every agent — `spec-architect` reading Jira tickets, `doc-publisher` writing Confluence pages, `cve-remediator` reading security advisories, `report-generator` posting status updates — a structured surface against your entire Atlassian footprint.

The capability surface, as of mid-2026:
- Search and summarize across Jira issues, Confluence pages, and Compass components
- Create and update Jira issues and Confluence pages
- Respect existing user permissions — Claude only sees what you can see
- Admin controls and whitelisting via the Atlassian admin console
- IP allowlisting works the same as for direct API access

### Configuration

The MCP entry for `.mcp.json` (committed to repo, no secrets):

```json
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v1/mcp"
    }
  }
}
```

OAuth happens on first use via `/mcp`. The older `/sse` endpoint is deprecated and stops working after June 30, 2026, so make sure the URL says `/mcp`, not `/sse`. For headless usage (CI, scheduled tasks), use API token authentication instead — it's stable and doesn't disconnect mid-session.

### Three gotchas worth knowing

**IP allowlisting applies to MCP requests too.** If your Atlassian org allowlists corporate IPs, working from a coffee shop will silently break MCP tool calls. The fix is org-side: have your admin allow your VPN range or add your laptop's outbound IPs.

**The first-time installation requires an admin or someone with access to all referenced Atlassian apps.** If your org has Jira but not Confluence (or vice versa), the initial 3LO consent flow needs to be completed by someone with access to all apps that the MCP scopes mention. After that, individual users with access to just one app can complete their own consent flow.

**Permissions are inherited from the authenticated user.** This is the security model's strongest feature: the agent can't do more than you can. A `doc-publisher` agent running as you cannot edit pages you can't edit; a `spec-architect` agent reading a Jira ticket gets the same redactions you'd see. For shared automation, create a dedicated service account with narrowly-scoped permissions rather than running as a human user.

### How the pipeline uses Atlassian

In the pipeline from Chapter 22:

- `spec-architect` calls `mcp__atlassian__getJiraIssue` to read the source ticket, then `mcp__atlassian__searchConfluence` to find related design docs. Both read-only.
- `doc-publisher` calls `mcp__atlassian__updateConfluencePage` to add a release-notes entry, `mcp__atlassian__addJiraComment` to link the PR back to the ticket, and `mcp__atlassian__transitionJiraIssue` to move it from "In Progress" to "In Review."
- `deployer` calls `mcp__atlassian__transitionJiraIssue` to mark the ticket Done after a successful production deploy, plus a comment with the release tag.
- `report-generator` calls `mcp__atlassian__createConfluencePage` to publish the session summary, plus comments on every Jira ticket touched.

Total Atlassian tool calls in a typical session: 5–15. Total time saved: every single context switch you used to make between terminal and browser.

For team setups, get your Atlassian admin to whitelist the Claude domain in Rovo MCP settings so the OAuth flow doesn't dead-end on first use. The setting is under Atlassian Administration → Rovo → MCP server settings.

---

## Chapter 25. Everything Together: The Concentric Picture

Stepping back from the details: your full Claude Code setup is five concentric layers, each enforcing different guarantees.

The **innermost layer is CLAUDE.md and skill descriptions** — the standing instructions Claude reads every turn. Specific, concise, well-routed. This is where Claude *learns what to do*.

Around that, **skills** are the procedures Claude invokes when the description matches. Loaded on demand, cheap, support dynamic context injection. This is where Claude *learns how to do things*.

Around that, **agents** are the specialists who actually do the work. Own context, own tools, own permission posture, optionally own worktree. This is *who does the work*.

Around that, **MCP servers** are the external systems agents reach into — Jira, Confluence, GitHub, Sentry, Postgres. Tool Search defers schemas so context cost stays flat regardless of how many you connect. This is *what they can touch*.

Wrapping it all, **hooks** are the deterministic gates that fire regardless of what Claude decides. SessionStart for context injection, PreToolUse for blocking, PostToolUse for formatting, Stop and SubagentStop for quality gates, Notification for pulling you back in. This is *what must happen no matter what Claude wants*.

When you're staring at "Claude did the wrong thing again," the layer to fix is almost always the outer one. Wrong content? Skill body. Wrong procedure selection? Skill description. Wrong agent picked? CLAUDE.md routing. Wrong external action? MCP scope or agent's tool allowlist. Action that should never have happened? Missing hook. Each layer has a different lever, and most failures point at exactly one of them.

# Part VII — Greenfield, Brownfield, and Scale

The pipeline from Part VI is the destination. This part covers the road to it from three different starting points: a brand new project, a large existing codebase you've just joined, and an established team with multiple engineers.

The order of operations matters a lot — wiring up too much too soon is the single biggest cause of "I tried Claude Code and it was a mess." Each section below is the order I'd give someone in that position today.

---

## Chapter 26. Greenfield from Day One

Run `/init` (consider `CLAUDE_CODE_NEW_INIT=1` for the multi-phase interactive flow that also proposes skills and hooks). Accept the generated CLAUDE.md, then immediately trim it — `/init` over-generates. Keep build/test/lint commands, a one-line layout map, 2–3 must-follow conventions. Stop at ~60 lines.

Add 1–2 MCP servers at user scope only if the project genuinely needs external context (GitHub if you're shipping to a real repo; nothing else yet). Resist the urge to wire up Sentry, Linear, Slack, and Postgres on day one — you don't know what you actually need.

Don't write skills yet. Wait until you've typed the same instruction into chat three times. The third time, that's a skill. Same for hooks — wait for the second time you forget to lint before committing.

Auto memory does its work silently. Look at `/memory` after a week and you'll find Claude has captured patterns you'd have forgotten to write down.

The trap to avoid: copying someone else's elaborate `.claude/` setup before you have any feel for the project's own friction. Their pipeline solves their problems. Yours doesn't have those problems yet. Build the harness as you discover what hurts.

---

## Chapter 27. Brownfield and Large Codebases

The order matters here. Walk these in sequence; skipping ahead is what makes brownfield setups feel overwhelming.

**First**, run `/init` and let Claude draft a CLAUDE.md by analyzing the codebase. It will read the README, package.json, and conventions and produce a useful first draft. **Edit it down ruthlessly** — what's left should be what a new hire would actually need.

**Second**, set up path-scoped rules under `.claude/rules/` for areas with strong conventions: `.claude/rules/api.md` with `paths: src/api/**/*.ts`, `.claude/rules/migrations.md` for `db/migrations/**`, `.claude/rules/frontend.md` for the UI tree. This is where large codebases get their lift — instead of one bloated CLAUDE.md trying to cover every team's standards, each area's rules load only when relevant.

**Third**, in a monorepo, use `claudeMdExcludes` in `.claude/settings.local.json` to skip CLAUDE.md files from teams you're not working with. Ancestor CLAUDE.md files get picked up automatically, and in a monorepo with 50 packages that's noise.

**Fourth**, add MCP servers based on the actual investigation surface area. Brownfield work usually means tracing bugs across systems, so Sentry, the issue tracker, and (carefully, read-only) the production DB schema are high-value. Skip Slack/email/calendar servers — they don't help with code.

**Fifth**, build skills for the *investigation patterns* you find yourself repeating. "Find all callers of this function," "trace this error through the request lifecycle," "summarize how feature X works." These become invokable shortcuts that pay for themselves within a week. Pair them with `context: fork` to keep main-session context clean.

The brownfield-specific failure mode to watch for: assuming context loads will catch up to a large codebase. They won't. A 500K-line repo will overflow any context window if you let Claude read freely. The discipline is: every skill that explores the codebase runs in a forked subagent (so its reads don't pollute main), every agent has a tool allowlist that prevents reading half the repo, and `bug-investigator` returns hypotheses with file:line references rather than dumping file contents.

---

## Chapter 28. Established Teams

The shift here is from "configure for myself" to "configure for the team." Three moves:

**Promote stable skills to a plugin.** When two engineers have written the same skill in their personal `~/.claude/skills/`, that's the signal. Package it, version it, distribute via a private marketplace, retire the personal copies. The plugin format namespaces things (`/internal-platform:deploy` instead of `/deploy`), which is a small price for shared maintenance.

**Centralize rules through symlinks.** Maintain a shared rules repo, symlink it into every project's `.claude/rules/shared/`. A standards update propagates without per-project PRs. This is the team-scale equivalent of the path-scoped rules pattern from Chapter 4.

**Use managed settings for org-wide policy.** Code that must run before every commit goes in a managed hook. Tools that must be denied (network access, certain `Bash(*)` patterns) go in managed `permissions.deny`. Behavioral rules every developer must follow go in the `claudeMd` field of managed settings. None of these can be excluded by individual settings, which is the point.

For larger orgs, the pattern is plugin-on-marketplace-on-private-git: a private repo containing a marketplace manifest pointing at one or more plugins; teammates run `/plugin marketplace add <repo>` once, then `/plugin install internal-platform`, and they get the whole stack. Updates roll out the next time anyone runs `/plugin update`.

The hard part isn't the technology; it's the governance. Who maintains the plugin? Who reviews changes? How do you handle "skill triggered the wrong way on my code" complaints? In practice this works the same as any internal tooling: a platform team or platform-curious volunteers, with a Slack channel for issues and a contribution guide for proposing additions. Treat the plugin like a library: semver, deprecation policies, the works.

---

## Chapter 29. The Where-Does-This-Go Cheat Sheet

The implicit rule across all three of the above:

- A rule that must hold in every session, codebase-wide → CLAUDE.md (root)
- A rule that holds only when touching certain paths → `.claude/rules/*.md` with `paths:`
- A procedure invoked sometimes, no external systems → skill
- A procedure that needs external data (GitHub, Postgres, Sentry) → skill that calls MCP tools
- A capability against an external system → MCP server
- Something that must fire automatically at a fixed lifecycle event → hook
- A specialist worker with constrained tools and isolation → agent
- Anything you want to share with a teammate or another machine → wrap in a plugin
- A learned fact Claude figured out → leave it in auto memory; promote to CLAUDE.md only if it's a rule, not a learning

Walk this list top to bottom when you're stuck on "where does this thing live." The first bullet that fits is usually right.

# Part VIII — Debugging the Stack

The single biggest mistake when debugging Claude Code is jumping in at the wrong layer. "Claude ignored my instruction" could be a CLAUDE.md that didn't load, a skill description that didn't match, a hook that fired but exit-2'd silently, an MCP server that's connected but returned zero tools, a subagent that didn't inherit project memory, or `settings.local.json` overriding `settings.json`. Six radically different problems, all looking identical from the chat interface.

The debugging discipline you want is: **observe before you theorize**. Claude Code gives you a complete set of inspection commands to see what actually loaded vs. what you intended to load. Use them first, every time, before you start editing files. The flow is always the same — *is the thing loaded → did it fire → did it produce the right output → did Claude actually act on it?* — and each layer has a corresponding inspection step.

---

## Chapter 30. The Triage Tree

When something doesn't work, walk this tree top-down. Each step takes seconds; doing them in order avoids hours of misdirected fixing.

**Step 1 — `/context`.** This shows the entire context window broken down by category: system prompt, memory files, skills, MCP tools, conversation. Run it first, always. If your CLAUDE.md isn't in the memory section, nothing else matters — fix that first. If your skill descriptions aren't in the skills section, Claude literally cannot route to them. If `/context` shows you're at 85% of the window, the problem may be displacement: things you wanted loaded got compacted out.

**Step 2 — the targeted listing for the layer that's failing.** `/memory` for CLAUDE.md and rules issues. `/skills` for skill issues. `/agents` for agent issues. `/hooks` for hook issues. `/mcp` for MCP issues. `/permissions` for "Claude is blocked from doing X." Each one tells you whether the thing exists at all and what configuration is actually in effect.

**Step 3 — `/doctor` and `/status`.** `/doctor` validates the configuration files themselves — invalid keys, schema errors, malformed YAML frontmatter that caused something to be silently dropped. `/status` tells you which settings sources are merged into the current session, including whether managed (org-level) settings are in effect.

**Step 4 — Live evaluation.** If the thing is loaded but isn't firing or producing wrong output, you need to watch it run. `claude --debug hooks` for hook evaluation, `claude --debug mcp` for MCP connection and tool calls, `claude --debug api` for the model's request/response traffic. Verbose mode (`--verbose`) shows more session activity but has a known display-ordering bug; prefer `--debug` for serious investigation.

**Step 5 — Clean room.** If you've checked all of the above and it still doesn't add up, launch a session with zero configuration: `cd /tmp && CLAUDE_CONFIG_DIR=/tmp/claude-clean claude`. Managed settings still apply (they live at a system path), but everything in `~/.claude` and the project's `.claude` is bypassed. If the problem disappears, the cause is in your configuration; if it persists, the cause is environmental or model-side.

That's the whole tree. Walk it in order. The number of "Claude is broken!" debugging sessions that turn into "oh, my matcher was lowercase" is staggering.

---

## Chapter 31. The Inspection Commands

Each `/`-prefixed command gives you a different view of the running system.

### `/context` — the whole picture

This is the most important command in your kit. Output groups context occupants by category and shows token counts. A typical healthy session looks roughly like this:

```
System prompt:       3,200 tokens
Memory files:        4,800 tokens   (root CLAUDE.md + 3 rules files)
Skills (listings):   2,100 tokens   (12 skills)
Agents (listings):   1,400 tokens   (10 agents)
MCP tools:             900 tokens   (Tool Search defers schemas)
Conversation:       45,200 tokens
Tool results:       18,000 tokens
─────────────────────────────────
Total:              75,600 / 200,000 (38%)
```

What to look for:

If **memory files is 0**, your CLAUDE.md isn't being picked up — check that it's at the project root, that Claude was launched from inside the project, and that the file isn't hidden by an `@import` that points at a missing target.

If **skills is missing or much smaller than expected**, some skills failed to load. Run `/skills` and check the badge column — invalid frontmatter shows up as "schema error."

If **conversation + tool results is over 60% of total**, you're context-pressured and compaction is imminent. Use `/compact` manually before it auto-fires (auto-compact tends to run at the worst moment, mid-task). After compaction, run `/context` again to see what survived.

If **MCP tools is huge**, Tool Search isn't deferring — check that `ENABLE_TOOL_SEARCH` isn't explicitly set to `0` somewhere.

### `/memory` — what CLAUDE.md and rules actually loaded

Lists every CLAUDE.md and rules file Claude is currently aware of, with their path and load reason. A common surprise: you have a `CLAUDE.md` in your home directory plus one in the project, both load, and they say conflicting things — the project file should win (closer scope), but if you've been editing `~/.claude/CLAUDE.md` thinking it was the project file, it'll silently override your assumptions.

Also lists auto-memory entries with their topic file names. If you see a `MEMORY.md` entry larger than 25KB or 200 lines, only the first 200 lines / 25KB are actually loaded — content past that is invisible until Claude reads the topic files on demand.

### `/skills` — what's discoverable

Lists every available skill: project, user, plugin, with a status badge. Watch for:

- **"user-only"** badge — skill has `disable-model-invocation: true`; Claude won't auto-trigger it, you must type `/skill-name`.
- **"hidden"** badge — `user-invocable: false`; Claude can invoke but it's not in the slash menu.
- **"schema error"** badge — frontmatter is malformed; skill is loaded but won't be triggered. Run `/doctor` for the specific error.
- **No badge but Claude doesn't invoke** — the description doesn't match how you phrase requests. This is where 80% of skill-triggering issues live. Reword the description with concrete trigger phrases, or run the skill-creator's Improve mode to optimize the description with real evaluation.

### `/agents` — what subagents exist

Same idea, lists every agent with its source (project/user/plugin), model, tool count, and isolation mode. A subtle one to watch: plugin-provided agents *don't* support `hooks`, `mcpServers`, or `permissionMode` even if they're listed in the agent file — those fields are silently ignored for plugin agents. If you need any of those, move the agent definition to `.claude/agents/`.

### `/hooks` — what's wired

Lists every hook by event, with matcher and command. **If your hook isn't here, it isn't loaded.** If it's not loaded, check two things in order: first, is it under the `"hooks"` key in `settings.json` (there is no standalone `hooks.json`)? Second, did you put it in `~/.claude.json` instead of `~/.claude/settings.json`? Those are two different files and the first one holds only app state — it ignores `hooks`, `permissions`, and `env`.

If the hook IS in `/hooks` but isn't firing, the matcher is the cause 90% of the time. Three specific failure modes:

```json
// WRONG — matcher as array. Schema error; hook is dropped silently.
"matcher": ["Edit", "Write"]

// WRONG — lowercase. Tool names are case-sensitive: Bash, Edit, Write.
"matcher": "bash"

// RIGHT — single string with pipe separator
"matcher": "Edit|Write|MultiEdit"
```

`/doctor` catches the first one (it's a schema violation). The lowercase one fails silently — the matcher just never matches anything. Always capitalize.

### `/mcp` — server status and tool counts

Each server shows: **connected / disconnected / failed**, the tool count, and approval status. Three states worth recognizing:

- **Disconnected, "needs approval"** — Project-scoped servers in `.mcp.json` require one-time approval the first time you start a session with them. If you dismissed the prompt, the server stays disabled. Select the server in `/mcp` and approve.
- **Failed to start** — Almost always a path issue. Relative paths in `command` or `args` resolve against the directory you launched from, not the `.mcp.json` location. Use absolute paths for local scripts.
- **Connected, 0 tools** — Server started but isn't returning tool definitions. Hit Reconnect; if it stays at 0, run `claude --debug mcp` and inspect the server's stderr. This is the case where Atlassian MCP can confuse people: the connection succeeds but tools don't appear until you've completed OAuth.

### `/doctor`, `/status`, `/permissions`

`/doctor` validates every loaded configuration file and prints schema errors. Run this whenever something silently dropped — if a skill's frontmatter has a typo, `/doctor` will tell you which file and which key. `/status` shows which settings sources are active (managed, user, project, local) so you can spot "oh, settings.local.json is overriding my settings.json" before it burns an hour. `/permissions` shows the merged allow/deny rules in effect — useful when "Claude won't run my command" turns out to be a deny rule you forgot you added.

---

## Chapter 32. Verbose Mode vs. Debug Mode

`--verbose` and `--debug` look similar but serve different purposes.

`--verbose` is a session-level setting that prints expanded action traces inline — what tools fired, what skills loaded, what hooks ran. It's useful for "watch what the model is doing" debugging. Known issue: in some recent versions on Linux, hook completion messages accumulate and display out of order in verbose mode. SessionStart messages can appear at the bottom of the output. The fix: don't trust verbose mode's ordering for hook timing; use debug mode for that.

`--debug` takes a comma-separated list of subsystems to trace: `hooks`, `mcp`, `api`, `tools`. It writes structured logs that are accurate and ordered, at the cost of being noisier:

```bash
claude --debug hooks           # hook evaluation only
claude --debug mcp             # MCP transport
claude --debug api             # request/response with the model
claude --debug "hooks,mcp"     # multiple subsystems
claude --debug "api,mcp,tools" # for "Claude is doing the wrong thing"
```

The hook debug log shows, for every event: which matchers were checked, whether each matched, the hook's stdin payload, the hook's stdout/stderr, and its exit code. This is how you find "the matcher matched but the hook exit-2'd silently for reasons" cases — the stderr in the log tells you exactly what the hook printed before exiting.

Two environment variables also work for headless or CI usage:

```bash
export CLAUDE_DEBUG=1            # debug everything
export CLAUDE_LOG_LEVEL=debug    # subprocess inheritance
```

These propagate into hooks and MCP servers spawned by Claude Code, so your hook scripts can do `echo "..." >&2` and the output ends up in the debug log alongside Claude's own traces. Tagging your hook stderr with a prefix is the trick for finding them quickly:

```bash
echo "[guard_bash] blocked: $CMD" >&2
```

---

## Chapter 33. Debugging Each Layer

### Memory and CLAUDE.md

The two questions in order: *did it load?* (use `/memory`) and *is the instruction specific enough?* (read it as a stranger would).

If `/memory` doesn't list the file, the file's location is wrong. Check the path against the hierarchy from Chapter 3: managed → user (`~/.claude/CLAUDE.md`) → project (`./CLAUDE.md`) → local (`./CLAUDE.local.md`). Subdirectory CLAUDE.md files load *on demand* when Claude reads a file in that subdirectory — they're invisible at session start. If you expected the subdirectory file to be loaded immediately, that's the bug; either move the rules up or accept that they only apply when Claude works in that subtree.

If `/memory` lists the file but Claude ignores a specific instruction, the issue is the writing, not the loading. Three patterns adhere poorly:

- **Vague phrasing.** "Be careful with X" doesn't tell Claude what to do. Write it as a procedure: "When editing X, first check Y, then do Z."
- **Conflicting rules.** Two files contradicting each other. CLAUDE.md vs. rules vs. auto-memory — when they disagree, Claude does whatever it does. Use `/memory` to find both sources, decide which wins, and delete the loser.
- **File too long.** Past ~80 lines, individual rules in CLAUDE.md get less attention. Split into `.claude/rules/*.md` with `paths:` frontmatter so the right rules load at the right times.

The deepest debug move for memory issues is the **InstructionsLoaded hook**, which logs exactly which files and rules were merged into the system prompt and in what order. Wire it once to a log file and you'll never wonder again whether a rule made it in:

```json
{
  "hooks": {
    "InstructionsLoaded": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "jq '.loaded_files' >> ~/.claude/instructions.log"
          }
        ]
      }
    ]
  }
}
```

Tail that file and you have a permanent record of what loaded when.

### Skills

The two questions: *is the skill discoverable?* (use `/skills`) and *does the description match how requests get phrased?*

The "schema error" badge in `/skills` catches malformed frontmatter; `/doctor` gives you the exact error. The most common one: forgetting the closing `---` after frontmatter, or using tabs instead of spaces in YAML. The second most common: a skill file placed at `.claude/skills/name.md` instead of `.claude/skills/name/SKILL.md`. Skills *must* be directories with a `SKILL.md` inside; a bare markdown file at the skills root won't be picked up.

For descriptions that don't match, you need data. Don't guess — run the skill-creator's Eval and Improve modes against a real test set:

```bash
/skill-creator eval spec-checker
/skill-creator improve spec-checker --max-iterations 5
```

Eval will run a dozen prompts and tell you which the skill triggered on and which it didn't. Improve will iterate the description against that data and produce a version that triggers more reliably. The under-triggering tendency Anthropic documents is real, and improving descriptions empirically beats writing them by intuition.

If you're investigating a *specific* failure ("I said X and the skill didn't fire"), the fast manual test is: write that exact prompt as an eval case and see if the current description matches it. If three runs out of three don't trigger, the description needs the trigger phrases you used in the prompt.

### Hooks

After `/hooks` confirms the hook is loaded with the right matcher, the next question is whether it's firing correctly. Three techniques:

**Manual JSON test.** Hooks read JSON from stdin; you can run them outside Claude Code by piping in a representative payload:

```bash
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | \
  bash .claude/hooks/guard_bash.sh
echo "Exit: $?"
```

Quickly confirms the script works in isolation. If it works here but doesn't work in Claude Code, the issue is the matcher (running on wrong tool) or the payload shape differs from what you expected. For Stop hooks, also test the anti-loop guard:

```bash
echo '{"stop_hook_active":true}' | bash .claude/hooks/test_gate.sh
echo "Exit: $?"   # Should be 0 — the guard exited early
```

**Live debug.** Run `claude --debug hooks` and trigger the action. The log shows the matcher check, the hook invocation, and stdin/stdout/stderr/exit code. This catches "hook fires but produces wrong output" — the stderr is right there in the log.

**The hook never fires at all.** Walk this short list:

1. Is it under `"hooks"` in `settings.json`? Not in a standalone `hooks.json`. Not in `~/.claude.json` (which is app state, not config).
2. Matcher case correct? `Bash`, not `bash`.
3. Matcher a string, not an array? Arrays are schema errors.
4. Tool name spelled correctly? Typos fail silently.
5. Hook script executable? `chmod +x .claude/hooks/*.sh`.
6. Path absolute or correctly using `$CLAUDE_PROJECT_DIR`? Relative paths resolve against launch directory.
7. On Windows + Git Bash, line endings LF not CRLF? CRLF breaks the shebang.

The infinite-loop case: a Stop hook with exit 2 plus no `stop_hook_active` check **will loop forever** until the session times out. The symptoms are obvious (Claude won't stop responding, token usage spikes), the fix is one line:

```bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi
```

Add it as the *first* action in every Stop and SubagentStop hook, no exceptions.

### MCP servers

`/mcp` is the first stop, but `claude --debug mcp` is where real debugging happens. The flag prints the JSON-RPC traffic with the server: initialization handshake, tool list response, tool call requests and responses. Common patterns:

**"Connected, 0 tools."** The server's tool list response is empty or malformed. Check the server's own logs. For HTTP MCP servers, often this is an authentication issue: the connection succeeds but the server can't authenticate the user, so it returns an empty tool set rather than an error. For Atlassian specifically, this means OAuth hasn't completed — the `/mcp` UI should prompt you; if it didn't, manually trigger an OAuth flow via the server's URL.

**Server fails to start.** `--debug mcp` shows the spawn error. Almost always either a missing executable on PATH or a relative path in `command`. Fix: use `npx -y package-name` for npm-distributed servers (npx resolves automatically) or absolute paths for local scripts.

**Tools defined but Claude doesn't call them.** Tool Search defers schemas, which is normally fine, but it also means Claude has to discover them on demand. If a skill's `allowed-tools` field doesn't list `mcp__server__tool` explicitly, Claude has to find it through the discovery search, which may not fire if the skill's body doesn't make the need obvious. Solution: list MCP tools explicitly in skill `allowed-tools` so the schema loads when the skill loads.

**OAuth token expired mid-session.** The server starts returning 401s. `/mcp` will show the server as connected but tool calls fail. Most servers re-prompt for auth automatically; if not, disconnect and reconnect from `/mcp`.

For the Atlassian MCP specifically, three gotchas: (1) the older `/sse` endpoint is deprecated and stops working June 30, 2026 — make sure your URL is `/mcp`, not `/sse`; (2) IP allowlisting on your Atlassian org applies to MCP requests too, so working from a coffee shop can break it; (3) for headless usage (CI, scheduled tasks), use API token auth instead of OAuth.

### Agents

The agent-level failure modes split into "agent never invoked" and "agent invoked but did the wrong thing."

For "never invoked," `/agents` is the listing. If the agent isn't there, the file isn't being read — check `.claude/agents/<name>.md`, valid frontmatter (`name` and `description` required), and `/doctor` for schema errors. If the agent is listed but Claude never delegates to it, the description doesn't match how you phrase requests. Add explicit trigger phrases to the description: "Use when the user says 'X', 'Y', or 'Z'." For the pipeline, this is why CLAUDE.md routing rules ("Bug → bug-investigator first") matter — they bias Claude toward the agent even if the description is borderline.

For "invoked but did the wrong thing," the agent's full session is captured in the transcript. After it returns, you can ask the main session "show me what the bug-investigator agent's full output was" and it'll dump everything. From there, identify whether the failure was:

- **Bad tool allowlist** — the agent tried to do something its tools didn't permit. Add to `tools:` or remove from `disallowedTools:`.
- **Hit maxTurns** — the agent ran out of turns before finishing. Bump `maxTurns:` or narrow the task scope.
- **Wrong context** — the agent didn't have enough information. Subagents **don't always inherit project CLAUDE.md**; put critical context directly in the agent's body (which becomes its system prompt).
- **Hook in the agent definition fired wrong** — agents can define their own hooks in frontmatter. A misconfigured SubagentStop test gate inside the agent can cause it to spin. Same `stop_hook_active` guard applies inside agent-level hooks.

The "doesn't inherit project memory" gotcha is the most common subagent debugging surprise. If your agent needs to know the build commands, write them into the agent body — don't rely on CLAUDE.md being in scope.

### Plugins

Plugin issues are almost always loading-related. `/plugin list` shows installed plugins and their status. If a plugin doesn't appear, the marketplace probably failed to fetch (network proxy, expired auth) — run `/plugin marketplace add <url>` again to refresh. If the plugin loads but its components don't show up in `/skills`, `/agents`, or `/hooks`, the plugin manifest probably has a wrong path. Plugin layout requires `skills/<name>/SKILL.md`, `agents/<name>.md`, `hooks/hooks.json` at the *plugin root*, not nested under `.claude/`.

The big plugin-specific gotcha: plugin-provided subagents silently ignore `hooks`, `mcpServers`, and `permissionMode` fields. If your plugin's agent has a SubagentStop hook in its frontmatter, the hook is silently dropped. The fix is either: (1) define those agents in `.claude/agents/` (project scope, not plugin scope) so the security-sensitive fields work, or (2) move the hook from the agent file to settings.json with a matcher targeting the agent name.

---

## Chapter 34. The Clean-Room Technique

There's a class of bug where you've checked everything, nothing is obviously wrong, and Claude is still misbehaving. This usually means you have a conflict between two configurations or a setting you've forgotten about. The clean-room technique resolves it in 30 seconds.

```bash
mkdir /tmp/claude-clean
cd /tmp/scratch-project  # any directory with no .claude or CLAUDE.md
CLAUDE_CONFIG_DIR=/tmp/claude-clean claude
```

This launches with zero personal config. No user CLAUDE.md, no user settings, no user skills/agents/hooks, no memory. Managed (org-level) settings still apply because they live at a system path. If the issue persists here, it's environmental (network, model availability, an Anthropic-side issue) or you've found a real bug.

**The one-flag shortcut (2.1.169+): `--safe-mode`.** As of 2.1.169 you can start Claude Code with all customizations — CLAUDE.md, plugins, skills, hooks, MCP servers — disabled in a single flag: `claude --safe-mode` (or set `CLAUDE_CODE_SAFE_MODE`). This is the fastest possible "is it my config or the tool?" check: if the problem vanishes under `--safe-mode`, it's something in your configuration and you move to the bisection below; if it persists, it's environmental or a real bug. Reach for `--safe-mode` first, and only set up the `CLAUDE_CONFIG_DIR` clean room when you specifically need managed settings out of the picture too, or when you want to bisect by copying config subtrees back in.

If the issue *disappears* in the clean room, the cause is in your configuration. Now bisect: copy your real `~/.claude/` to the clean dir, run again, see if it reappears. If yes, the cause is in user-scope config. Half it again: copy only `~/.claude/CLAUDE.md`, test; then add `~/.claude/skills/`; then `~/.claude/agents/`. Within four iterations you'll know which subtree contains the problem, and from there a quick read of the files identifies the culprit.

For the project-scope side, do the inverse: start in your real project, then move `.claude/` aside (`mv .claude .claude.bak`) and see if the problem persists. If it goes away, walk back: rename `.claude.bak/settings.json` back, test; add `agents/`; add `skills/`; add `hooks/`. Same bisection.

This is mechanical but it's reliable. Every time someone posts "I've tried everything and Claude is broken," the clean-room bisection identifies the cause in under five minutes.

---

## Chapter 35. A Worked Debugging Example

Here's a realistic scenario: you ask Claude to "implement next plan step" and it does the implementation, but the `security-reviewer` never runs and the Stop hook fires `report-generator` even though work isn't actually done. Three things went wrong simultaneously. Here's how you'd unwind it.

**Step 1 — Reproduce in `--debug` mode.** Restart with `claude --debug "hooks,api"` and re-run the prompt. Watch the live log.

**Step 2 — Verify each layer fired or didn't.** In the debug output, look for:

- Did `implementer` get invoked? (Look for a Task tool call to `implementer`.) If yes, good.
- Did `implementer`'s SubagentStop fire? (Look for `SubagentStop` event in the log.) If yes, what was the exit code?
- Did the project's `auto_review_handoff.sh` SubagentStop hook fire? (Look for that command in the log.) If no, the matcher is wrong.
- Did the project's `test_gate.sh` Stop hook fire? Did it pass or block?
- Did `report_gate.sh` fire? What did it print?

Suppose the debug log shows:

```
[SubagentStop:implementer] stop_hook_active=false
[SubagentStop matcher check] "implementer|cve-remediator" against agent="implementer"
[SubagentStop matcher result] NO MATCH
[SubagentStop hook] (none fired)
```

That's the clue. The matcher didn't match even though `implementer` was the agent. The reason: SubagentStop's matcher is checked against the *tool name pattern*, not the agent name, in older Claude Code versions. Newer versions added agent-name matching. Check your version: `claude --version`. If you're on a version that doesn't support agent matching in SubagentStop, the workaround is to omit the matcher and gate inside the hook:

```bash
#!/usr/bin/env bash
INPUT=$(cat)
AGENT=$(echo "$INPUT" | jq -r '.agent_type // empty')

if [ "$AGENT" != "implementer" ] && [ "$AGENT" != "cve-remediator" ]; then
  exit 0  # Not the agent we care about
fi

# ... rest of hook
```

**Step 3 — Verify the fix.** Re-run, watch the debug log again, confirm `auto_review_handoff.sh` fires now.

**Step 4 — Address the second issue.** Why did `report_gate.sh` fire when work wasn't done? Read its logic: it checks for open `- [ ]` items in `prompt_plan.md`. Maybe `implementer` marked the item done even though tests were failing — check the `prompt_plan.md` diff. If yes, the fix is in `implementer`'s body: tighten the "mark done" step to require tests passing first, or add a `test_gate.sh` SubagentStop hook to the agent itself so it can't finish with failing tests.

**Step 5 — Add instrumentation.** This whole debugging session would have been faster with explicit logging. That's Chapter 36.

---

## Chapter 36. Building Debug Instrumentation

A pipeline as large as the one in Part VI gets expensive to debug ad-hoc. Five small instrumentation additions pay for themselves the first time you hit a real bug.

**An always-on audit log.** Cheap, covers every event, gives you ground truth:

```bash
#!/usr/bin/env bash
# .claude/hooks/audit.sh — runs on every event we care about
INPUT=$(cat)
EVENT="$1"  # passed as argument from settings.json
echo "$(date -Iseconds) [$EVENT] $(echo "$INPUT" | jq -c '.')" >> ~/.claude/audit.log
exit 0
```

Wire it into settings.json on every event you want logged:

```json
{
  "hooks": {
    "SubagentStart": [{ "hooks": [{ "type": "command", "command": "bash .claude/hooks/audit.sh SubagentStart" }] }],
    "SubagentStop":  [{ "hooks": [{ "type": "command", "command": "bash .claude/hooks/audit.sh SubagentStop" }] }],
    "Stop":          [{ "hooks": [{ "type": "command", "command": "bash .claude/hooks/audit.sh Stop" }] }]
  }
}
```

Now `tail -f ~/.claude/audit.log` during a session shows you the event stream in real time. The next time something doesn't fire when expected, the audit log answers "did the event even occur?" without having to launch `--debug`.

**A skill-trigger log.** Wrap each skill's body in a one-line stderr write that the hooks then capture:

```markdown
---
name: spec-checker
description: ...
---
<!-- DEBUG: emit a signal so audit.sh sees this skill was invoked -->
!`echo "[skill:spec-checker] invoked at $(date -Iseconds)" >&2`

You are the spec-checker...
```

Now your audit log shows skill activations alongside events.

**A test-history log.** Your `test_gate.sh` already runs tests; have it record results to `.claude/.test-history.jsonl`:

```bash
RESULT="pass"
if ! pnpm test --silent > /tmp/test-out 2>&1; then
  RESULT="fail"
fi
jq -n --arg t "$(date -Iseconds)" --arg r "$RESULT" \
  '{timestamp:$t, result:$r}' >> "$CLAUDE_PROJECT_DIR/.claude/.test-history.jsonl"
```

When `report-generator` runs at session end, it reads this file and includes "tests went red 2x then green" in the report. You get the failure pattern preserved without rerunning anything.

**An MCP request log.** Every MCP server call logged so you can debug "Atlassian returned weird data" issues:

```bash
# .claude/hooks/log_mcp_calls.sh — wired as PostToolUse with matcher mcp__.*
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')
RESULT=$(echo "$INPUT" | jq -c '.tool_response')
echo "$(date -Iseconds) [MCP] $TOOL $RESULT" >> ~/.claude/mcp.log
exit 0
```

`tail ~/.claude/mcp.log` shows you exactly what each MCP server returned for every call. When Confluence updates fail or Jira transitions go to the wrong state, the response is right there.

**A startup self-check.** A SessionStart hook that walks your config and warns about common problems before they bite:

```python
#!/usr/bin/env python3
# .claude/hooks/startup_check.py
import json, os, sys
from pathlib import Path

warnings = []

settings_file = Path(".claude/settings.json")
if settings_file.is_file():
    s = json.loads(settings_file.read_text())
    pre_tool = s.get("hooks", {}).get("PreToolUse", [])
    if not any("guard_protected_files" in str(h) for h in pre_tool):
        warnings.append("guard_protected_files hook is missing!")

for h in Path(".claude/hooks").glob("*.sh"):
    if not os.access(h, os.X_OK):
        warnings.append(f"{h} is not executable (chmod +x needed)")

gitignore = Path(".gitignore")
if gitignore.is_file() and ".env" not in gitignore.read_text():
    warnings.append(".env is not in .gitignore!")

if warnings:
    msg = "STARTUP CHECK FAILURES:\n" + "\n".join(f"- {w}" for w in warnings)
    print(json.dumps({"hookSpecificOutput": {"hookEventName": "SessionStart", "additionalContext": msg}}))
```

The session starts with these warnings visible in Claude's context, so it can mention them to you or block actions until you fix them.

The companion starter kit (Appendix D) ships these instrumentation hooks wired up.

---

## Chapter 37. Common Failure Patterns Reference

The ones that actually steal hours from people:

**"Settings change isn't picked up."** Edits to `settings.json` apply after a brief file-stability delay, not instantly. Wait 2 seconds, run `/hooks` again. If still stale, you're editing the wrong file — check whether `~/.claude.json` (app state) is being edited when you meant `~/.claude/settings.json` (config).

**"Skill works for me but not my teammate."** Skill is at `~/.claude/skills/` (user scope) on your machine. They don't have it. Move to `.claude/skills/` (project scope) and commit.

**"Hook works manually but not in Claude Code."** The hook's environment is different. Specifically, `$CLAUDE_PROJECT_DIR` is set during hook execution and *not* when you test manually. If your hook uses it, simulate: `CLAUDE_PROJECT_DIR=$(pwd) bash hook.sh`.

**"MCP server worked yesterday, doesn't today."** OAuth token expired. `/mcp` shows the server connected but tool calls 401. Disconnect, reconnect.

**"Subagent gets confused about basics like build commands."** Subagents don't inherit project CLAUDE.md. Put critical context into the agent's body, not just in `CLAUDE.md`.

**"Plan mode bypassed even though I'm in plan mode."** Some agents and skills have `permissionMode` in their frontmatter that overrides the session default. Check `/agents` and `/skills` for the active permission mode of whatever's running.

**"AskUserQuestion returned empty answers."** Regression in v2.1.104 when called from plugin skills with `bypassPermissions`. Multi-select-array handling was fixed in 2.1.136 and auto-mode suppression in 2.1.147. Check `claude --version` and upgrade past 2.1.147.

**"Claude won't stop responding."** Stop hook missing the `stop_hook_active` guard. Add it.

**"Auto-compact destroyed my context."** It re-injects root CLAUDE.md after compaction, but skill bodies and tool outputs are dropped. Add a `SessionStart` hook with matcher `compact` that re-injects critical state.

**"Plugin agent's hooks aren't firing."** Plugin agents silently ignore `hooks`, `mcpServers`, `permissionMode`. Move the agent definition to `.claude/agents/`.

---

## Closing observation

The debugging discipline that scales is the same as the discipline for the rest of this stack: **make things observable, gate behavior with deterministic checks, and lean on the inspection commands before theorizing**. Claude Code gives you `/context`, `/memory`, `/skills`, `/agents`, `/hooks`, `/mcp`, `/doctor`, `/status` — eight commands that together answer "is the thing loaded, is it configured correctly, is it firing, is it producing the right output." Almost every Claude Code bug you'll ever hit reduces to one of those four questions; the inspection commands tell you which.

The pipeline from Part VI is large enough that without instrumentation, you'll spend real time debugging it. With the audit log, MCP call log, test-history log, and startup self-check wired in, future failures explain themselves. The investment is an hour; the savings are every time something goes sideways for the rest of the project's life.

# Appendices

---

## Appendix A. The Layer Decision Matrix

When you're stuck on "where does this thing belong," walk this table top to bottom. The first row that matches is usually right.

| If you want to express… | Put it in… | Why |
|---|---|---|
| A rule that must hold in **every session, codebase-wide** | Root `CLAUDE.md` | Loaded every session, survives compaction |
| A rule that holds **only when touching certain paths** | `.claude/rules/<topic>.md` with `paths:` frontmatter | Zero token cost when not relevant |
| A **personal preference** across all your projects | `~/.claude/CLAUDE.md` | User scope, doesn't pollute team config |
| A **personal project-specific** override | `./CLAUDE.local.md` (gitignored) | Local scope, your machine only |
| An **org-wide policy** users can't override | Managed `CLAUDE.md` at the OS managed path | Above all user scopes |
| A **procedure** Claude invokes sometimes, no external systems | Skill at `.claude/skills/<name>/SKILL.md` | On-demand load, cheap until invoked |
| A procedure **that needs external data** (GitHub, Postgres, Sentry) | Skill that calls MCP tools (`allowed-tools: mcp__server__tool`) | Skill = procedure; MCP = capability |
| A **capability against an external system** | MCP server (`.mcp.json` for shared, `claude mcp add --scope user` for personal) | Typed external tool interface |
| **Project-team shared** MCP server | `.mcp.json` at project root, **no secrets**, use `${ENV_VAR}` | Committed to git, env vars per developer |
| Something that **must fire automatically at a lifecycle event** | Hook in `settings.json` under `"hooks"` | Deterministic, runs whether Claude wants to or not |
| A **block on dangerous actions** that can't be bypassed | PreToolUse hook (exit 2) or `permissions.deny` rule | Enforcement, not guidance |
| A **format/lint after edit** | PostToolUse hook with `async: true` | Doesn't block Claude's next turn |
| A **quality gate at end of work** | Stop hook with `stop_hook_active` guard | Forces continuation if tests fail |
| A **specialist worker** with constrained tools and isolation | Agent at `.claude/agents/<name>.md` | Own context, own tools, own model |
| An **agent doing risky modifications** | Agent with `isolation: worktree` | Blast radius bounded to a temp checkout |
| Something **distributable to teammates or other projects** | Plugin (`/plugin marketplace` + `/plugin install`) | Versioned, namespaced, updatable |
| A **learned fact Claude figured out** | Leave in auto memory; promote to CLAUDE.md only if it's a rule | Auto memory is for learnings, not rules |
| An **interactive decision** during a workflow | `AskUserQuestion` invocation in skill body or CLAUDE.md directive | Waits by default (2.1.200); can't be called from subagents |
| A **prompt-driven workflow** invoked manually | Custom slash command at `.claude/commands/<name>.md` | Manual trigger; skill auto-triggers, command doesn't |

The bottom-line decision rule one more time: **if it must be true in every session, it's CLAUDE.md or a `.claude/rules/` file. If it's a procedure invoked sometimes, it's a skill. If it's a capability against an external system, it's an MCP server. If you want to ship any of the above to others, wrap it in a plugin. If it must happen at a fixed lifecycle event no matter what, it's a hook. If it's who does the work, it's an agent.**

---

## Appendix B. Command and Flag Reference

### Inspection commands (inside a session)

| Command | What it shows |
|---|---|
| `/context` | Full context window breakdown by category, with token counts |
| `/memory` | CLAUDE.md and rules files loaded, plus auto-memory entries |
| `/skills` | Available skills with status badges (project, user, plugin) |
| `/agents` | Configured subagents with source, model, tool count, isolation mode |
| `/hooks` | Active hook configurations grouped by event, with matcher + command |
| `/mcp` | Connected MCP servers, status, tool counts, approval state |
| `/permissions` | Resolved allow/deny rules currently in effect |
| `/doctor` | Configuration diagnostics: invalid keys, schema errors, install health |
| `/status` | Active settings sources (managed, user, project, local) |
| `/plugin list` | Installed plugins and their status |

### Session control commands

| Command | What it does |
|---|---|
| `/compact` | Force compaction of conversation history now |
| `/plan` (or shift-tab) | Toggle plan mode |
| `/goal <condition>` | Set a completion condition; Claude works across turns until met (2.1.139) |
| `/workflows` | View dynamic-workflow runs orchestrating many background agents (2.1.154; explicit trigger keyword is `ultracode` as of 2.1.160) |
| `/reload-skills` | Re-scan skill directories without restarting the session (2.1.152) |
| `/code-review [effort]` | Correctness-bug review at a chosen effort; `--fix` applies findings, `--comment` posts inline PR comments (renamed from `/simplify`, 2.1.147–2.1.152) |
| `/simplify` | Cleanup-only review (reuse, simplification, efficiency) that applies fixes (2.1.154) |
| `/ultrareview [PR#]` | Cloud-based parallel multi-agent review of the branch or a PR (2.1.111) |
| `/usage` | Usage and cost view; per-category breakdown by skills/subagents/plugins/MCP (merges `/cost` + `/stats`, 2.1.118; breakdown 2.1.149) |
| `/usage-credits` | Manage usage credits (renamed from `/extra-usage`, 2.1.144) |
| `/less-permission-prompts` | Scan transcripts and propose a read-only allowlist for settings (2.1.111) |
| `/tui fullscreen` | Switch to flicker-free fullscreen rendering in place (2.1.110) |
| `/focus` | Toggle focus view (split from `Ctrl+O`, 2.1.110) |
| `/effort` | Open the effort slider (Faster ↔ Smarter); supports `xhigh` on Opus 4.7/4.8 |
| `/skill-creator create` | Start interactive skill creation |
| `/skill-creator eval <name>` | Evaluate a skill against test queries |
| `/skill-creator improve <name>` | Auto-optimize a skill's description |
| `/skill-creator benchmark <name>` | Variance analysis on triggering |

### CLI flags

| Flag | Purpose |
|---|---|
| `--debug hooks` | Live hook evaluation logging |
| `--debug mcp` | MCP transport logging |
| `--debug api` | Request/response with the model |
| `--debug "hooks,mcp,api"` | Multiple subsystems |
| `--mcp-debug` | Synonym for `--debug mcp` in some versions |
| `--verbose` | Inline action traces (display ordering can be unreliable) |
| `--plugin-dir <path>` | Load a plugin from a local directory (also accepts a `.zip`) |
| `--plugin-url <url>` | Load a plugin from a URL/zip |
| `--effort <level>` | Set effort (`low`…`high`, `xhigh`, `max`) for the session |
| `--agent <name>` | Run/override the agent for the session (honors its frontmatter; 2.1.157) |
| `--fallback-model <id>` | Model to fall back to for the rest of the session if the primary is unavailable (2.1.152) |
| `--from-pr <url>` | Start from a PR/MR (GitHub, GHE, GitLab, Bitbucket) |
| `--worktree` | Run the session in an isolated git worktree |
| `--bg` / `--bg --exec '<cmd>'` | Start a background session (or run a shell command as one, 2.1.154) |
| `--version` | Print Claude Code version |
| `--safe-mode` | Start with all customizations (CLAUDE.md, plugins, skills, hooks, MCP) disabled (2.1.169) |
| `claude agents` | Open the agent-view dashboard of all sessions (2.1.139) |
| `claude agents --json` | Emit the live session list as JSON for scripting (2.1.145) |
| `claude plugin init <name>` | Scaffold a new plugin in `.claude/skills` (2.1.157) |
| `claude plugin details <name>` | Show a plugin's component inventory + projected token cost (2.1.139) |
| `claude ultrareview [target]` | Run `/ultrareview` non-interactively from CI (`--json`; 2.1.120) |
| `claude project purge [path]` | Delete all Claude Code state for a project (2.1.126) |
| `claude mcp add` | Register an MCP server |
| `claude mcp list` | Show configured MCP servers |
| `claude mcp doctor` | Diagnose MCP connection problems |

### Environment variables

| Variable | Purpose |
|---|---|
| `CLAUDE_CONFIG_DIR` | Override config root (clean-room debugging) |
| `CLAUDE_DEBUG=1` | Debug everything |
| `CLAUDE_LOG_LEVEL=debug` | Subprocess inheritance |
| `CLAUDE_VERBOSE=1` | Verbose for scripts/CI |
| `CLAUDE_PROJECT_DIR` | Set during hook execution (use in hook scripts) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` | Disable auto memory per session |
| `CLAUDE_CODE_FORK_SUBAGENT=1` | Enable subagent forks |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Enable agent teams (implicit team since 2.1.178; `TeamCreate`/`TeamDelete` removed) |
| `CLAUDE_CODE_SAFE_MODE` (or `--safe-mode`) | Start with all customizations disabled for troubleshooting (2.1.169) |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | Hide bundled skills/workflows/built-in commands from the model (2.1.169) |
| `CLAUDE_CODE_NEW_INIT=1` | Multi-phase interactive `/init` |
| `ENABLE_TOOL_SEARCH` | Toggle Tool Search (default on; off by default on Vertex) |
| `ANTHROPIC_MODEL` | Default model for first turn |
| `CLAUDE_CODE_EFFORT_LEVEL` | Default effort for CLI sessions (`max` is API-only; `xhigh` on Opus 4.7/4.8) |
| `CLAUDE_EFFORT` | Active effort level, exposed to hooks and Bash tool commands (2.1.133) |
| `CLAUDE_CODE_ENABLE_AUTO_MODE=1` | Enable auto mode on Bedrock/Vertex/Foundry for Opus 4.7/4.8 (2.1.158) |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | Consecutive-Stop-block ceiling before the turn ends (default 8; 2.1.143) |
| `CLAUDE_CODE_SESSION_ID` | Session id, exposed to Bash subprocess and stdio MCP servers (2.1.132/2.1.154) |
| `CLAUDECODE=1` | Set in stdio MCP server environment to signal Claude Code (2.1.154) |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | Clone GitHub plugin sources over HTTPS instead of SSH (2.1.141) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Toggle the PowerShell tool (default-on Windows for Bedrock/Vertex/Foundry as of 2.1.143; set `0` to keep Git Bash) |
| `ANTHROPIC_WORKSPACE_ID` | Scope a federated token to a specific workspace (2.1.141) |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock service tier: `default`, `flex`, or `priority` (2.1.122) |
| `OTEL_LOG_TOOL_DETAILS=1` | Include tool parameters (bash commands, MCP/skill names) in telemetry (2.1.157) |

### Hook lifecycle events

| Event | When it fires |
|---|---|
| `SessionStart` (matchers: `startup`, `resume`, `clear`, `compact`) | Session begins or compacts |
| `SessionEnd` | Session ends cleanly |
| `UserPromptSubmit` | You press enter |
| `PreToolUse` | Before a tool call |
| `PostToolUse` | After a successful tool call |
| `PostToolUseFailure` | After a failed tool call |
| `PermissionRequest` | When Claude requests permission |
| `SubagentStart` | A subagent begins |
| `SubagentStop` | A subagent finishes |
| `Stop` | Claude finishes a response turn |
| `StopFailure` | Claude's turn errored |
| `Notification` (matchers: `permission_prompt`, `idle_prompt`, `auth_success`, etc.) | Notification triggered |
| `MessageDisplay` | As each assistant message is shown; can transform or hide text (2.1.152) |
| `InstructionsLoaded` | After all CLAUDE.md and rules merge into the system prompt |

---

## Appendix C. Frontmatter Field Reference

### Skill frontmatter (`SKILL.md`)

```yaml
---
name: <unique-name>                     # required
description: <when to invoke>           # required; up to 1,536 chars combined with when_to_use
when_to_use: <additional guidance>      # optional; helps with auto-routing
allowed-tools: Tool Tool Bash(pattern)  # space-separated; restricts what the skill can use
disallowed-tools: Bash WebFetch         # remove tools from the model while active (2.1.152)
disable-model-invocation: false         # if true, only /skill-name works (Claude won't auto-call)
user-invocable: true                    # if false, hidden from / menu but Claude can still call
context: fork                           # run in isolated subagent
agent: Explore                          # which agent profile when context: fork
argument-hint: <hint for arguments>     # shown in / menu
model: <override>                       # use a specific model just for this skill
effort: high                            # pin effort for this skill; body can read ${CLAUDE_EFFORT}
hooks: { ... }                          # skill-scoped hooks (rare)
---
```

### Agent frontmatter (`.claude/agents/<name>.md`)

```yaml
---
name: <unique-name>                     # required
description: <when to invoke>           # required; routes Claude to this agent
model: claude-opus-4-7                  # or sonnet, haiku, inherit, full model ID
effort: high                            # low, medium, high, xhigh (Opus 4.7/4.8), max (API-only)
tools:                                  # explicit allowlist
  - Read
  - Edit
  - Bash(pnpm test*)
disallowedTools:                        # explicit denylist (wins over tools)
  - WebFetch
  - Bash(git push*)
permissionMode: acceptEdits             # default, acceptEdits, plan, bypassPermissions
maxTurns: 40                            # cap on turns before forced stop
isolation: worktree                     # run in a fresh git worktree
background: false                       # run as a background task
memory: project                         # which memory scope to inherit
skills:                                 # skills to preload
  - spec-checker
mcpServers:                             # MCP servers to expose (project agents only)
  - github
  - postgres
hooks:                                  # agent-scoped hooks (project agents only)
  SubagentStop:
    - hooks:
        - type: command
          command: bash test_gate.sh
initialPrompt: <auto-submitted first turn>
color: <UI tag color>
---
```

### Rules frontmatter (`.claude/rules/<topic>.md`)

```yaml
---
paths:                                  # glob patterns; rule loads only when matching
  - "src/api/**/*.ts"
  - "src/handlers/**/*.ts"
---
```

### Plugin manifest (`.claude-plugin/plugin.json`)

```json
{
  "name": "internal-platform",          // required
  "description": "...",                  // required
  "version": "1.2.0",                   // semver
  "author": { "name": "Platform Team" },
  "homepage": "https://...",
  "repository": "https://..."
}
```

---

## Appendix D. The Companion Starter Kit

The book references a companion document called `claude-code-pipeline-starter.md`. It's a complete, ready-to-drop-in `.claude/` directory containing all ten agents from Part VI, the path-scoped rules, the skills, the full `settings.json` with hooks wired in, and the debug instrumentation from Chapter 36 (audit.sh, log_mcp_calls.sh, startup_check.py, the updated test_gate.sh that records test history).

Drop it into a project, fill in the credentials in `.env`, and you have the pipeline described in Part VI running. The starter kit is the implementation; this book is the explanation. Read this book to know *why* each piece is the way it is; use the starter kit to skip the typing.

The starter kit ships separately from this handbook. If you've received only one of the two files, ask for the other — they're designed to be used together.

---

## Appendix E. Further Reading

The official Claude Code documentation at **https://code.claude.com/docs** is the source of truth and changes frequently. The pages most worth bookmarking:

- `code.claude.com/docs/en/memory` — CLAUDE.md, rules, auto memory, compaction semantics
- `code.claude.com/docs/en/skills` — full skill spec including frontmatter fields
- `code.claude.com/docs/en/sub-agents` — agent definition reference
- `code.claude.com/docs/en/agent-teams` — the agent teams experimental feature
- `code.claude.com/docs/en/hooks` — every hook event with payload schemas
- `code.claude.com/docs/en/hooks-guide` — hooks tutorial and troubleshooting
- `code.claude.com/docs/en/mcp` — MCP configuration, scopes, Tool Search
- `code.claude.com/docs/en/plugins` — plugin format and marketplace
- `code.claude.com/docs/en/settings` — settings.json keys, precedence
- `code.claude.com/docs/en/debug-your-config` — the inspection commands in detail
- `code.claude.com/docs/en/troubleshooting` — installation and connectivity issues
- `code.claude.com/docs/en/env-vars` — every environment variable
- `code.claude.com/docs/en/sandboxing` — for stronger isolation than worktrees
- `code.claude.com/docs/en/headless` — programmatic and CI usage
- `code.claude.com/docs/en/scheduled-tasks` — running Claude on a schedule

For prompt-engineering and skill design beyond Claude Code itself, the **Anthropic engineering blog** (`anthropic.com/engineering`) publishes the most technically substantive material. The "Equipping agents for the real world with Agent Skills" and "Effective context engineering" posts are foundational. The **Claude Cookbook** GitHub repo (`github.com/anthropics/anthropic-cookbook`) has working code; the **Skills Cookbook** is its skill-specific counterpart. The **Memory Tool** and **Tool Search Tool** documentation on the Anthropic Platform site covers the API-side primitives that Claude Code is built on, useful when you start writing your own Claude agents from scratch with the SDK.

For the agent SDK specifically (if you want to build your own agentic systems on the same primitives as Claude Code), see `code.claude.com/docs/en/agent-sdk/overview`. The SDK exposes the same tool loop and context management — Claude Code is essentially the canonical client of the SDK.

The community has settled on a few common patterns worth knowing about: the **"business brain"** pattern (separating brand/project context from agent instructions), the **"agent-coordination skill"** approach (using a single shared skill as the canonical project memory for multi-agent setups), and the **plan-execute loop** (specs/ + prompt_plan.md + CLAUDE.md, which this book builds on). Articles documenting these patterns appear regularly on Medium and dev.to; the underlying mechanics are all explained in this handbook.

---

## Appendix F. Release Delta — Changes Since 2.1.132

This appendix catalogs the changes between Claude Code 2.1.132 and 2.1.158 (May 6–30, 2026) that affect what this handbook teaches. It's organized by the book's own structure so you can see, surface by surface, what moved. Bug fixes are omitted unless they change guidance; version numbers are in parentheses so you can check `claude --version` against them. The headline of the window is **Opus 4.8** (2.1.154), which now defaults to high effort, adds an `xhigh` step below `max`, and brings a 1M-token context window; and **dynamic workflows** (2.1.154), which raise the ceiling on parallel autonomous work from a handful of agents to hundreds.

### Models and effort

Opus 4.8 shipped in 2.1.154 and is the assumed model for this edition. It defaults to **high** effort, and `/effort xhigh` is available for the hardest tasks (the `xhigh` level sits between `high` and `max`, and was introduced for Opus 4.7 in 2.1.111). Fast mode on Opus 4.8 runs at 2× the standard rate for 2.5× the speed (2.1.154); fast mode's default backing model moved from 4.6 to 4.7 in 2.1.142, and the `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` knob is deprecated (removed 06/01/2026). The `/effort` slider's labels were renamed from "Speed/Intelligence" to "Faster/Smarter" (2.1.154). A lean system prompt is now the default for all models except Haiku, Sonnet, and Opus 4.7-and-earlier (2.1.154) — Opus 4.8 sessions start with less baseline context overhead, which is free headroom for your CLAUDE.md and skills.

### Autonomy (Chapters 16, 19, 20)

This is the most consequential area for a vibe-coding-at-scale pipeline. **Auto mode** matured from an opt-in experiment into a first-class permission posture: it no longer requires a flag or consent prompt (2.1.152), appears in the Shift+Tab cycle (2.1.143), and — critically for AWS Bedrock users — is now available on Bedrock, Vertex, and Foundry for Opus 4.7 and 4.8 via `CLAUDE_CODE_ENABLE_AUTO_MODE=1` (2.1.158). It's classifier-driven rather than blanket-approve, with `autoMode.allow` / `soft_deny` / `hard_deny` / `environment` rules (hard-deny added 2.1.136), and `"$defaults"` to extend rather than replace the built-in ruleset (2.1.118). The exfiltration classifier was hardened against bulk repo-content transfers in 2.1.154. See Chapter 19 for where it sits relative to `acceptEdits` and `bypassPermissions`.

**Dynamic workflows** (`/workflows`, 2.1.154) orchestrate tens to hundreds of background agents from a natural-language request — the right tool for massively parallel, uniform jobs that you'd never hand-wire as subagents. A `/config` "Workflow keyword trigger" toggle (2.1.157) controls whether the bare word "workflow" auto-offers one. **`/goal`** (2.1.139) sets a natural-language completion condition and keeps Claude working across turns until met, with a live overlay; it's the judgment-based cousin of the deterministic Stop-hook gate. Both are covered in Chapter 18.

The **agent view** (`claude agents`, Research Preview, 2.1.139) is a cross-session dashboard, with a steady stream of flags added through 2.1.157 (`--add-dir`, `--settings`, `--mcp-config`, `--plugin-dir`, `--permission-mode`, `--model`, `--effort`, `--dangerously-skip-permissions`, `--cwd`, `--json`, and `! <command>` background shells). Background sessions also got `/resume` support (2.1.144) and now preserve model, effort, and permission mode across idle/wake (2.1.141, 2.1.143).

### Hooks (Chapters 13, 14)

A genuinely new event: **`MessageDisplay`** (2.1.152) transforms or hides assistant text at display time — the clean path for output redaction. New configuration on existing events: exec-form **`args: string[]`** (2.1.139) spawns without a shell so paths never need quoting; **`continueOnBlock`** for PostToolUse (2.1.139) feeds a rejection reason back and continues the turn; **`terminalSequence`** in hook output (2.1.141) emits notifications/titles/bells from headless and background contexts. Hooks now receive **`effort.level`** and `$CLAUDE_EFFORT` (2.1.133), and Stop/SubagentStop input carries **`background_tasks`** and **`session_crons`** (2.1.145). SessionStart can return **`reloadSkills: true`** and set **`sessionTitle`** (2.1.152). Hooks can invoke MCP tools via **`type: "mcp_tool"`** (2.1.118) and rewrite output via **`updatedToolOutput`** for all tools (2.1.121); PostToolUse input gained **`duration_ms`** (2.1.119). The loop backstop: a Stop hook that blocks 8 times in a row now ends the turn with a warning, tunable via **`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`** (2.1.143) — defense-in-depth behind the `stop_hook_active` guard, not a replacement. Two constraints: prompt/agent-type hooks on `SessionStart`/`Setup`/`SubagentStart` are now rejected with a clear error (2.1.142), and hooks run without direct terminal access (2.1.139), so write to stderr.

### Skills (Chapters 7, 8, 12)

**`disallowed-tools`** frontmatter (2.1.152) removes tools from the model while a skill or command is active — the inverse of `allowed-tools`. **`/reload-skills`** and SessionStart `reloadSkills` (2.1.152) make new skills live without a restart. Skill bodies can read **`${CLAUDE_EFFORT}`** (2.1.120) and pin **`effort:`** frontmatter. **`skillOverrides`** (working 2.1.129) controls per-skill visibility (`off`, `user-invocable-only`, `name-only`). A reliability fix: subagents discover project/user/plugin skills via the Skill tool again as of 2.1.133, and a `context: fork` self-re-invocation loop was fixed in 2.1.145. The Read tool now returns a truncated "PARTIAL view" page instead of a hard error when a whole-file read exceeds the token limit (2.1.145) — relevant to skills that read large files.

### MCP (Chapter 11)

**`alwaysLoad`** (2.1.121) opts a server out of Tool Search deferral so all its tools are always present — use only for small, hot servers. Stdio MCP subprocesses now receive **`CLAUDE_PROJECT_DIR`** (2.1.139) and **`CLAUDE_CODE_SESSION_ID` + `CLAUDECODE=1`** (2.1.154), and plugin configs can interpolate `${CLAUDE_PROJECT_DIR}`. **`MCP_TOOL_TIMEOUT`** finally raises the per-request ceiling above 60s (fixed 2.1.142), and paginated `tools/list` responses are fully consumed (fixed 2.1.144). `/mcp` Reconnect picks up `.mcp.json` edits without a restart (2.1.139), `workspace` is a reserved server name (2.1.128), and `allowAllClaudeAiMcps` (2.1.149) loads claude.ai cloud connectors alongside managed MCP config. Note for API-key/Bedrock setups: Remote Control, `/schedule`, claude.ai connectors, and notifications are disabled when `ANTHROPIC_API_KEY` / `apiKeyHelper` / `ANTHROPIC_AUTH_TOKEN` is set (2.1.139); unset the key to use them.

### Plugins (Chapter 10)

Plugins in `.claude/skills` are now auto-loaded with no marketplace required (2.1.157), and `claude plugin init <name>` scaffolds one (2.1.157). Plugins can ship disabled with **`defaultEnabled: false`** in `plugin.json` (2.1.154). Dependency enforcement (2.1.143): `plugin disable` refuses when a dependent is enabled, `plugin enable` force-enables transitive deps, and `plugin prune` removes orphans (2.1.121). `claude plugin details` and the `/plugin` panes now show full component inventories and projected per-session token cost before install (2.1.139, 2.1.145). A plugin with a root-level `SKILL.md` and no `skills/` directory is surfaced as a skill (2.1.142). The `pluginSuggestionMarketplaces` managed setting (2.1.152) lets admins allowlist marketplaces for context-aware suggestions.

### Settings and worktrees (Chapters 15, 19)

New settings worth knowing: **`worktree.baseRef`** (`fresh` | `head`, 2.1.133) — default `fresh` branches agent worktrees from `origin/<default>`, so set `head` to include unpushed local commits; **`worktree.bgIsolation: "none"`** (2.1.143) lets background sessions edit the working copy directly; **`sandbox.network.deniedDomains`** (2.1.113) and `sandbox.bwrapPath`/`socatPath` (2.1.133); **`parentSettingsBehavior`** (2.1.133) and `prUrlTemplate` (2.1.119). `/config` settings now persist to `settings.json` and participate in scope precedence (2.1.119). On Windows, the PowerShell tool is default-on for Bedrock/Vertex/Foundry (2.1.143) — set `CLAUDE_CODE_USE_POWERSHELL_TOOL=0` to stay on Git Bash, which is the relevant choice for a Git-Bash-on-Windows workflow.

### Debugging and observability (Part VIII)

`/usage` now breaks down what's driving your limits by skills, subagents, plugins, and per-MCP-server cost (2.1.149) — a fast way to find the expensive layer in a big pipeline. Subagent API requests carry `x-claude-code-agent-id` / `parent_agent_id` headers and OTEL spans (2.1.139, 2.1.145), `tool_decision` telemetry includes `tool_parameters` under `OTEL_LOG_TOOL_DETAILS=1` (2.1.157), and status-line JSON now includes GitHub repo/PR info (2.1.145) plus `COLUMNS`/`LINES` (2.1.153). `claude doctor` reports the result of your last update attempt (2.1.153).

### Command renames to update muscle memory

`/simplify` → `/code-review` (2.1.147), then `/simplify` returned as a cleanup-only review (2.1.154); `/code-review` takes an effort level and `--fix` / `--comment`. `/cost` and `/stats` merged into `/usage` (2.1.118, both remain as shortcuts). `/extra-usage` → `/usage-credits` (2.1.144). `/model` now sets the default for new sessions (press `s` for current-session-only); the old `modelPicker:setAsDefault` keybinding is renamed `modelPicker:thisSessionOnly` (2.1.153).

---

## Appendix G. Release Delta — 2.1.159 to 2.1.204 (Deprecations, Removals, and New Capabilities)

This appendix covers everything from 2.1.159 (May 31) through 2.1.204 (July 8, 2026). It leads with what has been **deprecated and removed** — the changes most likely to break an existing setup — then the model landscape (Fable 5, Sonnet 5), then behavior changes that supersede earlier guidance, then new features by surface. As always, version numbers let you check `claude --version`.

### G.1 Deprecated and removed

These are the changes that can break a pipeline built against an older version. Read this section first.

| Item | Status | Version | What to do |
|---|---|---|---|
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **Removed** (now a no-op) | 2.1.160 | It was deprecated in 2.1.154 and is gone. For fast mode on an older Opus, switch model with `/model claude-opus-4-6[1m]` then `/fast on`. |
| `TeamCreate` / `TeamDelete` tools (Agent Teams) | **Removed** | 2.1.178 | Every session now has one implicit team; spawn teammates directly via the Agent tool's `name` parameter. The `team_name` parameter is accepted but ignored. |
| The `/agents` interactive wizard | **Removed** | 2.1.198 | Ask Claude to create or manage subagents in plain language, or edit `.claude/agents/*.md` directly. |
| Dynamic-workflow trigger keyword `workflow` | **Renamed** to `ultracode` | 2.1.160 | The bare word "workflow" no longer triggers a run; use `ultracode`, or ask in your own words. |
| Permission mode label "default" | **Renamed** to "Manual" | 2.1.200 | UI now says "Manual"; `--permission-mode manual` / `"defaultMode": "manual"` accepted alongside the still-valid `default`. |
| AskUserQuestion auto-continue on timeout | **Behavior removed** (no longer default) | 2.1.200 | Dialogs now wait instead of auto-continuing; opt into an idle timeout via `/config` for unattended runs. |
| Windsurf (IDE label) | **Renamed** to Devin Desktop | 2.1.162 | Cosmetic, in `/ide`, `/terminal-setup`, `/scroll-speed`. |
| `CLAUDE_CODE_MAX_RETRIES` unbounded value | **Capped at 15** | 2.1.186 | For unattended sessions use `CLAUDE_CODE_RETRY_WATCHDOG` (raises retries to 300 as of 2.1.199) instead. |
| Startup "setup issues" / "command missing or broken" lines | **Removed** from startup | 2.1.183, 2.1.204 | The information moved to `/doctor` and `/status`. |
| `/review <pr>` multi-agent behavior | **Reverted** to fast single-pass | 2.1.202 | Use `/code-review <level> <pr#>` for the multi-agent review at a chosen effort. |
| Left-arrow closing background/diff/workflow detail views | **Changed** to Esc | 2.1.204 | Press Esc to close those views; left-arrow no longer does. |
| JetBrains-plugin install suggestion at startup | **Removed** | 2.1.160 | Cosmetic. |

Two behavior *defaults* also flipped in ways worth flagging here, because they change what an unattended run does even though nothing was "removed": **subagents now run in the background by default** (2.1.198), and the **stream idle watchdog is on by default for all providers** (2.1.196, aborts and retries after 5 minutes of stream silence — disable with `CLAUDE_ENABLE_STREAM_WATCHDOG=0`). And one telemetry default to watch on upgrade: the new `claude_code.assistant_response` OTel event (2.1.193) is redacted unless `OTEL_LOG_ASSISTANT_RESPONSES=1`, but when that variable is *unset* it follows `OTEL_LOG_USER_PROMPTS` — so a deployment that already logs prompt content will silently start logging response content too. Set `OTEL_LOG_ASSISTANT_RESPONSES=0` to keep prompts-only.

### G.2 The model landscape: Fable 5 and Sonnet 5

**Claude Fable 5** (2.1.170, June 9) is a Mythos-class model made available for general use in Claude Code — update to 2.1.170 or later to select it. Two Fable-specific mechanics matter for anyone who pins models: Fable 5 ships with a **1M-token context window by default**, so a `[1m]` suffix on the model name is redundant and is now **stripped automatically** (2.1.173) — relevant if your `ANTHROPIC_DEFAULT_OPUS_MODEL`-style pinning appends `[1m]`, because the same normalization logic applies and a doubled `[1m][1m]` suffix was a real bug that's since been fixed. Auto mode on Fable 5 falls back to the best available Opus classifier for organizations that don't have Opus 4.8 enabled (2.1.176), so auto mode keeps working even where Fable is the session model.

**Claude Sonnet 5** (2.1.197, June 30) is now the **default model in Claude Code**, with a native 1M-token context window. If your workflow assumed Opus as the default, it no longer is — pin Opus explicitly with `/model`, an `ANTHROPIC_MODEL` env var, or agent frontmatter. (On third-party providers like Bedrock, model availability and the region-derived inference-profile prefix still govern what you actually get; `/status` shows where the region came from as of 2.1.172.)

Alongside the new models, org-level model governance matured: **`availableModels`** with **`enforceAvailableModels`** (2.1.175) lets admins constrain even the Default model and prevents user/project settings from widening a managed allowlist; **organization default models** (2.1.196) show as "Org default" / "Role default" in `/model`; and a **`fallbackModel`** setting (2.1.166) configures up to three fallback models tried in order on overload — the interactive-session complement to `--fallback-model`.

### G.3 Behavior changes that supersede earlier chapters

Beyond the renames in G.1, a few functional changes update guidance elsewhere in this book. **Stop and SubagentStop hooks can return `hookSpecificOutput.additionalContext`** (2.1.163) to give Claude feedback and continue the turn without raising a hook error — a softer alternative to exit 2, now noted in Chapter 14. **`!` bash commands trigger Claude to respond to their output automatically** (2.1.186); set `"respondToBashCommands": false` to keep the old context-only behavior. **Hook matchers with hyphenated identifiers now exact-match** rather than substring-match (2.1.195) — a matcher like `mcp__brave-search` no longer accidentally matches siblings, so use `mcp__brave-search__.*` to match all tools from a hyphenated server. **`acceptEdits` now prompts before writing execution-granting config files** (`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/`, shell startup files) (2.1.160), which slightly narrows how "hands-off" acceptEdits is — a good change for the trust model in Chapter 19.

### G.4 New capabilities by surface

**Debugging (Part VIII).** The big addition is `--safe-mode` / `CLAUDE_CODE_SAFE_MODE` (2.1.169) — a one-flag clean room that disables CLAUDE.md, plugins, skills, hooks, and MCP servers, now folded into Chapter 34. Also: `/config key=value` sets any setting from the prompt (2.1.181); `/config --help` lists the shorthand keys (2.1.183); `requiredMinimumVersion` / `requiredMaximumVersion` managed settings refuse to start outside an approved range (2.1.163); and `disableBundledSkills` / `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` hides built-ins from the model (2.1.169).

**Agents.** Subagents can spawn their own subagents up to 5 levels deep (2.1.172); they run in the background by default and auto-open draft PRs when finishing worktree code work (2.1.198); the built-in Explore agent inherits the session model capped at Opus rather than running on Haiku (2.1.198); and subagents inherit the session's extended-thinking configuration (2.1.198). New permission granularity: `Tool(param:value)` rules with wildcards, e.g. `Agent(model:opus)` to block Opus subagents (2.1.178), and `Agent(type)` deny / `Agent(x,y)` allowed-type restrictions on named spawns (2.1.186). Cross-session messaging was hardened so relayed `SendMessage` traffic never carries user authority (2.1.166, 2.1.183).

**Auto mode and security.** Destructive commands are now blocked by the classifier unless you asked for them — `git reset --hard`, `git checkout -- .`, `git clean -fd`, `git stash drop`, `git commit --amend` on commits the agent didn't make, and `terraform`/`pulumi`/`cdk destroy` (2.1.183). `autoMode.classifyAllShell` routes *all* Bash/PowerShell through the classifier rather than only code-execution patterns (2.1.193), denial reasons now appear in the transcript, toast, and `/permissions` (2.1.193), and subagent spawns are classifier-evaluated before launch (2.1.178). A `sandbox.credentials` setting blocks sandboxed commands from reading credential files and secret env vars (2.1.187).

**Hooks and skills.** Beyond `additionalContext` (G.3): a self-hosted-runner `post-session` lifecycle hook runs after the session ends and before workspace teardown (2.1.169); `SessionStart`/`Setup`/`SubagentStart` hooks now surface stderr on exit 2 instead of hiding it (2.1.199). For skills: stacked slash invocations like `/skill-a /skill-b …` load all leading skills up to 5 (2.1.199); nested `.claude/skills` load when you work on files there, disambiguated as `<dir>:<name>` on a clash (2.1.178); frontmatter keys accept kebab/snake/camelCase and a malformed `SKILL.md` loads its body with empty metadata rather than failing silently (2.1.186); and a `$` escape lets command bodies include a literal `$` before a digit (2.1.163). A bundled `/dataviz` skill was added (2.1.198).

**MCP.** `claude mcp login <name>` / `claude mcp logout <name>` authenticate servers from the CLI without the interactive `/mcp` menu, with `--no-browser` for SSH (2.1.186); remote MCP tool calls that hang now abort after 5 minutes (override `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT`) (2.1.187); the session's additional working directories are exposed via MCP `roots/list` with change notifications (2.1.203); `headersHelper` re-runs and reconnects on a 401/403 (2.1.193); and a config with `url` but no `type` now gets a clear "add `\"type\": \"http\"`" error instead of a misleading one (2.1.202).

**Other.** Claude in Chrome is generally available (2.1.198); `/cd` moves a session's working directory without breaking the prompt cache (2.1.169); `/rewind` can resume from before a `/clear` (2.1.191); `/plugin list` gained `--enabled`/`--disabled` filters (2.1.163); and `CLAUDE_CLIENT_PRESENCE_FILE` suppresses mobile push while you're at the machine (2.1.181).

---

*End of handbook.*
