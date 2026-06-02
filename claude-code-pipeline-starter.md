# Claude Code Autonomous Pipeline — Complete Starter Kit (v3, aligned to 2.1.158 / Opus 4.8)

A full software-development-lifecycle pipeline built on Claude Code agents, hooks, skills, and MCP servers, with debug instrumentation built in. Drop the file tree below into a project's `.claude/` directory, fill in the credentials, and you have a setup that takes a Jira ticket and turns it into a reviewed, tested, documented, deploy-ready change — with you in the loop only at the points where judgment is genuinely required, and with audit logs that explain every failure when it happens.

**What's new in v2:**
- `audit.sh` event log captures every SubagentStart, SubagentStop, Stop, and major lifecycle event with structured JSON
- `log_mcp_calls.sh` records every MCP server response for post-mortem debugging
- `startup_check.py` runs a self-check at session start and warns about common misconfigurations
- `test_gate.sh` now writes test history to `.claude/.test-history.jsonl` so the report-generator can include trends
- `settings.json` wires all instrumentation in cleanly
- A new section explaining how to read the logs

**What's new in v3 (aligned to Claude Code 2.1.158, May 2026):**
- Stop/SubagentStop gates note the runtime 8-block safety net (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) layered behind the `stop_hook_active` guard
- `worktree.baseRef: "head"` set in `settings.json` so worktree-isolated agents (bug-investigator, cve-remediator) see your unpushed commits
- `notify.sh` modernized to emit a `terminalSequence` so notifications work from background/headless sessions
- Optional auto-mode block (`CLAUDE_CODE_ENABLE_AUTO_MODE=1`) documented for Bedrock/Vertex/Foundry autonomy on Opus 4.7/4.8
- `startup_check.py` self-check updated for current versions and the later AskUserQuestion fixes

---

## File tree

```
your-project/
├── CLAUDE.md
├── .mcp.json
├── .gitignore                        # add .claude/audit.log, .claude/mcp.log, etc.
└── .claude/
    ├── settings.json
    ├── settings.local.json           # gitignored
    ├── agents/
    │   ├── spec-architect.md
    │   ├── spec-checker.md
    │   ├── implementer.md
    │   ├── bug-investigator.md
    │   ├── cve-remediator.md
    │   ├── security-reviewer.md
    │   ├── test-runner.md
    │   ├── doc-publisher.md
    │   ├── deployer.md
    │   └── report-generator.md
    ├── skills/
    │   ├── review-diff/SKILL.md
    │   └── triage-sentry/SKILL.md
    ├── hooks/
    │   ├── session_context.py
    │   ├── startup_check.py          # NEW
    │   ├── prompt_validator.py
    │   ├── guard_bash.sh
    │   ├── guard_protected_files.sh
    │   ├── format_changed.sh
    │   ├── test_gate.sh              # UPDATED — writes test history
    │   ├── report_gate.sh
    │   ├── auto_review_handoff.sh
    │   ├── audit.sh                  # NEW — event audit log
    │   ├── log_mcp_calls.sh          # NEW — MCP response log
    │   └── notify.sh
    └── rules/
        ├── api.md
        ├── migrations.md
        └── security.md
```

The new files are marked. The rest match v1.

---

## .gitignore additions

Add these lines so the log files don't pollute commits:

```gitignore
# Claude Code logs (debug instrumentation)
.claude/audit.log
.claude/mcp.log
.claude/.test-history.jsonl
.claude/.last-report-hash
.claude/settings.local.json
```

---

## CLAUDE.md (project root)

Keep this under 80 lines. Routing rules live here; procedures live in skills and agents.

```markdown
# Project: <name>

## Build & test
- Install: `pnpm install`
- Test: `pnpm test`
- Lint: `pnpm lint --fix`
- Type-check: `pnpm tsc --noEmit`

## Architecture (one-line per area)
- `src/api/` — REST handlers, one file per resource
- `src/services/` — pure business logic, no I/O
- `src/db/` — Drizzle ORM, migrations in `src/db/migrations/`
- `specs/` — numbered spec files
- `prompt_plan.md` — checklist driving systematic implementation

## Conventions
- Named exports only
- 2-space indentation
- All API endpoints validate input with Zod (see `.claude/rules/api.md`)
- Tests sit next to source as `*.test.ts`

## Agent routing (read every session)

- "Design", "spec out", "from this Jira ticket" → spec-architect (plan mode)
- "Implement", "build", "next step", "continue" → implementer (worktree)
- "Bug", "broken", "regression", "fails" → bug-investigator first, then implementer
- "CVE", "vulnerability", "security advisory" → cve-remediator
- "Review", "audit", "before merge" → security-reviewer (read-only)
- "Test", "run tests" → test-runner
- "Document", "publish to Confluence", "update docs" → doc-publisher
- "Deploy", "ship", "promote" → deployer (requires AskUserQuestion confirmation)
- End of session → report-generator (auto-fires via Stop hook)

When more than one applies in sequence, invoke in order. Each agent returns
a summary; pass relevant fields to the next agent.

## Hard rules
- Never call `deployer` without AskUserQuestion confirmation
- After `implementer` or `cve-remediator` finishes, the SubagentStop hook
  routes to `security-reviewer` automatically — don't bypass
- Use AskUserQuestion when ambiguity exists; don't guess
- Never commit unless an agent explicitly does so as part of its procedure
- Never modify files under `secrets/`, `.env*`, or `infra/prod/`

## Debug instrumentation
The audit log at `.claude/audit.log` records every agent and hook event.
The MCP log at `.claude/mcp.log` records every MCP server response.
Both are gitignored. To inspect a recent issue: `tail -50 .claude/audit.log`.
```

