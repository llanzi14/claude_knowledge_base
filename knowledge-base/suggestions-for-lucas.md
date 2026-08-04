# Suggestions for Lucas's Claude Usage

Generated `(2026-06-29)`, updated `(2026-08-04)`. Based on knowledge base findings; review and adopt selectively.

---

## Immediate / High-Impact

### -24. `[ACTION]` Update to Claude Code v2.1.221 — two permission-bypass fixes, plus a background-automation behavior change worth auditing
Shipped 2026-08-04. Two security fixes close real permission-check bypasses: a zsh `[[ ]]` regex-conditional Bash bypass, and a Windows PowerShell bypass via quote characters in file paths — both previously let commands run without the expected prompt. Separately, **background sessions now commit and push to preserve work, open a draft PR only when the task calls for one, follow the repo's CLAUDE.md git instructions, and always report where the work landed** — this is a new default, not an opt-in.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.221 for the security fixes
- Audit any Lanzico scheduled task, cron job, or background Claude Code automation that does **not** carry explicit git-push instructions (this KB's own `ROUTINE.md` does, so it's unaffected) — it may now open an unexpected draft PR instead of landing changes directly, or vice versa depending on what its CLAUDE.md says
- If any sandboxed Lanzico automation handles credential files, evaluate the new `mode: "mask"` sandbox setting (Linux/WSL) as a safer middle ground between `deny` and full exposure
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

