# Suggestions for Lucas's Claude Usage

Generated `(2026-06-22)`. Based on knowledge base findings; review and adopt selectively.

---

## Immediate / High-Impact

### 1. `[ACTION]` Check Fable 5 billing — TODAY
Fable 5 is free on subscription plans through June 22 (today). Usage credits are required from June 23. If Lanzico relies on Fable 5 for Claude Code sessions, verify your billing is set up to cover credits.
- [Pricing details](https://claudefa.st/blog/models/claude-fable-5-mythos-5)

### 2. `[ACTION]` Enable `attribution.sessionUrl: false` in client-facing repos
As of v2.1.183, you can omit the `claude.ai` session link from git commits and PRs via the `attribution.sessionUrl` setting. If Lanzico pushes commits clients can see, this is worth enabling now.
- Add to `.claude/settings.json`: `"attribution": { "sessionUrl": false }`

### 3. `[ACTION]` Replace stored API keys with Workload Identity Federation
Anthropic now supports WIF — short-lived scoped credentials that integrate with AWS IAM, GCP, and other identity providers. This eliminates the risk of leaked long-lived API keys.
- [Releasebot](https://releasebot.io/updates/anthropic)

---

## New Workflows to Adopt

### 4. Use `/btw` for side questions during long sessions
`/btw` runs a question in a dismissible overlay — the answer never enters conversation history. This keeps context clean when you need a quick lookup mid-session without derailing focus.

### 5. Writer/Reviewer parallel session pattern
For any significant PR: implement in Session A, then open Session B and ask it to review just the diff against your plan. Session B has no memory of writing the code, so its review is unbiased. Feed findings back to Session A to iterate.
- `[ACTION]` Use this for all Lanzico feature PRs before client delivery.

### 6. Let Claude interview you for complex features
Start with: `"I want to build [description]. Interview me using AskUserQuestion about implementation, UX, edge cases, and tradeoffs. Then write a complete spec to SPEC.md."` Then start a fresh session to execute from the spec.
- Surfaces blind spots before code is written, not after.

### 7. Use `/goal` for unattended runs
Set a `/goal` condition (e.g., "all tests pass") at the start of a session. A separate evaluator re-checks after every turn and keeps Claude working until the condition holds — no babysitting needed.

### 8. Fan-out large migrations with `claude -p` loops
For repetitive tasks across many files (e.g., migrating components, updating API calls):
```bash
for file in $(cat files.txt); do
  claude -p "migrate $file from X to Y. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```
Test on 2-3 files first, then run at scale overnight.

---

## Skills & Agents to Build

### 9. `[ACTION]` Create a `fix-issue` skill
Save a skill that automates the full GitHub issue → fix → PR cycle:
```markdown
.claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: Fix a GitHub issue end-to-end
disable-model-invocation: true
---
Analyze and fix the GitHub issue: $ARGUMENTS.
1. gh issue view $ARGUMENTS
2. Understand the problem
3. Search codebase for relevant files
4. Implement fix
5. Write and run tests
6. Lint/type-check
7. Commit and open PR
```
Invoke with `/fix-issue 1234`.

### 10. `[ACTION]` Create a `security-reviewer` subagent
```markdown
.claude/agents/security-reviewer.md
---
name: security-reviewer
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for injection
vulnerabilities, auth flaws, secrets in code, and insecure data handling.
Provide specific line references and fixes.
```
Tell Claude: `"use a subagent to review this PR for security issues."` Or evaluate the new **Claude Security** product for automated scanning.

### 11. `[ACTION]` Map Lanzico MCP connections
You have Notion, Slack, Gmail, Google Calendar, Google Drive, Miro, Canva, Clay, and Lemlist MCP tools connected in this session. Confirm each is wired into your Claude Code environment (`.claude/settings.json` → `mcpServers`), so Claude can act on them directly during coding sessions.

---

## Model Strategy

### 12. Use Fable 5 for coding; Opus 4.8 Fast for fast iteration
- **Fable 5**: complex multi-file tasks, architecture work, debugging hard problems (best SWE-Bench score)
- **Opus 4.8 + Fast mode** (`/fast`): rapid iteration, reviews, quick answers — 2.5× faster at 2× cost
- **Haiku 4.5**: simple lookups, boilerplate generation, cost-sensitive automations

### 13. Consider API plan if hitting Pro limits
Community reports Pro plan ($20/month) runs out after ~12 heavy prompts per session. If you're doing intensive coding, the API plan with usage-based billing may be more cost-effective and doesn't impose session caps.

---

## Governance (for team use)

### 14. `enforceAvailableModels` for team consistency
If others at Lanzico use Claude Code, the `enforceAvailableModels` managed setting prevents team members from switching to unapproved (expensive) models. Manage via `.claude/settings.json` in a shared config.

### 15. Evaluate Claude Platform on AWS
If Lanzico is AWS-native, Claude Platform on AWS provides the full API (including Managed Agents) via AWS billing and IAM auth — no separate Anthropic billing account needed.