---

## .mcp.json (project root, committed)

```json
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v1/sse"
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    }
  }
}
```

OAuth happens on first use of each. Use `/mcp` inside Claude Code to complete the flow. For headless/CI, Atlassian supports API token auth — see Atlassian Rovo MCP docs and pin secrets to env vars referenced via `${VAR}` in this file.

**On the `worktree.baseRef: "head"` key (2.1.133+):** by default Claude Code branches isolation worktrees from `origin/<default>`, so the bug-investigator and cve-remediator agents would *not* see your unpushed local commits. Setting `head` branches from your local `HEAD` instead, which is what you want when you're asking an agent to investigate work you haven't pushed yet. Drop this key (or set `"fresh"`) if you specifically want agents to start from a clean pushed baseline.

---

## .claude/settings.json (project, committed) — updated with debug instrumentation

```json
{
  "autoMemoryEnabled": true,
  "skillListingBudgetFraction": 0.02,
  "worktree": {
    "baseRef": "head"
  },
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "python \"$CLAUDE_PROJECT_DIR/.claude/hooks/startup_check.py\"",
            "timeout": 10
          },
          {
            "type": "command",
            "command": "python \"$CLAUDE_PROJECT_DIR/.claude/hooks/session_context.py\"",
            "timeout": 10
          },
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" SessionStart",
            "async": true
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
          },
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" UserPromptSubmit",
            "async": true
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
      },
      {
        "matcher": "mcp__.*",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/log_mcp_calls.sh\"",
            "async": true
          }
        ]
      }
    ],
    "SubagentStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" SubagentStart",
            "async": true
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" SubagentStop",
            "async": true
          },
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/auto_review_handoff.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" Stop",
            "async": true
          },
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
    "Notification": [
      {
        "matcher": "permission_prompt|user_question|elicitation_dialog",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/notify.sh\"",
            "async": true
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/audit.sh\" SessionEnd",
            "async": true
          }
        ]
      }
    ]
  }
}
```

All audit hooks run `async: true` so they never block the agent loop. The guard and test hooks stay synchronous because their exit codes need to gate actions.

---

## Debug instrumentation hooks

### .claude/hooks/audit.sh (NEW)

The universal event log. Wired into every lifecycle event in settings.json. Every record is one JSON line so `jq` can slice it.

```bash
#!/usr/bin/env bash
# Universal event audit log. Captures every Claude Code lifecycle event
# as a structured JSON line in .claude/audit.log. Use this as your
# first inspection target when debugging "did X actually happen?"
#
# Usage (from settings.json):
#   "command": "bash .claude/hooks/audit.sh <EventName>"
#
# Each event reads its JSON payload from stdin and writes a single
# JSON line to the audit log with timestamp + event + payload.

set -euo pipefail

EVENT="${1:-Unknown}"
LOG="${CLAUDE_PROJECT_DIR:-$(pwd)}/.claude/audit.log"
mkdir -p "$(dirname "$LOG")"

INPUT=$(cat || echo "{}")

# Extract a few high-signal fields up-front for easy grep'ing
AGENT=$(echo "$INPUT" | jq -r '.agent_type // .agent // empty' 2>/dev/null || echo "")
TOOL=$(echo "$INPUT" | jq -r '.tool_name // empty' 2>/dev/null || echo "")
STOP_ACTIVE=$(echo "$INPUT" | jq -r '.stop_hook_active // false' 2>/dev/null || echo "false")
SESSION=$(echo "$INPUT" | jq -r '.session_id // empty' 2>/dev/null || echo "")

# Compose one structured line
jq -nc \
  --arg ts "$(date -Iseconds)" \
  --arg ev "$EVENT" \
  --arg agent "$AGENT" \
  --arg tool "$TOOL" \
  --arg session "$SESSION" \
  --argjson stop_active "$STOP_ACTIVE" \
  --argjson payload "$INPUT" \
  '{
    timestamp: $ts,
    event: $ev,
    session: $session,
    agent: $agent,
    tool: $tool,
    stop_hook_active: $stop_active,
    payload: $payload
  }' >> "$LOG"

exit 0
```

Make it executable: `chmod +x .claude/hooks/audit.sh`.

