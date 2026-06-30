# Suggestions for Lucas's Claude Usage

Generated `(2026-06-30)`. Based on knowledge base findings; review and adopt selectively.

---

## Immediate / High-Impact

### 00. `[ACTION URGENT]` Audit for deprecated model IDs — `claude-sonnet-4` and `claude-opus-4` now return errors
As of ~June 28, 2026, Anthropic retired the legacy model IDs `claude-sonnet-4` and `claude-opus-4`. Any API call using these IDs returns an error immediately. Replacement IDs: `claude-sonnet-4-6` and `claude-opus-4-8`.
- Search all Lanzico repos: `grep -r "claude-sonnet-4\|claude-opus-4" --include="*.json" --include="*.yaml" --include="*.env" --include="*.ts" --include="*.js" --include="*.py" .`
- Update any matches; then re-run CI.
- [Releasebot](https://releasebot.io/updates/anthropic/claude-developer-platform)

### 0a. `[ACTION]` Update Claude Code to v2.1.195 (latest as of June 26)
v2.1.195 fixes background job reliability, hook matchers for hyphenated identifiers (now exact-match — check your hooks), and voice dictation on macOS. v2.1.193 adds live file path autocomplete in `!` bash mode. v2.1.191 brings a 37% CPU reduction during streaming and `/rewind` across `/clear`.
- `npm update -g @anthropic-ai/claude-code` or equivalent
- **Verify after update**: if you have hook names with hyphens (e.g. `pre-commit`), test they still fire — the matching behaviour changed in v2.1.195.
- [Changelog](https://code.claude.com/docs/en/changelog)

### 0aa. `[ACTION]` Try Artifacts for client deliverables
New in beta for Team & Enterprise: run a Claude session to produce a report, spec, or changelog, and Claude can publish it as a live shareable page on claude.ai that auto-updates. Removes the "export → re-share" cycle for Lanzico client deliverables. Test on your next internal report or client status update.
- [Releasebot](https://releasebot.io/updates/anthropic/claude)

### 0b. `[ACTION]` Enable `autoMode.classifyAllShell: true` for automated pipelines
New in v2.1.193: routes all Bash/PowerShell commands through the auto-mode safety classifier, not just agent-issued ones. For Lanzico automated runs (like this routine), this adds a safety gate without manual permission prompts.
- Add to `.claude/settings.json`: `"autoMode": { "classifyAllShell": true }`

### 0. `[ACTION]` Try Claude Tag for Lanzico's Slack workspace
Claude Tag (launched June 23) puts a persistent AI teammate inside shared Slack channels — shared by the whole team, with ambient mode so it proactively follows up on threads. Available on Enterprise and Team plans. You already have the Slack MCP connected in Claude Code sessions; Claude Tag extends that into your actual Slack workspace where conversations happen. Start with one internal ops channel to test ambient mode and context persistence before rolling out to client-facing channels.
- [Anthropic intro](https://www.anthropic.com/news/introducing-claude-tag) / [TechCrunch deep-dive](https://techcrunch.com/2026/06/23/anthropics-claude-tag-is-learning-your-company-one-slack-message-at-a-time/)

### 1. `[ACTION]` Keep production pinned to Opus 4.8 — Fable 5 still offline
Fable 5 and Mythos 5 remain suspended under a US government export-control directive (active since June 12). No restoration timeline. Any session or API call targeting `claude-fable-5` will fail. Ensure all Lanzico automations, skills, and Claude Code configs explicitly reference `claude-opus-4-8` or `claude-sonnet-4-6`.
- [Anthropic statement](https://www.anthropic.com/news/fable-mythos-access)

### 2. `[ACTION]` Enable `sandbox.credentials: true` in automated run configs
New in v2.1.187: prevents sandboxed agent commands from reading credential files or secret environment variables. This is a meaningful security control for any automated Lanzico workflow that runs in an environment with credentials present.
- Add to `.claude/settings.json`: `"sandbox": { "credentials": true }`

### 3. `[ACTION]` Enable `attribution.sessionUrl: false` in client-facing repos
As of v2.1.183, you can omit the `claude.ai` session link from git commits and PRs via the `attribution.sessionUrl` setting. If Lanzico pushes commits clients can see, this is worth enabling now.
- Add to `.claude/settings.json`: `"attribution": { "sessionUrl": false }`

### 4. `[ACTION]` Replace stored API keys with Workload Identity Federation
Anthropic now supports WIF — short-lived scoped credentials that integrate with AWS IAM, GCP, and other identity providers. This eliminates the risk of leaked long-lived API keys.
- [Releasebot](https://releasebot.io/updates/anthropic)

---

## New Workflows to Adopt

### 5. `[ACTION]` Use Claude Design `/design-sync` for client work
Claude Design now has a full overhaul: import a client's design system from GitHub, lock it so all AI-generated designs stay on-brand, and sync two-ways with Claude Code using `/design-sync` and `/design`. For Lanzico's client work involving design-to-code handoffs, this closes the loop between Figma/brand assets and coded components.
- Run `/design-sync` from Claude Code to pull the design system into Claude Design
- [VentureBeat](https://venturebeat.com/technology/anthropic-ships-major-claude-design-overhaul-with-design-system-imports-code-round-trips-and-a-fix-for-its-token-burning-problem)

### 5b. `[ACTION]` Compact at 50% context, not the default auto-trigger
Community consensus: manually run `/compact Focus on <current task>` when context reaches ~50%. Waiting for auto-compaction at 70% puts you in the "agent dumb zone" where Claude noticeably loses coherence. Set a mental habit or hook to trigger this proactively.
- [DEV.to](https://dev.to/galian/claude-code-workflow-best-practices-that-ship-code-na)

### 5c. `[ACTION]` Use git worktrees for parallel Claude Code sessions
Run each Claude Code session in a separate worktree (`git worktree add ../feature-branch feature-branch`) rather than the same checkout. Prevents state bleed and merge conflicts when running parallel sessions for different features.
- [DEV.to](https://dev.to/evan-dong/10-battle-tested-claude-code-practices-4n81)

### 6. Use `/btw` for side questions during long sessions
`/btw` runs a question in a dismissible overlay — the answer never enters conversation history. This keeps context clean when you need a quick lookup mid-session without derailing focus.

### 7. Writer/Reviewer parallel session pattern
For any significant PR: implement in Session A, then open Session B and ask it to review just the diff against your plan. Session B has no memory of writing the code, so its review is unbiased. Feed findings back to Session A to iterate.
- `[ACTION]` Use this for all Lanzico feature PRs before client delivery.

### 8. Let Claude interview you for complex features
Start with: `"I want to build [description]. Interview me using AskUserQuestion about implementation, UX, edge cases, and tradeoffs. Then write a complete spec to SPEC.md."` Then start a fresh session to execute from the spec.
- Surfaces blind spots before code is written, not after.

### 9. Use `/goal` for unattended runs
Set a `/goal` condition (e.g., "all tests pass") at the start of a session. A separate evaluator re-checks after every turn and keeps Claude working until the condition holds — no babysitting needed.

### 10. Fan-out large migrations with `claude -p` loops
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

### 12. Current model strategy — Fable 5 offline
Fable 5 and Mythos 5 remain suspended (US export control, no return date). Active model lineup:
- **Opus 4.8**: complex multi-file tasks, architecture work, debugging hard problems — current best available
- **Opus 4.8 + Fast mode** (`/fast`): rapid iteration, reviews, quick answers — 2.5× faster at 2× cost
- **Sonnet 4.6**: balanced speed/capability for mid-complexity tasks
- **Haiku 4.5**: simple lookups, boilerplate, cost-sensitive automations

### 13. Consider API plan if hitting Pro limits
Community reports Pro plan ($20/month) runs out after ~12 heavy prompts per session. If you're doing intensive coding, the API plan with usage-based billing may be more cost-effective and doesn't impose session caps.

---

## Governance (for team use)

### 14. `enforceAvailableModels` for team consistency
If others at Lanzico use Claude Code, the `enforceAvailableModels` managed setting prevents team members from switching to unapproved (expensive) models. Manage via `.claude/settings.json` in a shared config.

### 15. Evaluate Claude on Azure / Claude Platform on AWS for cloud-native workloads
Two options now available for cloud-native Lanzico workloads:
- **Azure (Microsoft Foundry):** Claude Opus 4.8 + Haiku 4.5 via Azure with Entra identity, Azure billing, governance, and optional US data zone.
- **AWS:** Full Claude API (Messages, Files, Batches, Managed Agents) via AWS IAM auth and AWS billing — no separate Anthropic account needed.
Pick whichever matches your existing cloud identity and billing. [Azure announcement](https://www.anthropic.com/news)

### 16. `[ACTION]` Use MCP Connector Observability to audit active connectors
Now that Anthropic surfaces connector errors, latency, and token spend per-connector in the org admin panel, run a quick audit of the 8+ MCP connectors active in this session (Notion, Slack, Gmail, Google Drive, Miro, Canva, Clay, Lemlist). Identify which ones are actually used vs. just connected — disconnecting unused connectors reduces token overhead and attack surface. [Releasebot](https://releasebot.io/updates/anthropic/claude)