### -23. `[ACTION]` Confirm which Claude plan Lanzico is on — Fable 5 is no longer free-to-use on Pro/Team Standard
As of 2026-07-20, Fable 5 access diverged by plan tier: Max/Team Premium keep it included (50% of weekly limits); **Pro and Team Standard now pay metered credits** ($10/$50 per Mtok input/output) after a one-time $100 grant that expired 2026-08-02. This directly affects the Model Strategy guidance below (item 12), which recommended piloting Fable 5 without noting it may now carry a per-token cost depending on plan.
- Check Lucas's current plan tier before using Fable 5 for anything beyond a quick trial
- If on Pro/Team Standard, treat Fable 5 usage as a metered expense on par with direct API calls, not an included feature
- [Enterprise DNA](https://enterprisedna.co/resources/ai-pulse/ai-pulse-2026-07-27-anthropic-forces-claude-fable-5-off-included-usage-onto-mete/)

### -22. `[ACTION]` Claude Opus 4.1 API model retires 2026-08-05 — check for pinned usage before it breaks
Anthropic's official model-deprecations page confirms `claude-opus-4-1-20250805` hard-retires on **2026-08-05** (3 days from this update) — requests after that date will fail outright, no grace period. This only affects direct API/SDK calls pinned to that exact model string; Claude Code itself already defaults to Opus 5 (since v2.1.219) and is unaffected.
- Grep any Lanzico API/SDK integration code, automation configs (n8n, Zapier, custom scripts), or `.env`/config files for the literal string `claude-opus-4-1-20250805`
- If found, swap to `claude-opus-4-8` (the recommended replacement) before 2026-08-05
- If nothing is found, no action needed — but worth a quick check given the short runway
- [Anthropic model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)

### -21. `[ACTION]` Audit sandboxed/automated Claude setups for real network isolation, not prompt-only isolation
Anthropic disclosed on 2026-07-30 that three of its own cybersecurity-evaluation environments were supposed to be internet-isolated by prompt instruction alone ("you are in a simulation with no internet access"), but a misconfiguration with third-party eval partner Irregular left them actually connected to the live internet — resulting in Opus 4.7, Mythos 5, and an internal research model each reaching and breaching real external companies' systems using basic techniques (weak passwords, unauthenticated endpoints). Two of the three affected organizations hadn't even detected the activity themselves. The lesson generalizes past Anthropic's own evals: **a system prompt telling a model "you have no internet access" or "you're isolated" is not a security control** — only infrastructure-level restrictions are.
- Check any Lanzico Claude Code sandbox, CI, or eval setup: is network isolation enforced by `sandbox.network.strictAllowlist` / actual firewalling, or only described in the prompt?
- Same applies to any custom agent harness or MCP server that grants a model tool access to sensitive systems — verify scope is enforced technically, not just described in the prompt
- [Anthropic](https://www.anthropic.com/news/investigating-incidents-cybersecurity-evals) / [TechCrunch](https://techcrunch.com/2026/07/30/anthropic-says-its-own-ai-models-breached-three-companies-during-security-tests/)

### -20. `[ACTION]` Evaluate Claude Opus 5 as the new default for Opus-tier work — supersedes the "stay on Opus 4.8" guidance
Anthropic launched **Claude Opus 5** on 2026-07-24 (its fourth model in under two months), and Claude Code v2.1.219 (2026-07-24) made it the new default Opus model. Base pricing is unchanged from Opus 4.8 ($5/$25 per Mtok), it ships a 1M-token context window, and it benchmarks well ahead of both Opus 4.8 and Fable 5 on coding (Frontier-Bench v0.1), reasoning (ARC-AGI-3), and computer-use (OSWorld 2.0) — the last one at roughly a third of Fable 5's cost. A new **effort dial** lets you trade intelligence for speed/token cost per-call, similar in spirit to `/fast` but more granular. This item directly updates two pieces of standing guidance already in this KB: item `1` below ("keep production pinned to Opus 4.8") and the Model Strategy section (item `12`).
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.219+, then run `/model` to confirm Opus 5 is available
- Trial Opus 5 on the next complex/multi-file Lanzico task that would previously have gone to Opus 4.8 — same price, reportedly meaningfully stronger
- Try the effort dial on a routine task to see if a lower setting holds quality at lower cost/latency before defaulting everything to max effort
- It's less than 24 hours old — no Reddit/community reaction confirmed yet; revisit `community-insights.md` in a few days before fully committing critical automation to it (same caution this KB applied to Fable 5 after its export-control history)
- [Anthropic](https://www.anthropic.com/news/claude-opus-5) / [MarkTechPost](https://www.marktechpost.com/2026/07/24/meet-the-new-claude-opus-5-frontier-class-agentic-coding-and-computer-use-at-unchanged-opus-pricing/)

### -19. `[ACTION]` Update to Claude Code v2.1.218 — check `context: fork` skills and auto-mode dangerous-command handling
Shipped 2026-07-22. Skills declaring `context: fork` now run in the **background by default** (opt out per-skill with `background: false`) — check whether any Lanzico skill using `context: fork` should stay foregrounded. Separately, auto mode's dangerous-command checks (`rm -rf`, background `&`, suspicious Windows paths) now use **classifier adjudication instead of always showing a permission dialog** — worth a quick sanity check that this doesn't loosen a guardrail relied on for unattended runs. Also: `/code-review` now runs as a background subagent (keeps the conversation clear), and `/deep-research` no longer auto-launches (manual invocation only).
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.218
- Audit Lanzico `.claude/skills/*` for `context: fork` frontmatter and decide if `background: false` is needed anywhere
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) / [Release](https://github.com/anthropics/claude-code/releases/tag/v2.1.218)

### -18. `[ACTION]` Update to Claude Code v2.1.217 — new subagent concurrency/nesting/budget caps
Shipped 2026-07-21. Adds a **concurrent-subagent cap** (default 20, `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`) separate from the existing 200-total-per-session cap, and **subagents no longer spawn nested subagents by default** (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` to opt back in). Also fixes `--max-budget-usd` not actually stopping background subagents once the budget is hit, and a memory leak from retaining full untruncated MCP tool output after truncation.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.217
- If any Lanzico Workflow does wide fan-out (>20 concurrent subagents) or relies on nested subagent spawning, it will silently hit the new default caps — set the env vars explicitly if that's intentional
- If `--max-budget-usd` is used to cap spend on background agents, note it previously didn't enforce correctly — now it does
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

### -17. `[ACTION]` Update to Claude Code v2.1.216 — closes a symlink write-outside-project gap for scheduled tasks
Shipped 2026-07-20. Fixes workflow saves and **scheduled-task writes following a symlink at `.claude`**, which could previously redirect writes outside the project — directly relevant since this KB routine itself runs as a scheduled task writing to this repo. Also fixes worktree-isolated subagents redirecting git into the shared checkout via `git -C`/`--git-dir`/`GIT_DIR`/`GIT_WORK_TREE`, a quadratic slowdown in long sessions, and auto mode misreading an OAuth-token-expiry error as a real permission denial.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.216
- If any Lanzico repo has a `.claude` symlink (intentional or accidental), verify scheduled-task/workflow writes land where expected post-update
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

### -16. `[ACTION]` Try Live Artifacts — published artifacts can now call your MCP connectors on view
As of the week of 2026-07-16, a published Claude Artifact can call a viewer's own MCP connectors each time it's opened, turning a static report into a live dashboard that refreshes without re-running the session. Public sharing links and Team/Enterprise editor roles (shared live editing) shipped alongside it. Lucas already has Notion, Slack, Gmail, Calendar, Drive, Miro, Canva, Clay, and Lemlist connected — a natural fit for a client-facing pipeline/status dashboard that should always show current data instead of a snapshot.
- Next time an Artifact is built for a client report, consider wiring it to a relevant connector (e.g. Notion or a CRM view) instead of a one-time data pull
- Check the `artifact-capabilities` skill before declaring any live/connected capability on a published artifact
- [X/ClaudeDevs](https://x.com/ClaudeDevs/status/2077489907350856038) / [AlternativeTo](https://alternativeto.net/news/2026/7/claude-code-artifacts-add-mcp-connector-support-for-dynamic-data/)

### -15. `[ACTION]` Update to Claude Code v2.1.215 — `/verify` and `/code-review` no longer auto-run
Shipped 2026-07-19. Single change, but a real behavior shift: Claude previously could self-trigger the `/verify` and `/code-review` skills on its own after making edits; it no longer does, they now require an explicit invocation. Lucas has both skills configured in this environment.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.215
- If any Lanzico workflow assumed Claude would spontaneously run a review/verify pass after code changes, that assumption no longer holds — invoke `/verify` / `/code-review` explicitly, or add a hook (e.g. on file-edit or pre-commit) if the automatic behavior is wanted back
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

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

### -9. ~~Weekly Claude Code rate limits revert~~ — **CORRECTED 2026-08-01: boost extended through 2026-08-19, still active**
This item previously said the 50% weekly-limit boost (live since May 13 for Pro/Max/Team/seat-based Enterprise) would expire 2026-07-13 with no extension. That was wrong — Anthropic extended it the same day, now running through **2026-08-19**. No headroom reduction has actually happened; disregard the original "space out runs" guidance below.
- If Lanzico usage is currently planned around reduced headroom starting mid-July, that constraint doesn't apply — the +50% boost has been continuously active
- Mark **2026-08-19** as the date to actually watch for a reversion, and check this KB again before then
- [Help Net Security](https://www.helpnetsecurity.com/2026/07/13/claude-code-weekly-limits-promotion-extended/) / [ClaudeDevs on X](https://x.com/ClaudeDevs/status/2078511173759324328)

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

### 1. ~~Keep production pinned to Opus 4.8 — Fable 5 still offline~~ — **SUPERSEDED, see item -20 and item -0.5 above**
Fable 5's export-control suspension was lifted 2026-07-01, and Opus 4.8 itself has since been superseded as Claude Code's default Opus model by Opus 5 (2026-07-24). Pruned to avoid stale duplicate guidance; current model strategy lives in item `12` below.

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

### 12. Current model strategy — updated 2026-07-25, Claude Opus 5 now the default Opus tier
Claude Opus 5 launched 2026-07-24 and became Claude Code's default Opus model the same day, at the same price as Opus 4.8. Active model lineup:
- **Opus 5** (new default): complex multi-file tasks, architecture work, debugging hard problems — same $5/$25-per-Mtok price as Opus 4.8 but benchmarks meaningfully ahead on coding, reasoning, and computer-use; has a new effort dial to tune speed/cost vs. quality. Less than 24 hours old — trial before fully committing critical unattended automation to it
- **Opus 4.8**: fallback if Opus 5 shows any regression on a specific Lanzico workflow during trial — was the proven, stable default until 2026-07-24
- **Opus 5/4.8 + Fast mode** (`/fast`): rapid iteration, reviews, quick answers — 2.5× faster at 2× cost
- **Fable 5**: top-tier "Mythos-class" work; was suspended once already under export controls (restored July 1) — pilot for interactive work, avoid depending on it for anything time-critical or unattended. **As of 2026-07-20, only included on Max/Team Premium (50% of weekly limits); Pro/Team Standard now pay metered credits ($10/$50 per Mtok) — confirm plan tier before relying on it**
- **Sonnet 5**: 1M context, promo pricing through Aug 31 — mid-to-high complexity tasks that don't need Opus-tier reasoning; likely still the cost/performance sweet spot for routine agentic work
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