**Reading the log:**
```bash
# All events in this session, latest first
tail -100 .claude/audit.log | jq -c

# Just the agent invocations
jq -c 'select(.event == "SubagentStart" or .event == "SubagentStop")' .claude/audit.log

# Did the implementer agent actually run today?
jq -c 'select(.agent == "implementer")' .claude/audit.log

# Stop hook fires that DIDN'T have the loop guard set — these are the
# ones that forced continuation
jq -c 'select(.event == "Stop" and .stop_hook_active == false)' .claude/audit.log

# Count events by type for the day
jq -r '.event' .claude/audit.log | sort | uniq -c
```

### .claude/hooks/log_mcp_calls.sh (NEW)

Records every MCP server response so you can post-mortem "Atlassian returned weird data" or "the Jira transition went to the wrong state." Wired as a PostToolUse hook with matcher `mcp__.*`.

```bash
#!/usr/bin/env bash
# Logs every MCP tool call's response for post-mortem debugging.
# Wired as PostToolUse with matcher "mcp__.*" — fires for any MCP tool.

set -euo pipefail

LOG="${CLAUDE_PROJECT_DIR:-$(pwd)}/.claude/mcp.log"
mkdir -p "$(dirname "$LOG")"

INPUT=$(cat || echo "{}")

TOOL=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
SERVER=$(echo "$TOOL" | sed -E 's/^mcp__([^_]+)__.*/\1/')
RESPONSE=$(echo "$INPUT" | jq -c '.tool_response // {}')
INPUT_PARAMS=$(echo "$INPUT" | jq -c '.tool_input // {}')

# Truncate very large responses to keep log readable (full thing stored elsewhere if needed)
RESPONSE_SIZE=$(echo -n "$RESPONSE" | wc -c)
if [ "$RESPONSE_SIZE" -gt 4000 ]; then
  RESPONSE=$(echo "$RESPONSE" | head -c 4000)
  RESPONSE="${RESPONSE}...<truncated, ${RESPONSE_SIZE} bytes total>"
fi

jq -nc \
  --arg ts "$(date -Iseconds)" \
  --arg tool "$TOOL" \
  --arg server "$SERVER" \
  --argjson params "$INPUT_PARAMS" \
  --arg resp "$RESPONSE" \
  '{
    timestamp: $ts,
    server: $server,
    tool: $tool,
    params: $params,
    response: $resp
  }' >> "$LOG"

exit 0
```

**Reading the log:**
```bash
# All Atlassian calls in this session
jq -c 'select(.server == "atlassian")' .claude/mcp.log

# What did the last Jira issue lookup return?
jq -c 'select(.tool | contains("getJiraIssue"))' .claude/mcp.log | tail -1

# Did a Confluence update succeed?
jq -c 'select(.tool | contains("ConfluencePage"))' .claude/mcp.log | tail -5
```

### .claude/hooks/startup_check.py (NEW)

Runs at SessionStart, validates the configuration, and injects warnings into Claude's context if anything looks off. Catches misconfigurations *before* they cause silent failures during a real run.

