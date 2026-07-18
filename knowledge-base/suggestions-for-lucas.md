# Suggestions for Lucas's Claude Usage

Generated `(2026-06-29)`, updated `(2026-07-18)`. Based on knowledge base findings; review and adopt selectively.

---

## Immediate / High-Impact

### -14. `[ACTION]` Update to Claude Code v2.1.214 — permission-bypass batch, check `dir/**` allow rules
Shipped 2026-07-18. Several Bash/permission-check fixes: single-segment `dir/**` allow rules (e.g. `Edit(src/**)`) were wrongly auto-approving writes to any nested directory of that name anywhere in the tree, not just under the working directory — the most consequential fix, since it means any such rule in a Lanzico `.claude/settings.json` was silently broader than intended. Also closes gaps where very long Bash commands (>10,000 chars), zsh variable subscripts in `[[ ]]`, and certain `help`/`man` invocations could run without a prompt. New `EndConversation` tool lets Claude end a session with an abusive/jailbreak user. Separately fixes scheduled tasks refusing their own configured prompt as untrusted input — directly relevant since this KB routine itself runs as a scheduled task.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.214
- Audit any Lanzico `.claude/settings.json` for single-segment `dir/**` allow rules and re-verify the intended scope
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

### -13. `[ACTION]` Update to Claude Code v2.1.212 — closes a plan-mode Bash-permission bypass
Shipped 2026-07-17. Plan mode was auto-running file-modifying Bash commands (`touch`, `rm`, etc.) without a permission prompt or SDK `canUseTool` callback — now fixed. Also adds session-wide caps on WebSearch calls (200) and subagent spawns (200) to stop runaway loops, and auto-backgrounds MCP tool calls over 2 minutes instead of stalling the session.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.212
- If any Lanzico session or automation uses plan mode, this closes a real gap — file-modifying Bash could previously run unapproved during planning
- Note the new 200/200 WebSearch and subagent-spawn ceilings if any Workflow-based automation fans out wide (tunable via `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` / `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`)
- [Changelog](https://code.claude.com/docs/en/changelog)

### -12. `[ACTION]` Update to Claude Code v2.1.211 — chat-approval spoofing fix + hook `ask`-override fix
Shipped 2026-07-15. Fixes permission previews relayed to chat channels (e.g. Slack approvals) not neutralizing bidirectional-override/zero-width/look-alike characters — previously a malicious tool input could visually alter what the approval message appeared to say. Also fixes auto mode silently overriding a hook's explicit `ask` decision for unsandboxed Bash. Separately fixes a prompt-caching billing regression on Bedrock/Vertex/Mantle/Foundry that was billing cached system context as fresh input tokens every request.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.211+
- If tool-call approvals are ever relayed through Slack or another chat channel for Lanzico automations, this closes a spoofing vector — update promptly
- If any hook gates unsandboxed Bash with an `ask` decision, verify it's now actually honored
- If Lanzico runs Claude Code via Bedrock/Vertex/Mantle/Foundry, this fixes an overbilling bug on cached context
- [Changelog](https://code.claude.com/docs/en/changelog)

### -11. `[ACTION]` Update to Claude Code v2.1.210 — closes a prompt-injection path in unattended runs like this one
Shipped 2026-07-15. Fixes the `ultracode` keyword opt-in firing on non-human-originated input (webhook payloads, relayed PR comments) — previously, external content could trigger an expensive multi-agent Workflow run without a real user asking for it. Also hardens the Agent tool against indirect prompt injection via content a subagent reads, and fixes `isolation: 'worktree'` subagents being able to run git-mutating commands against the main repo checkout instead of their own worktree.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.210
- Directly relevant to this KB routine, which runs unattended and reacts to GitHub webhook activity/PR comments — and to any other Lanzico automation using Workflow or worktree isolation
- [Changelog](https://code.claude.com/docs/en/changelog)

### -10. `[ACTION]` Update to Claude Code v2.1.208 — memory-leak and headless-reliability batch
Shipped 2026-07-14. Fixes several memory leaks that matter for long-running or scheduled sessions (MCP stdio server stderr, LSP documents, async hook output, headless/SDK tool-result payload growth), plus headless `stream-json` hangs and crashes that could silently kill an unattended run. Also fixes a Bedrock SSO auth regression from 2.1.207, and cuts transcript size up to 79x / speeds up tool rounds up to 7x for MCP-heavy sessions.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.208
- Directly relevant to this KB routine itself and any other scheduled/headless Lanzico automation — the memory-leak and hang fixes reduce the odds of a silent failure
- If any Lanzico auth uses AWS SSO with Bedrock where the SSO region differs from the Bedrock region, this update fixes a regression that broke that specific setup
- [Changelog](https://code.claude.com/docs/en/changelog)

### -9. `[ACTION]` Weekly Claude Code rate limits revert tonight (2026-07-13, 6PM PDT)
The 50% weekly-limit boost that's been live since May 13 for Pro/Max/Team/seat-based Enterprise expires today with no announced extension. If any Lanzico workflow (including scheduled routines like this one) runs close to the weekly cap, expect less headroom starting tomorrow.
- Check current weekly usage via `/usage-credits` or the Claude console before the boost lapses
- If usage regularly approaches the cap, consider spacing out non-urgent scheduled runs or moving cost-sensitive automation to Haiku 4.5
- [AI Catchup](https://aicatchup.com/news/claude-code-weekly-limits-50-percent-promo) / [ChatForest](https://chatforest.com/builders-log/claude-code-weekly-limit-50-percent-boost-expires-july-13-builder-action-guide/)

### -8. `[ACTION]` Update to Claude Code v2.1.207 — two security fixes directly relevant to automated runs
v2.1.207 (2026-07-11) fixes a bug where remote managed settings applied from a non-interactive run (`claude -p`, the SDK) were recorded as consented **without ever showing the security consent dialog** — this affects headless/scheduled automation like this KB routine. It also closes a shell-injection vector in plugin hooks/monitors/MCP `headersHelper` (`${user_config.*}` interpolation in shell-form commands is now rejected). Separately, auto mode **no longer reads config from repo-resident `.claude/settings.local.json`** — it now only honors `~/.claude/settings.json`.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.207
- If any Lanzico plugin hooks/monitors use `${user_config.*}` in a shell-form command, they'll now be rejected — switch to exec form (`args` array) or `$CLAUDE_PLUGIN_OPTION_<KEY>`
- If auto mode was ever configured via `.claude/settings.local.json` in any repo, re-add it to `~/.claude/settings.json` — it silently stopped applying
- [Changelog](https://code.claude.com/docs/en/changelog)

### -7. Try Claude Reflect, and run the new `/doctor` CLAUDE.md-trim check
Two small but relevant items from 2026-07-09: **Claude Reflect** (beta) is a usage-habits dashboard on Free/Pro/Max that summarizes what you use Claude for, when, and how — worth a look for Lucas's own usage patterns once Memory is on, though note press reaction has been mixed (see `community-insights.md`). Separately, Claude Code v2.1.206 added a `/doctor` check that proposes trimming checked-in `CLAUDE.md` files down to what Claude can't already derive from the codebase — a good match for the "CLAUDE.md under 200 lines" rule already in this KB.
- Turn on Memory and check Settings → Reflect for a usage recap
- Run `/doctor` (or `/checkup`) against Lanzico's `CLAUDE.md` files next update and apply any trim suggestions
- [MacRumors](https://www.macrumors.com/2026/07/09/anthropic-reflect-claude-tracking/) / [Changelog](https://code.claude.com/docs/en/changelog)

### -6. `[ACTION]` Update to Claude Code v2.1.204+ — fixes idle-reap risk in headless `SessionStart` hooks
v2.1.204 (2026-07-08) fixed hook events not streaming during `SessionStart` hooks in headless sessions, which could get a remote/background worker idle-reaped mid-hook. This applies directly to this KB routine and any other scheduled/headless Claude Code automation that uses a `SessionStart` hook.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.205 (latest, also picks up the fabricated-approval hardening below)
- If this routine or any other scheduled job has ever failed mysteriously partway through startup, this fix is the likely explanation
- Separately, v2.1.205 makes background task notifications explicitly state that no human input has occurred — a hardening measure directly relevant to unattended runs like this one, worth being aware of if writing prompts that reason about "the user confirmed X"
- [Changelog](https://code.claude.com/docs/en/changelog)

### -5. Try Cowork on mobile/web, and check the new Microsoft 365 write tools
Cowork expanded from desktop-only to web and mobile on July 7 (remote sessions, synced files, shared Chat/Cowork home — starting on Max plan). Separately, Claude gained Microsoft 365 write tools this week: drafting/sending email, managing calendar events, and creating/updating OneDrive/SharePoint files, not just reading them.
- If Lucas is on the Max plan, try a Cowork session from phone/web for a task that doesn't need desktop file access
- If Lanzico or a client runs on M365, the write tools could replace manual steps in report/email drafting — worth a trial before building custom integrations for the same purpose
- [TechCrunch](https://techcrunch.com/2026/07/07/the-coding-agent-wars-are-spilling-into-the-rest-of-the-office-claude-cowork/) / [blog.mean.ceo](https://blog.mean.ceo/anthropic-claude-news-july-2026/)

### -4. `[ACTION]` Set Claude Enterprise/Team spend alerts and model entitlements now
Anthropic shipped richer admin analytics, per-role model entitlements, and spend-threshold alerts (75%/90% of org limit) for Claude Enterprise on July 2. Unconfirmed aggregator reports this week claim Microsoft cut Claude Code for ~5,000 engineers over runaway token costs (~$2,000/engineer/month) — treat that specific story as unverified, but it's a timely reminder that token spend on agentic workflows (including this KB routine) can scale faster than expected without a cap.
- If Lanzico is on Enterprise/Team, turn on spend alerts and set model defaults per role so routine tasks don't default to the priciest model
- Otherwise, keep an eye on this routine's own token usage as a sanity check
- [Claude blog](https://claude.com/blog/giving-admins-more-visibility-and-control-over-claude-usage-and-spend)

### -3. `[ACTION]` Check unattended/scheduled runs for `AskUserQuestion` hang risk (v2.1.200)
As of v2.1.200 (July 3), `AskUserQuestion` dialogs no longer auto-continue by default — an idle dialog now waits indefinitely instead of eventually proceeding. This routine and any other scheduled/background Claude Code automation at Lanzico could hang indefinitely if a code path ever triggers `AskUserQuestion` with nobody available to answer.
- Audit scheduled routines (like this one) and any `-p`/background agent workflows for prompts that could surface `AskUserQuestion`
- If found, set an idle timeout via `/config` so the run degrades gracefully instead of hanging forever
- Separately, the permission mode formerly labeled "default" is now shown as "Manual" in the CLI/VS Code/JetBrains — cosmetic rename only, no action needed beyond awareness
- [Changelog](https://code.claude.com/docs/en/changelog)

### -2. `[ACTION]` Update to Claude Code v2.1.198 — Claude in Chrome is GA, `/dataviz` ships built-in
As of July 1, Claude in Chrome (browser control extension) is generally available, and a new `/dataviz` skill ships in the box for building consistent charts/dashboards with a runnable color-palette validator. Also changes default behavior: **background agents started from `claude agents` now auto-commit, push, and open a draft PR** when they finish code work in a worktree, instead of pausing to ask first.
- `npm update -g @anthropic-ai/claude-code` (or equivalent)
- If you rely on background agents pausing before pushing code, re-check that workflow — the default changed
- Try `/dataviz` on the next client chart/report instead of ad hoc styling
- [Changelog](https://code.claude.com/docs/en/changelog)

### -1. `[ACTION]` Update to Claude Code v2.1.197 and try Claude Sonnet 5
Claude Sonnet 5 shipped June 30 and is now the default model in Claude Code — native 1M-token context window, promotional pricing $2/$10 per Mtok through August 31. Requires updating to v2.1.197+.
- `npm update -g @anthropic-ai/claude-code` (or equivalent), then run `/model` to confirm Sonnet 5 is available
- Worth a trial run on a mid-complexity Lanzico task to see if it now covers what previously needed Opus 4.8, at a fraction of the cost
- Early partner feedback (Cursor, Zapier) is strongly positive on agentic reliability — multi-step tasks that used to stall now complete end-to-end
- [Anthropic](https://www.anthropic.com/news/claude-sonnet-5) / [Changelog](https://code.claude.com/docs/en/changelog)

### -0.5. Fable 5 / Mythos 5 are back — but treat cautiously
The June 12 export-control suspension was lifted June 30; Fable 5 is available globally again from July 1 (Pro/Max/Team: up to 50% of weekly limits through July 7, then usage credits). This reverses prior guidance in this KB to avoid Fable 5. Given it was suspended once already on national-security grounds, don't rebuild critical automation around it yet — treat as a strong option for interactive/manual work, keep automated pipelines on `claude-opus-4-8` or `claude-sonnet-5` for now.
- [Anthropic](https://www.anthropic.com/news/redeploying-fable-5)

### 00. `[ACTION URGENT]` Audit for deprecated model IDs — `claude-sonnet-4` and `claude-opus-4` now return errors
As of ~June 28, 2026, Anthropic retired the legacy model IDs `claude-sonnet-4` and `claude-opus-4`. Any API call using these IDs returns an error immediately. Replacement IDs: `claude-sonnet-4-6` and `claude-opus-4-8`.
- Search all Lanzico repos: `grep -r "claude-sonnet-4\|claude-opus-4" --include="*.json" --include="*.yaml" --include="*.env" --include="*.ts" --include="*.js" --include="*.py" .`
- Update any matches; then re-run CI.
- [Releasebot](https://releasebot.io/updates/anthropic/claude-developer-platform)

### 0a. `[ACTION]` Update Claude Code to v2.1.197 (superseded from v2.1.195, see item -1 above for Sonnet 5)
Since v2.1.195: v2.1.196 added org default models, a fix for `.mcp.json` servers auto-spawning in untrusted-but-self-approved workspaces (security), and cut `/code-review` token usage ~25%. v2.1.197 ships Sonnet 5 access. Earlier: v2.1.195 fixed background job reliability, hook matchers for hyphenated identifiers (now exact-match — check your hooks), and voice dictation on macOS.
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

### 12. Current model strategy — updated 2026-07-01, Fable 5 back online, Sonnet 5 shipped (still current as of 2026-07-02)
Fable 5 and Mythos 5 access was restored July 1 after the export-control suspension was lifted. Claude Sonnet 5 also shipped June 30 with a 1M context window and promo pricing through August 31. Active model lineup:
- **Fable 5**: top-tier "Mythos-class" work, but restored only 1 day ago after a prior suspension — pilot cautiously, don't yet depend on it for anything time-critical
- **Opus 4.8**: complex multi-file tasks, architecture work, debugging hard problems — proven, stable default for critical work
- **Opus 4.8 + Fast mode** (`/fast`): rapid iteration, reviews, quick answers — 2.5× faster at 2× cost
- **Sonnet 5** (new): 1M context, $2/$10 promo pricing through Aug 31 — try for mid-to-high complexity tasks that don't strictly need Opus; likely the new cost/performance sweet spot given the price and context window
- **Sonnet 4.6**: balanced speed/capability fallback if Sonnet 5 isn't yet validated for a workflow
- **Haiku 4.5**: simple lookups, boilerplate, cost-sensitive automations

### 13. Consider API plan if hitting Pro limits
Community reports Pro plan ($20/month) runs out after ~12 heavy prompts per session. If you're doing intensive coding, the API plan with usage-based billing may be more cost-effective and doesn't impose session caps.

---

## Governance (for team use)

### 14. `enforceAvailableModels` for team consistency
If others at Lanzico use Claude Code, the `enforceAvailableModels` managed setting prevents team members from switching to unapproved (expensive) models. Manage via `.claude/settings.json` in a shared config.

### 15. Evaluate Claude Platform on AWS
If Lanzico is AWS-native, Claude Platform on AWS provides the full API (including Managed Agents) via AWS billing and IAM auth — no separate Anthropic billing account needed.