```python
#!/usr/bin/env python3
"""Startup self-check. Validates that the project's .claude/ setup is
internally consistent and that no obvious misconfiguration is present.
Warnings are injected into the session context via SessionStart's stdout."""
import json
import os
import sys
from pathlib import Path


def main() -> None:
    project_dir = Path(os.environ.get("CLAUDE_PROJECT_DIR", os.getcwd()))
    warnings: list[str] = []
    info: list[str] = []

    claude_dir = project_dir / ".claude"
    settings_file = claude_dir / "settings.json"

    # --- Configuration sanity ---
    if not settings_file.is_file():
        warnings.append(f"No .claude/settings.json found at {settings_file}")
    else:
        try:
            settings = json.loads(settings_file.read_text())
        except json.JSONDecodeError as e:
            warnings.append(f"settings.json is invalid JSON: {e}")
            settings = {}

        hooks = settings.get("hooks", {})

        # Verify protected-files guard is present
        pre_tool = hooks.get("PreToolUse", [])
        has_protect = any(
            "guard_protected_files" in str(h) for h in pre_tool
        )
        if not has_protect:
            warnings.append(
                "Missing PreToolUse hook: guard_protected_files. "
                "Without it, Claude may modify .env, secrets/, or infra/prod/."
            )

        # Verify bash guard is present
        has_bash_guard = any(
            "guard_bash" in str(h) for h in pre_tool
        )
        if not has_bash_guard:
            warnings.append(
                "Missing PreToolUse Bash guard. Destructive commands aren't blocked."
            )

        # Verify Stop hook has stop_hook_active guards
        stop_hooks = hooks.get("Stop", [])
        for entry in stop_hooks:
            for h in entry.get("hooks", []):
                cmd = h.get("command", "")
                if "test_gate" in cmd or "report_gate" in cmd:
                    # we trust these have the guard; check the file exists
                    pass

    # --- Hook file sanity ---
    hooks_dir = claude_dir / "hooks"
    if hooks_dir.is_dir():
        for hook_file in hooks_dir.glob("*.sh"):
            if not os.access(hook_file, os.X_OK):
                warnings.append(
                    f"Hook not executable: {hook_file.name} "
                    "(run `chmod +x .claude/hooks/*.sh`)"
                )

    # --- Secrets sanity ---
    gitignore = project_dir / ".gitignore"
    if gitignore.is_file():
        gi = gitignore.read_text()
        critical = [".env", ".claude/audit.log", ".claude/mcp.log",
                    ".claude/.test-history.jsonl"]
        missing = [p for p in critical if p not in gi]
        if missing:
            warnings.append(
                f".gitignore missing entries for: {', '.join(missing)}"
            )

    # --- Version sanity ---
    # Parse the CLI version and flag known-relevant thresholds. This is the
    # spot to keep current as Claude Code moves; thresholds below are the ones
    # that affect this pipeline's behavior.
    try:
        import re
        import subprocess
        raw = subprocess.check_output(["claude", "--version"], text=True).strip()
        info.append(f"Claude Code version: {raw}")

        m = re.search(r"(\d+)\.(\d+)\.(\d+)", raw)
        if m:
            ver = tuple(int(x) for x in m.groups())

            # 2.1.104: AskUserQuestion regression with plugin skills +
            # bypassPermissions. The multi-select-array fix landed in 2.1.136
            # and the auto-mode-suppression fix in 2.1.147.
            if ver == (2, 1, 104):
                warnings.append(
                    "Claude Code 2.1.104 has the AskUserQuestion regression "
                    "(plugin skills + bypassPermissions). Upgrade past 2.1.147."
                )

            # This kit assumes the hook/skill/MCP features documented for the
            # 2.1.139–2.1.158 window (MessageDisplay, reloadSkills, worktree
            # baseRef, Stop-hook block cap, auto mode on Bedrock, etc.).
            if ver < (2, 1, 147):
                warnings.append(
                    f"Claude Code {raw} predates several features this kit "
                    "relies on (AskUserQuestion fixes, worktree.baseRef, the "
                    "Stop-hook block cap). Upgrade to 2.1.147+ (ideally 2.1.158+)."
                )
            elif ver < (2, 1, 154):
                info.append(
                    "On 2.1.154+ you also get Opus 4.8 defaults and dynamic "
                    "workflows (/workflows) — consider upgrading."
                )
    except Exception:
        pass  # Version check is best-effort

    # --- Auto mode awareness (Bedrock/Vertex/Foundry, 2.1.158+) ---
    # Surface whether unattended auto mode is actually enabled, so an overnight
    # run doesn't silently fall back to interactive permission prompts.
    if os.environ.get("CLAUDE_CODE_ENABLE_AUTO_MODE") == "1":
        info.append("Auto mode is enabled (CLAUDE_CODE_ENABLE_AUTO_MODE=1).")
    else:
        info.append(
            "Auto mode is OFF. For unattended Bedrock/Vertex/Foundry runs on "
            "Opus 4.7/4.8, set CLAUDE_CODE_ENABLE_AUTO_MODE=1."
        )

    # --- Quick stats for context ---
    agents_dir = claude_dir / "agents"
    skills_dir = claude_dir / "skills"
    if agents_dir.is_dir():
        info.append(f"Agents loaded: {len(list(agents_dir.glob('*.md')))}")
    if skills_dir.is_dir():
        info.append(f"Skills available: {len(list(skills_dir.glob('*/SKILL.md')))}")

    # --- Emit ---
    output_lines = []
    if warnings:
        output_lines.append("## STARTUP CHECK — WARNINGS")
        for w in warnings:
            output_lines.append(f"- ⚠ {w}")
    if info:
        output_lines.append("\n## STARTUP CHECK — INFO")
        for i in info:
            output_lines.append(f"- {i}")

    if output_lines:
        context = "\n".join(output_lines)
        print(json.dumps({
            "hookSpecificOutput": {
                "hookEventName": "SessionStart",
                "additionalContext": context
            }
        }))


if __name__ == "__main__":
    main()
```

The warnings show up in Claude's session context immediately, so any setup issue is visible on session start, not three turns in when something silently misbehaves.

### .claude/hooks/test_gate.sh (UPDATED — now records test history)

```bash
#!/usr/bin/env bash
# Quality gate on Stop and SubagentStop. Forces continuation with feedback
# if tests fail. Uses stop_hook_active guard to prevent infinite loops.
# v2: records test history to .claude/.test-history.jsonl so report-generator
# can include trends.

set -euo pipefail

INPUT=$(cat)

# CRITICAL: anti-loop guard. Always check this first.
# (As of 2.1.143 the runtime also caps consecutive Stop-hook blocks at 8 —
#  tunable via CLAUDE_CODE_STOP_HOOK_BLOCK_CAP — but that's a backstop, not a
#  substitute. A gate that ever blocks 8x in a row is still a bug to fix here.)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi

# Only enforce if there are uncommitted changes (otherwise nothing to test)
if [ -z "$(git status --porcelain 2>/dev/null || true)" ]; then
  exit 0
fi

HISTORY="${CLAUDE_PROJECT_DIR:-$(pwd)}/.claude/.test-history.jsonl"
mkdir -p "$(dirname "$HISTORY")"

TS=$(date -Iseconds)
OUTPUT_FILE=$(mktemp)
RESULT="pass"
EXIT_CODE=0

if ! pnpm test --silent > "$OUTPUT_FILE" 2>&1; then
  RESULT="fail"
  EXIT_CODE=2
fi

# Always record the run, pass or fail
TAIL=$(tail -20 "$OUTPUT_FILE" | jq -R -s '.')
jq -nc \
  --arg ts "$TS" \
  --arg result "$RESULT" \
  --argjson tail "$TAIL" \
  '{timestamp: $ts, result: $result, tail: $tail}' \
  >> "$HISTORY"

if [ "$RESULT" = "fail" ]; then
  echo "Tests failing — fix before declaring done:" >&2
  tail -20 "$OUTPUT_FILE" >&2
fi

rm -f "$OUTPUT_FILE"
exit "$EXIT_CODE"
```

The `report-generator` agent (below) reads `.claude/.test-history.jsonl` and includes a "tests went red 2x, then green" trend in the report.

---

## Other hooks (unchanged from v1)

### .claude/hooks/session_context.py

```python
#!/usr/bin/env python3
"""Injects current state into Claude's context at session start, resume, and post-compact."""
import json
import subprocess
from pathlib import Path


def run(cmd: str) -> str:
    try:
        return subprocess.check_output(
            cmd, shell=True, text=True, stderr=subprocess.DEVNULL
        ).strip()
    except subprocess.CalledProcessError:
        return ""


def main() -> None:
    branch = run("git rev-parse --abbrev-ref HEAD") or "(detached)"
    uncommitted = run("git status --porcelain | wc -l").strip() or "0"
    last_commit = run("git log -1 --pretty=format:'%h %s'")

    specs_dir = Path("specs")
    latest_spec = ""
    if specs_dir.is_dir():
        files = sorted(specs_dir.glob("*.md"), reverse=True)
        if files:
            latest_spec = files[0].name

    plan = Path("prompt_plan.md")
    next_step = "(no plan file)"
    if plan.is_file():
        for line in plan.read_text().splitlines():
            if line.startswith("- [ ]"):
                next_step = line.strip()
                break

    adr_dir = Path("docs/adrs")
    recent_adrs: list[str] = []
    if adr_dir.is_dir():
        for f in sorted(adr_dir.glob("*.md"), reverse=True)[:3]:
            recent_adrs.append(f.name)

    context = f"""## Session context (auto-injected)

Branch: {branch}
Uncommitted changes: {uncommitted} files
Last commit: {last_commit}
Latest spec: {latest_spec}
Next plan step: {next_step}
Recent ADRs: {', '.join(recent_adrs) or '(none)'}
"""
    print(json.dumps({
        "hookSpecificOutput": {
            "hookEventName": "SessionStart",
            "additionalContext": context
        }
    }))


if __name__ == "__main__":
    main()
```

### .claude/hooks/prompt_validator.py

```python
#!/usr/bin/env python3
"""Validates user prompts and injects warnings if needed."""
import json
import sys


def main() -> None:
    data = json.load(sys.stdin)
    prompt = data.get("prompt", "")
    additional = ""

    if "deploy" in prompt.lower() and "production" in prompt.lower():
        additional = (
            "REMINDER: production deploys require explicit double-confirmation. "
            "The deployer agent will ask twice."
        )

    if additional:
        print(json.dumps({
            "hookSpecificOutput": {
                "hookEventName": "UserPromptSubmit",
                "additionalContext": additional
            }
        }))


if __name__ == "__main__":
    main()
```

### .claude/hooks/guard_bash.sh

```bash
#!/usr/bin/env bash
# Blocks destructive Bash commands.
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$CMD" ]; then
  exit 0
fi

DENY_PATTERNS=(
  'rm -rf /'
  'rm -rf \*'
  'rm -rf ~'
  'rm -rf \$HOME'
  'mkfs'
  'dd if=.* of=/dev/'
  '> /dev/sd'
  ':\(\)\{ :|:'
  'curl .* \| sh'
  'wget .* \| sh'
  'sudo rm'
  'chmod -R 777 /'
)

for pattern in "${DENY_PATTERNS[@]}"; do
  if echo "$CMD" | grep -Eq "$pattern"; then
    echo "[guard_bash] blocked dangerous command: $CMD" >&2
    exit 2
  fi
done

exit 0
```

### .claude/hooks/guard_protected_files.sh

```bash
#!/usr/bin/env bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // .tool_input.path // empty')

if [ -z "$FILE" ]; then
  exit 0
fi

PROTECTED_PATTERNS=(
  '^secrets/'
  '^\.env'
  '/\.env$'
  '/\.env\.'
  '^infra/prod/'
  '\.pem$'
  '\.key$'
  'id_rsa'
)

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if echo "$FILE" | grep -Eq "$pattern"; then
    echo "[guard_protected_files] blocked write to protected path: $FILE" >&2
    echo "If this is intentional, edit it yourself outside Claude Code." >&2
    exit 2
  fi
done

exit 0
```

### .claude/hooks/format_changed.sh

```bash
#!/usr/bin/env bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // .tool_input.path // empty')

if [ -z "$FILE" ] || [ ! -f "$FILE" ]; then
  exit 0
fi

case "$FILE" in
  *.ts|*.tsx|*.js|*.jsx|*.json|*.md)
    npx --no-install prettier --write "$FILE" 2>/dev/null || true
    ;;
  *.py)
    ruff format "$FILE" 2>/dev/null || black "$FILE" 2>/dev/null || true
    ;;
  *.go)
    gofmt -w "$FILE" 2>/dev/null || true
    ;;
esac

exit 0
```

### .claude/hooks/report_gate.sh

```bash
#!/usr/bin/env bash
INPUT=$(cat)

if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi

PLAN="$CLAUDE_PROJECT_DIR/prompt_plan.md"
SENTINEL="$CLAUDE_PROJECT_DIR/.claude/.last-report-hash"

if [ ! -f "$PLAN" ]; then
  exit 0
fi

if grep -q '^- \[ \]' "$PLAN"; then
  exit 0
fi

CURRENT_HASH=$(sha256sum "$PLAN" | cut -d' ' -f1)
LAST_HASH=$(cat "$SENTINEL" 2>/dev/null || echo "")

if [ "$CURRENT_HASH" = "$LAST_HASH" ]; then
  exit 0
fi

echo "$CURRENT_HASH" > "$SENTINEL"

echo "Plan complete. Invoke the report-generator agent to summarize this session." >&2
exit 2
```

### .claude/hooks/auto_review_handoff.sh

```bash
#!/usr/bin/env bash
INPUT=$(cat)

if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0
fi

AGENT=$(echo "$INPUT" | jq -r '.agent_type // empty')

case "$AGENT" in
  implementer|cve-remediator)
    echo "Implementation complete. Hand off to the security-reviewer agent to review the diff before any further action." >&2
    exit 2
    ;;
esac

exit 0
```

### .claude/hooks/notify.sh

```bash
#!/usr/bin/env bash
# OS notification when Claude needs input.
#
# v3 (2.1.141+): the preferred path is to return a `terminalSequence` in the
# hook's JSON output. Claude Code emits it for us — a bell + window-title
# update + OSC desktop notification — and crucially this works from
# background and headless sessions where the shell-out fallbacks below run
# without a controlling terminal and silently do nothing. We keep the
# shell-outs as a fallback for environments/terminals that don't surface the
# OSC sequence, but the terminalSequence is what makes notifications reliable
# in `claude agents` / `--bg` runs.

set -euo pipefail

TITLE="Claude Code"
MSG="Input needed"

# BEL (audible) + OSC 9 desktop notification + OSC 0 window-title update.
# \u0007 = BEL, \u001b]9;...\u0007 = notification, \u001b]0;...\u0007 = title.
TERMSEQ=$(printf '\007\033]9;%s: %s\007\033]0;%s — %s\007' "$TITLE" "$MSG" "$TITLE" "$MSG")

# Emit the sequence for Claude Code to render (works headless/background).
jq -nc --arg seq "$TERMSEQ" '{terminalSequence: $seq}'

# Best-effort native fallbacks (no-ops when detached from a desktop session).
if command -v powershell.exe >/dev/null 2>&1; then
  powershell.exe -NoProfile -Command "New-BurntToastNotification -Text '$TITLE', '$MSG'" 2>/dev/null &
fi

if command -v terminal-notifier >/dev/null 2>&1; then
  terminal-notifier -title "$TITLE" -message "$MSG" -sound Ping 2>/dev/null &
fi

if command -v notify-send >/dev/null 2>&1; then
  notify-send "$TITLE" "$MSG" 2>/dev/null &
fi

exit 0
```

Because this hook now prints JSON to stdout, keep it on the `Notification` event (where stdout isn't injected into Claude's context). The `terminalSequence` field is consumed by Claude Code directly, so it reaches you even when the session is backgrounded and the `notify-send`/`terminal-notifier` shell-outs have no desktop to talk to.

---

## Agent definitions

Same as v1, except `report-generator` now reads the test history file. Updated body shown; other agents (spec-architect, spec-checker, implementer, bug-investigator, cve-remediator, security-reviewer, test-runner, doc-publisher, deployer) are unchanged from v1 — see the original starter kit for those.

### .claude/agents/report-generator.md (UPDATED)

```markdown
---
name: report-generator
description: Generates the end-of-session report. Auto-fires via the Stop hook when prompt_plan.md is complete (hash-sentinel idempotency prevents re-fire). Summarizes session work including test history trends, posts to Confluence, commits the report.
model: claude-sonnet-4-6
effort: medium
permissionMode: acceptEdits
maxTurns: 15
tools:
  - Read
  - Write
  - Bash(git log*)
  - Bash(git diff*)
  - Bash(date*)
  - Bash(jq *)
  - mcp__atlassian__createConfluencePage
  - mcp__atlassian__addJiraComment
---

You generate the end-of-session report.

## Procedure

1. Collect data:
   - All commits since session start (`git log --since`)
   - All files touched (`git diff --name-only`)
   - All plan items completed in this session (diff `prompt_plan.md`)
   - Test history from `.claude/.test-history.jsonl` — count fail→pass cycles,
     identify any tests that took multiple attempts
   - Audit log highlights from `.claude/audit.log` — agent invocations,
     any Stop hook exits with code 2 (forced continuations)
2. Group by Jira ticket. For each:
   - Spec implemented
   - Files changed
   - Tests added; flag any that initially failed
   - Decisions made (parse from commit bodies and agent outputs)
3. Write the report to `reports/<YYYY-MM-DD>-<branch>.md`.
4. Publish to the team Confluence space as a new page.
5. For each Jira ticket touched, add a comment linking to the Confluence page.

## Report template

```
# Session Report: <branch>
Date: <ISO>
Duration: <hours>

## Summary
<one paragraph: what got done>

## Per ticket
### <JIRA-KEY>: <title>
- Spec: SPEC-NNNN
- Files: <list>
- Tests: <count added / count modified>
- Test cycle: <e.g. "1 fail, then pass" or "green first try">
- Commits: <list of shas>
- Decisions:
  - <decision 1>

## Session metrics
- Agents invoked: <count, by name>
- Forced continuations (Stop hook exit 2): <count>
- MCP calls: <count, by server>
- Duration: <hours>

## What didn't go to plan
- <anything that diverged from the spec, with reasoning>

## Open items for next session
- <anything started but not finished>
```

## Output

```
REPORT: reports/<YYYY-MM-DD>-<branch>.md
CONFLUENCE: <page-url>
JIRA COMMENTS: <list of ticket:comment-id>
TICKETS: <count>
COMMITS: <count>
FILES: <count>
TEST RUNS: <pass count / fail count>
```
```

---

## How to read the logs

This is the operational manual for the debug instrumentation. Treat it as the answer to "something just went wrong — where do I start?"

### When an agent didn't fire that should have

```bash
# Was the SubagentStart event recorded?
jq -c 'select(.event == "SubagentStart")' .claude/audit.log | tail -10

# If not, was anything dispatched in the timeframe?
jq -c 'select(.timestamp > "2026-05-15T14:00:00")' .claude/audit.log
```

If `SubagentStart` is missing, the main session never delegated. Cause is usually a description mismatch — re-read your CLAUDE.md routing rules and the agent's `description` field.

### When a hook fired but produced wrong output

```bash
# Stop hooks that exited with code 2 (forced continuation)
jq -c 'select(.event == "Stop" and .stop_hook_active == false)' .claude/audit.log
```

Cross-reference with `claude --debug hooks` for the actual stderr of the hook script. The audit log shows *that* it fired; the debug log shows *what* it said.

### When an MCP call returned the wrong data

```bash
# Last 10 calls to Atlassian
jq -c 'select(.server == "atlassian")' .claude/mcp.log | tail -10

# Did the Jira ticket transition succeed?
jq -c 'select(.tool | contains("transition"))' .claude/mcp.log | tail -3
```

The response field shows exactly what the server returned. If it's an error or unexpected payload, you've found your root cause.

### When tests went red repeatedly

```bash
# Test runs from the last hour
jq -c 'select(.timestamp > "'"$(date -Iseconds -d '1 hour ago')"'")' \
  .claude/.test-history.jsonl

# Just the failures
jq -c 'select(.result == "fail")' .claude/.test-history.jsonl | tail -5
```

The `tail` field has the last 20 lines of test output so you can see the assertion message without re-running anything.

### When a session-start config issue isn't obvious

The `startup_check.py` warnings are injected into the session context, but if you missed them in the chat, they're also visible in:

```bash
jq -c 'select(.event == "SessionStart" and .payload.hookSpecificOutput.additionalContext | contains("WARNINGS"))' \
  .claude/audit.log
```

### Log rotation

The audit log grows. Add a daily rotation if you run heavily:

```bash
# .claude/hooks/rotate_logs.sh — wire into SessionEnd
DATE=$(date +%Y%m%d)
LOG_DIR="$CLAUDE_PROJECT_DIR/.claude"
for f in audit.log mcp.log .test-history.jsonl; do
  [ -f "$LOG_DIR/$f" ] && [ "$(wc -c < "$LOG_DIR/$f")" -gt 10485760 ] && \
    mv "$LOG_DIR/$f" "$LOG_DIR/${f}.${DATE}" && touch "$LOG_DIR/$f"
done
```

Or skip rotation and just `truncate -s 0 .claude/audit.log` when you start a new project phase.

---

## Path-scoped rules (unchanged)

### .claude/rules/api.md

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
- Status codes follow RFC convention; never invent 4xx codes
- No business logic in handlers; delegate to `src/services/`
```

### .claude/rules/migrations.md

```markdown
---
paths:
  - "src/db/migrations/**"
---

# Migration Rules

- One concern per migration; do not combine schema and data changes
- All migrations must be reversible
- Use `up()` and `down()` exports; never inline SQL strings
- Add an index for any new foreign key
- Test the down migration locally before submitting
```

### .claude/rules/security.md

```markdown
---
paths:
  - "src/auth/**"
  - "src/middleware/**"
  - "src/api/users/**"
---

# Security-Sensitive Code

- Any auth/authz change requires the security-reviewer agent before merge
- Never log raw request bodies for auth endpoints
- Never log tokens, even truncated
- Bcrypt cost factor must not drop below 12
- Session cookies: HttpOnly, Secure, SameSite=Strict
- Every new endpoint must have a permission decorator
```

---

## Lightweight skills (unchanged)

### .claude/skills/review-diff/SKILL.md

```markdown
---
name: review-diff
description: Lightweight pre-commit diff review. Use when the user wants a quick check of the staged diff before committing. Lighter than the security-reviewer agent — runs inline, not as a subagent.
allowed-tools: Bash(git diff *) Read
---

## Current diff
!`git diff --staged`

## Changed files
!`git diff --staged --name-only`

## Task

Review the diff above. Flag:
- Missing error handling on new code paths
- Hardcoded values that should be config
- New code without tests
- Breaking API changes
- Commented-out code
- Console.log / println debug statements

Output a short bulleted list grouped by severity. End with a one-line
recommendation: "OK to commit" or "Fix issues before committing".
```

### .claude/skills/triage-sentry/SKILL.md

```markdown
---
name: triage-sentry
description: Triages a Sentry error by pulling the issue, finding responsible code, and opening a Jira ticket. Use when the user mentions a Sentry error URL or issue ID.
allowed-tools: mcp__sentry__get_issue mcp__sentry__list_events mcp__atlassian__createJiraIssue Read Grep Glob
context: fork
agent: Explore
---

Investigate Sentry issue $ARGUMENTS:

1. Call `mcp__sentry__get_issue` for the stack trace and frequency.
2. Identify the source file from the top frame.
3. Read that file. Use Grep to find recent changes to that function.
4. Form a hypothesis: regression / environmental / edge case / unhandled input.
5. Call `mcp__atlassian__createJiraIssue` with:
   - Project: PLATFORM
   - Title: "Triage: <error summary>"
   - Body: stack trace, hypothesis, fix location, link to Sentry
   - Labels: ["bug", "triage", "claude-generated"]
   - Do NOT assign; leave for human review.
6. Return the new Jira issue key to the main session.
```

---

## Quickstart

1. **Drop the file tree** into your project. The agent definitions (spec-architect, spec-checker, implementer, bug-investigator, cve-remediator, security-reviewer, test-runner, doc-publisher, deployer) are in the v1 starter kit; copy them over.
2. **Make hooks executable** (one-time):
   ```bash
   chmod +x .claude/hooks/*.sh .claude/hooks/*.py
   ```
   On Windows + Git Bash, check that files have LF line endings (not CRLF) so shebangs work.
3. **Authenticate MCP servers.** Start Claude Code, run `/mcp`, complete OAuth for Atlassian, GitHub, Sentry.
4. **Verify everything loaded.** Run in order:
   ```
   /context     # confirm CLAUDE.md, rules, skills, agents all present
   /agents      # confirm all 10 agents listed
   /hooks       # confirm all hooks registered
   /mcp         # confirm all servers connected
   /doctor      # surface any schema errors
   ```
5. **Verify instrumentation.** Trigger any action and check:
   ```bash
   tail -5 .claude/audit.log | jq
   ```
   You should see at least a SessionStart and UserPromptSubmit entry.
6. **First run:** `Look at PLATFORM-1247 and walk it end-to-end.` Watch `tail -f .claude/audit.log` in a second terminal.

---

## Extending the instrumentation

The four things teams add next:

**Per-agent timing.** Modify `audit.sh` to record duration between SubagentStart and SubagentStop for the same agent. Lets you find which agent is slowing down the pipeline.

**Slack/Discord notifications on Stop forced-continuation.** Wire a hook to ping a channel when a Stop hook exits 2 — useful for shared workflows where someone other than you might be waiting.

**Cost tracking.** Tag the audit log with the model used per agent invocation; aggregate at SessionEnd. Helps with Bedrock cost attribution.

**Pre-flight CI gate.** Use the audit log in CI to verify a session followed the expected agent sequence before allowing merge. If `report-generator` didn't run, fail the merge.

Each addition is a 20-50 line script. The instrumentation harness pays back ten times over once the pipeline is doing real work for you.
