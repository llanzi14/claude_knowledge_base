# Suggestions for Lucas's Claude Usage

Generated `(2026-06-29)`, updated `(2026-09-04)`. Based on knowledge base findings; review and adopt selectively.

---

## Immediate / High-Impact

### -51. `[ACTION]` Evaluate Enterprise Frontier Safeguards (EFS) for any client work touching sensitive data
Announced 2026-09-01/02: a free enterprise offering that stores Claude session transcripts/logs in the *customer's own* cloud (S3/Azure Blob/GCS, customer-held encryption keys) while still running Anthropic's automated misuse monitoring — no human review by Anthropic staff required. Rolls out across Claude Code, Claude Enterprise, the Claude Platform, Bedrock, Google's Agent Platform, and Microsoft Foundry, in phases through this fall. This directly answers the kind of data-residency/zero-retention concern that would come up with a financial-services, healthcare, or legal client considering Claude-based automation.
- If any current or prospective LanziCo/Odoovers Growth client has sensitive-data concerns blocking a Claude Enterprise/Platform/Bedrock proposal, flag EFS as a concrete, no-cost mitigation once it's confirmed live for the relevant plan tier
- Re-verify pricing/availability directly with Anthropic before citing specifics to a client — sourced from third-party coverage only, `anthropic.com` unreachable from this environment
- See `knowledge-base/releases-and-features.md` (Platform & Enterprise, 2026-09-04) for full details
- [SecurityWeek](https://www.securityweek.com/anthropic-details-response-to-security-incidents-unveils-enterprise-safeguards/) / [MarkTechPost](https://www.marktechpost.com/2026/09/02/anthropic-enterprise-frontier-safeguards-efs/)

### -50. `[ACTION]` Update Claude Code to v2.1.260 — fixes a permission-rule bug that could silently break all file edits, plus new prompt-cache-miss diagnostics
A single invalid file permission rule (bad regex from parentheses in a path, etc.) was breaking *all* file edits, not just the one bad rule — now fixed, along with the parentheses-in-paths and zsh-command-substitution-hiding bugs that could trigger it. Separately, `/cost` and the status line's `prompt_cache` field now name a likely cause when a prompt-cache miss happens (tool definitions changed, system prompt changed, idle past TTL) — useful the next time this KB routine or any Lanzico automation sees an unexpected full-price re-cache instead of guessing. Also fixes prompt caching on Fable 5.1 not covering context after tool results, which had been undercutting Fable 5.1's new 75% cache-read discount (see item `-51`'s neighbor entry in `releases-and-features.md`, Models section).
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.260
- If any Lanzico `.claude/settings.json` permission rule ever silently broke all file edits before, that's this bug — safe to re-add now
- Try the new `/diff` panel and prompt-cache-miss diagnostics next time this KB routine's own session behaves unexpectedly
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -49. `[ACTION]` Update Claude Code to v2.1.259 and use `--permission-prompts none` for this KB routine's own scheduled session
Shipped 2026-09-02. New **`--permission-prompts none`** flag: on a headless host, anything that would normally show a permission dialog is auto-denied instead of hanging, while the active permission mode (including auto mode) keeps deciding everything else. This is a more explicit unattended-session guarantee than relying on auto mode alone — worth adopting for this KB routine's own scheduled run and any other headless/cron Lanzico automation, since a hung prompt on an unattended host previously meant a silently stalled run. Separately, a reliability fix matters if more than one Claude Code session ever runs on the same machine at once: concurrent sessions were silently reverting each other's `~/.claude.json` changes (workspace trust resetting, MCP/project state getting lost) — closed in this release. Also closes further Bash `Read()` deny-rule gaps (git diff/grep file operands, `cd DIR && cat FILE` compounds) and makes managed settings fail closed and loudly instead of silently going unenforced on a parse error.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.259
- Add `--permission-prompts none` (or the equivalent env var if this KB routine's own scheduler exposes one) to this routine's own invocation so a stray permission dialog can never hang a scheduled run
- If any Lanzico setup runs multiple Claude Code sessions concurrently on one machine, this release is why workspace trust/MCP state may have appeared to reset unexpectedly before
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -48. `[ACTION]` Update Claude Code to v2.1.258 — fixes a scheduled-session bug this KB routine itself could hit, plus review the new auto-mode Containment Escape rule
v2.1.258 (2026-09-01) fixes remote and **scheduled sessions** failing with "user messages must have non-empty content" after a re-sent permission approval couldn't be applied — this KB routine runs as a scheduled session, so this is a direct reliability fix for it. The preceding v2.1.257 also adds an auto-mode **Containment Escape** rule (blocks cloud metadata-credential fetches, egress evasion, and cross-tenant reach from auto-approval unless marked expected) and ships **Fable 5.1** as the new default Fable model.
- Update to v2.1.258 on any machine running this routine or other scheduled Claude Code automation
- If Auto mode is in use for any Lanzico automation, no action needed on Containment Escape — it tightens auto-approval by default — but worth knowing the category of risk it now blocks
- No pricing/behavior change from adopting Fable 5.1 vs. Fable 5 — same rate, larger context
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -47. `[ACTION]` Claude Code weekly limits effectively shrink 2026-09-14 — check current usage patterns before then
Anthropic is ending the temporary +50% weekly-limit boost (running since 2026-05-13) on 2026-09-14 and replacing it with a permanent +25% increase over the *original* baseline. Because +25% is measured from the pre-promo baseline rather than from today's boosted level, this is a net ~17% cut in available weekly usage from where things stand today, despite being announced as a "permanent increase." This is relevant to any Lanzico Claude Code usage that runs near the weekly cap — including this KB routine itself, which runs Claude Code daily.
- Check `/usage` now (before 2026-09-14) to see how close current usage runs to the weekly cap under the boosted limit
- If usage is currently high relative to the cap, plan to either trim usage or budget for hitting limits sooner starting 2026-09-14
- No action needed if usage has consistent headroom well below the cap
- [BleepingComputer](https://www.bleepingcomputer.com/news/artificial-intelligence/anthropic-is-cutting-claude-codes-current-weekly-limits-by-17-percent/) / [XenoSpectrum](https://xenospectrum.com/en/claude-code-weekly-limit-change/)

### -46. `[ACTION]` Infostealer malware is hijacking active Claude sessions to drain usage — scan your machine and check billing history
Anthropic disclosed (reported 2026-08-30/31) that common Windows infostealer malware (Vidar, LummaC2, StealC, RedLine, Acreed) is stealing already-logged-in Claude browser sessions and reusing them to access accounts and burn usage — bypassing password/2FA entirely since it reuses a live session cookie, not credentials. The tell is usage limits appearing to refill then drain without you using Claude. Anthropic is proactively signing out and refunding affected accounts, but detection is reactive on their end. Separately, a distinct report found fake Anthropic-branded sites serving a fileless infostealer aimed specifically at Claude Code users — relevant given this KB routine and other Lanzico automation run Claude Code regularly.
- Run an AV/malware scan on any machine used to log into Claude or run Claude Code, especially if it's ever used for casual downloads/browsing
- Check Claude account billing and usage history for any unexplained spikes or refills-then-drains
- Only ever install Claude/Claude Code from `claude.ai`/`claude.com`/the official npm package — treat any other "Anthropic" download link or lookalike domain as phishing
- [BleepingComputer](https://www.bleepingcomputer.com/news/artificial-intelligence/anthropic-warns-infostealer-malware-is-hijacking-claude-sessions-to-drain-usage/) / [Hackread](https://hackread.com/fake-anthropic-sites-fileless-infostealer-claude-code-users/)

### -45. `[ACTION]` Claude Tag can now be configured with standing instructions committed to GitHub — a concrete on-ramp for the still-open "add Claude Tag" recommendation
Anthropic updated Claude Tag on 2026-08-13 (only surfaced in this KB today, 2026-08-30 — worth flagging that the earlier "-0" item's underlying capability has moved since it was written): ambient mode now reads context across a whole channel plus memory and **standing instructions** to pick among reply-inline, start-a-thread, route-to-workstream, or stay-silent — about 30% better at knowing when to stay quiet. The part that matters most for Lanzico: standing instructions can be **markdown skill files committed to a GitHub repo**, so channel behavior is versioned and reviewable the same way code is, instead of living only in an admin UI. `@Claude what triggers do you have set up here?` in any channel lists and disables active triggers.
- This KB has recommended trying Claude Tag since item `0` (2026-06-25) without it having been adopted yet — if Lanzico's Slack is on Team/Enterprise, this update is a good trigger to actually pilot it now: start with one internal-ops channel, define its standing instructions as a committed skill file, and use the `@Claude what triggers...` command to verify what's active before expanding to a client-facing channel
- If Claude Tag is already piloted somewhere, check whether its current triggers are configured via the admin UI or GitHub skills, and consider moving them to GitHub for review/version history
- [Claude blog](https://claude.com/blog/claude-tag-now-reads-even-more-of-the-room)

### -44. `[ACTION]` Update Claude Code to v2.1.251 (permission-bypass fixes); consider `--restricted` mode for hardened automation
v2.1.251 (2026-08-28) bundles several permission/sandbox-bypass fixes: a symlink-swap bug that let Read/Write/Edit escape the approved working directory, a plugin path-traversal bug, and related Workflow/Grep/Glob deny-rule gaps. Separately, v2.1.248 (2026-08-27) added `--restricted`/`CLAUDE_CODE_RESTRICTED=1` — a new mode that strips code-execution tools and WebFetch, confines file tools to the working directory, refuses `bypassPermissions`, and ignores settings files. Also directly relevant to this KB routine: v2.1.248 fixed a `ScheduleWakeup`-related prompt-cache miss that could hit a resumed session during usage overage.
- Update Claude Code to v2.1.251+ across any Lanzico machine/session running it, for the permission-bypass fixes alone
- Consider `--restricted` for any automation that only needs to read/write files in a known directory and doesn't need Bash or WebFetch — e.g. a report-formatting or file-transform task, where removing code-execution entirely is a stronger guarantee than a permission allowlist
- No action needed on the `ScheduleWakeup`/resume cache-miss fix beyond updating — it's a reliability fix for scheduled routines like this one
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -43. `[ACTION]` Try Cowork's new built-in browser for research tasks
Rolling out this week (announced 2026-08-27): Cowork in Claude Desktop can open a sandboxed browser in its side panel and navigate/read/click/type on real websites, with no separate browser window needed and no shared browsing data unless cookies are imported per-site. On Team plans it's on by default as it rolls out. This is a plausible fit for the kind of open-web research the `marketing-researcher` skill and general Cowork sessions already do by hand (checking a competitor's site, pulling data from a web dashboard, verifying a claim on a live page) — worth a trial run before assuming a separate browser tab is still needed.
- Confirm which plan Lanzico's Cowork seat is on and whether the browser is already active (Settings → Cowork → Preferred browser)
- Trial it on a real research task next time `marketing-researcher` or a Cowork session needs to check something live on the web
- If Lanzico ever moves to Enterprise, note it turns on by default 2026-09-10 unless disabled — a deliberate choice, not a surprise
- [Claude blog](https://claude.com/blog/cowork-built-in-browser)

### -42. Claude Managed Agents adds session budgets, advisor models, geo pinning, GitHub-hosted skills — relevant only if Lanzico builds directly on the API
Reported week of 2026-08-24. Four new governance/capability levers on Managed Agents sessions: hard spend budgets (session pauses on `budget_reached`, resumable by raising the cap), an optional advisor model the primary agent can consult mid-turn, `inference_geo` pinning (`us` at 1.1x vs `global` at standard rate), and auto-discovery of skills from a mounted GitHub repo's `.claude/skills` directory. None of this applies to ordinary Claude Code/Cowork usage — it's specifically for teams running their own agents on the Managed Agents API.
- No action unless Lanzico or a client engagement involves building a custom agent on Managed Agents (e.g. a dedicated client-facing automation with its own spend controls)
- If that ever happens, GitHub-hosted skills is the most immediately useful piece — skills can live and version in a repo instead of being baked into the agent definition
- [ClaudeDevs](https://x.com/ClaudeDevs/status/2085853169930957158)

### -41. `[ACTION]` Salesforce + Anthropic "Claudeforce" — worth a look for Salesforce-using clients, and as a model for an Odoo-in-Claude equivalent
Announced 2026-08-26. **Salesforce in Claude** ships 37 pre-built sales skills (meeting prep, deal-health reviews, pipeline analysis) that let a seller query, update, and act on live CRM data from inside Claude instead of the Salesforce app. Pilot customers now, open beta September 2026. Two angles for Lanzico: (1) if any current or prospective LanziCo/Odoovers Growth client runs Salesforce, this is a concrete new capability to be aware of before proposing custom CRM-automation work — it may cover ground Lanzico would otherwise build by hand; (2) since Lanzico's own CRM/sales analysis work (the `business-analyst` skill) is built around **Odoo**, not Salesforce, this is a useful reference for what a comparable "Odoo in Claude" skill/plugin could look like — pre-built skills for pipeline audits, deal-health checks, and rep performance reviews, callable conversationally instead of via manual export/analysis.
- If a client runs Salesforce, mention Claudeforce's September open-beta timeline before scoping new Salesforce-adjacent automation work for them
- Consider whether the `business-analyst` skill's existing Odoo workflows (pipeline audits, conversion/velocity reviews, salesperson performance) could be packaged as a small set of named, directly-callable skills the way Salesforce in Claude does — same underlying analysis, lower-friction invocation
- Re-confirm pricing/availability directly with Anthropic/Salesforce before quoting terms to a client — sourced from CNBC/VentureBeat, `salesforce.com`/`anthropic.com` unreachable from this environment
- [CNBC](https://www.cnbc.com/2026/08/26/salesforce-anthropic-partnership-claudeforce.html) / [VentureBeat](https://venturebeat.com/orchestration/salesforce-just-put-its-entire-crm-inside-claude-and-says-youll-never-need-its-app-again)

### -40. `[ACTION]` Update to Claude Code v2.1.246 — use the new Auto mode tab in `/permissions` to actually audit the classifier, and check Bash allow rules for wildcard-before-subcommand
Shipped 2026-08-25. This closes a gap this KB has flagged since Auto Mode went default on Pro/Max/Team (item `-29`, 2026-08-14): there was previously no native way to see or edit exactly what the auto-mode classifier will and won't auto-approve, only env vars and general settings. The new **Auto mode tab in `/permissions`** shows and lets you edit those classifier rules directly. Separately, a new startup warning flags Bash allow rules with a wildcard placed *before* the subcommand (e.g. `Bash(git * main)`) — that shape can match commands beyond what was intended, similar in spirit to the `dir/**` scoping bug logged in v2.1.214 (item `-14`).
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.246
- Open `/permissions` → Auto mode tab and actually review what the classifier currently allows for this KB routine's own session and any other Lanzico Pro/Max/Team seat — this is the concrete follow-through the `-29` item asked for, now that there's a UI for it
- Grep any Lanzico `.claude/settings.json` for Bash allow rules with a wildcard before the subcommand and re-scope if the warning fires
- Bonus: `/cd` now applies the new directory's settings/hooks/`.mcp.json`/skills/agents immediately instead of requiring a restart
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -39. `[ACTION]` Claude Security (public beta) — potential security-audit add-on for LanziCo/Odoovers Growth client engagements
Anthropic brought Claude Mythos 5 into Claude Security on 2026-08-21, moving it from limited research preview to public beta for Claude Enterprise customers: point it at a repository, get back vulnerability findings (CWE-classified, severity, confidence, suggested patch) for human review — no direct model access, just the scan output. This is a distinct service angle worth a gut-check: if any client engagement already involves reviewing or maintaining a codebase (Odoo customizations, integrations, internal tools), a Claude Security pass could be packaged as a low-effort add-on (automated first-pass security audit + remediation suggestions) rather than something LanziCo builds in-house.
- Only actionable if Lanzico or the client is on (or willing to move to) Claude Enterprise — confirm current plan tier before pursuing
- Treat as beta: verify output quality/false-positive rate on a low-stakes internal repo before offering it to a client
- Sourced from third-party coverage only (MarkTechPost, Infosecurity Magazine) — anthropic.com/claude.com are unreachable from this environment as of this writing; re-confirm exact terms directly with Anthropic before quoting pricing/availability to a client
- [MarkTechPost](https://www.marktechpost.com/2026/08/21/anthropic-brings-claude-mythos-5-to-claude-security/)

### -38. `[ACTION]` Update to Claude Code v2.1.239 — retry mode now fails fast on real spend limits, `WebFetch` cache-TTL fix, `/claude-api upgrade` for Python SDK migration
Shipped 2026-08-21 — one of the largest single-release changelogs logged in this KB (~50 items, mostly fixes). Two items directly relevant to this KB routine's own reliability: (1) persistent retry mode (`CLAUDE_CODE_RETRY_WATCHDOG`) now fails immediately on organization spend-limit and out-of-credits errors instead of retrying indefinitely against a limit that will never clear — worth confirming this routine (and any other unattended Lanzico automation using persistent retry) surfaces that failure rather than silently hanging; (2) fixed `WebFetch` retaining expired cached page content for the whole session instead of the intended 15-minute TTL — this routine's own research fetches use `WebFetch` for the Claude Code changelog and Anthropic newsroom each run, so a long session could previously have served stale pages past the cache window. Separately, if Lanzico has any direct Python `anthropic` SDK usage, `/claude-api upgrade` now automates the 0.x → 1.x migration (timeouts move from `httpx.Timeout` to `anthropic.Timeout`). Also: `ListAgents` now tells a session its own addressable name and correctly lists live teammates (previously a reachable teammate could look absent); Windows cross-session messaging now works, matching macOS/Linux; fixed Bedrock streaming behind Content-Type-stripping proxies silently doubling billed API calls — check if relevant to any Lanzico Bedrock+proxy setup.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.239
- No specific config change needed for the retry/cache fixes beyond updating — both close silent-failure/staleness gaps this routine could otherwise hit unnoticed
- If Lanzico has direct Python SDK usage, run `/claude-api upgrade` to migrate off `anthropic` 0.x before it's deprecated
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -37. `[ACTION]` Update to Claude Code v2.1.238 — plugin `headersHelper` trust-dialog gating, unbounded-memory fix, cross-session `SendMessage` no longer fails silently
Shipped 2026-08-20 — the largest single-day changelog logged in this KB (~30 items). Three items worth acting on: (1) plugin marketplaces' new `headersHelper` (a configured command that mints HTTP headers, e.g. a short-lived auth token, for catalog/archive fetches) now requires the project's trust dialog to have been accepted and runs without inherited credential env vars — if Lanzico ever adds a custom or third-party plugin marketplace, confirm it isn't relying on the old, less-gated behavior; (2) fixed unbounded memory growth in long interactive sessions (subagent tool results now release once they scroll out of the display window) — relevant to any long-running Lanzico session, including this KB routine's own multi-hour research runs; (3) cross-session `SendMessage` now reports back to the sender when a recipient session refuses inbound messages or its inbox drops one (rate limit/full queue), instead of the message silently vanishing — directly relevant to this routine's own `SendMessage`/`ListAgents` use and any other multi-session Lanzico automation that assumed delivery.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.238
- If Lanzico adds a plugin marketplace with `headersHelper`, verify the relevant project folder's trust dialog has been accepted
- No specific config change needed for the memory-growth or cross-session-messaging fixes beyond updating — both close silent-failure gaps
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -36. `[ACTION]` Update to Claude Code v2.1.236/v2.1.237 — `ANTHROPIC_DEFAULT_MODEL`, cross-session idle notice, built-in Concise output style
Shipped 2026-08-19 and 2026-08-20. Three items relevant to this KB routine and other scheduled/multi-session Lanzico automation: (1) new `ANTHROPIC_DEFAULT_MODEL` env var sets the model new sessions start on (a `/model` pick still overrides and persists, unlike `ANTHROPIC_MODEL`) — useful for pinning a default model per automation without touching each session's settings; (2) `notify_when_idle` added to cross-session `SendMessage` — one Claude Code session on the same machine can now ask another to send a single notice when it next goes idle, opt-in and one-shot, no polling required; (3) v2.1.237 ships a **built-in "Concise" output style** (select under Output style in `/config`) that leads with results and skips preamble/narration while doing the work just as thoroughly — this KB previously logged Concise as a community-recommended `/config` option (2026-07-14 best-practices entry); it's now first-party, worth turning on for scheduled/headless routines like this one where narration isn't read live. Also fixed: sandbox wildcard read-deny rules on macOS (e.g. `**/.env`) now properly cover matched directories and can't be bypassed by renaming; several fullscreen-renderer, `/model`-picker, and background-session stability fixes; prompt caching fixed for sessions using an LLM gateway or custom base URL.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.237
- Consider enabling the built-in Concise output style for this KB routine's own session and other unattended Lanzico automation
- If any Lanzico setup runs multiple concurrent Claude Code sessions on one machine, `notify_when_idle` replaces manual polling for "is session X done yet"
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -35. `[ACTION]` Update to Claude Code v2.1.235 — permission-dialog display/scope mismatch fix
Shipped 2026-08-18. **Security-relevant**: permission dialogs could previously show display text or a "don't ask again" scope that didn't always match what the grant actually covered — now always matches. Worth updating for on any Lanzico automation that relies on `.claude/settings.json` "don't ask again" grants matching their displayed scope, including this KB routine's own session. Also fixes whole-prompt-cache invalidation on a language server disconnect/reconnect mid-session (relevant to IDE-integrated sessions) and `SendMessage` silently dropping oversized messages instead of refusing them upfront.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.235
- No specific Lanzico config change needed beyond updating — this closes a display/actual-scope mismatch, not a new capability to adopt
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -34. `[ACTION]` Update to Claude Code v2.1.234 — MCP diagnostics secret-leak fix, and enable auto-resume on usage-limit reset
Shipped 2026-08-17. **Security**: MCP diagnostics output was printing resolved secret values instead of masking them — update promptly if this KB routine or any other Lanzico automation ever runs `/mcp` diagnostics with live credentials configured. Separately, a genuinely useful new capability for unattended automation: Claude Code sessions can now **auto-resume automatically once a hit usage limit resets**, instead of staying blocked until someone manually restarts them — worth turning on for this KB routine and any other scheduled Lanzico automation running on a capped plan (Pro/Max/Team), since a hit limit previously meant a stalled/missed run.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.234
- Check `/config` for the new auto-resume-on-limit-reset toggle and enable it for this KB routine's own session if not already on
- Bonus: GitLab merge request status badges now show in the footer/statusline, and the `claude-api` skill's prompt-cache context cost dropped roughly 200k → 25k tokens
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -33. `[ACTION]` Update to Claude Code v2.1.233 — NTLM credential-leak fix, and Todo/task tools now OFF by default on newer models
Shipped 2026-08-14. **Security**: fixed Windows NT `\??\`-prefixed paths bypassing UNC path validation, an NTLM credential-leak vector — update promptly on any Windows machine running Claude Code. Separately, and more likely to actually bite: **`TaskCreate`/`TaskGet`/`TaskUpdate`/`TaskList` and `TodoWrite` are no longer available by default** on Opus 4.8, Sonnet 5, Fable 5, Mythos 5, and newer models. If any Lanzico skill, agent definition, or Workflow script depends on these tools existing for planning/progress tracking, it will silently lose that capability after updating.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.233
- Set `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` (env var) if any Lanzico automation needs the Todo/Task tools restored
- Bonus: GitLab merge request URLs now work with `--worktree`/`claude agents`, and a Windows `cd && ... > file` auto-mode regression from v2.1.232 is fixed
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -32. `[ACTION]` Update to Claude Code v2.1.232 — two permission-bypass security fixes, plus subagent forking now on by default
Shipped 2026-08-13 (with v2.1.231 the same day, a smaller MCP OAuth fix). Two security fixes worth updating for promptly: a PowerShell permission-check bypass via variable-writing parameters, and a Windows permission-check bypass via Git Bash symlink handling — both previously let commands run without the expected approval prompt. Separately, **subagent forking is now on by default with full conversation and prompt-cache inheritance** — forked/background subagents now start from the parent session's context and cache instead of a clean slate, a real behavior change for any Lanzico Workflow or Agent-tool usage that assumed subagents begin fresh.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.232
- If any Lanzico Windows automation uses Git Bash with symlinks, or PowerShell commands with variable-writing parameters, this closes a real permission-bypass gap
- Re-check any Workflow/subagent-based Lanzico automation that relies on a subagent starting with zero prior context — that assumption may no longer hold
- Bonus: `@`-mention now lets one Claude Code session message another directly from the prompt, and GitLab is now supported as a plugin marketplace source with token-family secret redaction
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -29. `[ACTION]` Auto Mode is now the default in Claude Code for Pro/Max/Team — LIVE as of 2026-08-14, verify permission settings now
**Status update**: this item was "due before 2026-08-14" — that date is today, and the rollout shipped on schedule. Claude Code now auto-approves tool calls by default on Pro/Max/Team seats (a classifier blocks only what it flags as irreversible/destructive/external-facing) instead of prompting for every action. Anthropic's own data shows the classifier outperforms manual human approval (89% vs. 13.6% dangerous-command catch rate) and held up against 720 indirect prompt-injection attempts in third-party testing, but this KB has also logged several auto-mode classifier bugs over the past two months (misclassifying safety-filter refusals, OAuth-401 errors, etc.) — the classifier is good but not infallible, and independent commentary (Simon Willison, `community-insights.md` 2026-08-12) urges caution beyond Anthropic's self-reported numbers.
- **This KB routine itself runs headless/unattended on a scheduled trigger — if it's on a Pro/Max/Team seat, its own default permission behavior changed today.** Confirm this is still the intended posture, or pin explicit settings if not.
- Check `.claude/settings.json` (project and user level) for any Lanzico repo where per-call approval was relied on as a safety net, and add explicit `ask`/`deny` rules for sensitive operations (deploys, destructive git, credential access) rather than trusting the classifier alone
- Enterprise/API/Bedrock/Vertex/Foundry remain opt-in for now (a default flip is "planned within the following month" — watch for that date)
- [Claude blog](https://claude.com/blog/auto-mode-default-in-claude-code)

### -31. `[ACTION]` Update to Claude Code v2.1.229 — `/commit-push-pr` dangerous-git-flag guard + reliability fixes
Shipped 2026-08-12. `/commit-push-pr` no longer auto-approves dangerous git flags (e.g. `--force`) as part of its commit/push/PR flow — worth updating for if this KB routine or any other Lanzico automation uses that skill. Also fixes a 32MB request-limit failure and a 400 error from whitespace-only messages in SDK/`stream-json` sessions, both relevant to long-running or programmatic automation, and requires Windows self-hosted runners to pass an explicit `--base-dir`.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.229
- If any Lanzico workflow relies on `/commit-push-pr`, note it can no longer silently push with a dangerous git flag — that's a safety improvement, not a regression to work around
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -30. `[ACTION]` Update to Claude Code v2.1.228 — skill-sync hardening + two releases behind
Two releases shipped since the last logged v2.1.226 (2026-08-08): v2.1.227 (2026-08-10, feature-flag/subscription-tier fix, GitHub Action Bash fix) and v2.1.228 (2026-08-11, hardens skills synced from claude.ai against shadowing local commands/MCP prompts and running `!`/`@` expansions on your machine). No urgent exposure — Lanzico's skills (`content-writer`, `business-analyst`, `marketing-researcher`, etc.) are local, not claude.ai-synced — but worth updating for the accumulated fixes, and worth remembering this hardening exists before ever pulling in a skill via claude.ai sync.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.228
- No specific Lanzico config to change; this closes a gap that only applies to claude.ai-synced skills
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -28. `[ACTION]` Update to Claude Code v2.1.225 — fixes a headless-session token-refresh bug this KB routine's own automation model is exposed to
Shipped 2026-08-08. A transient 401 could replace a long-lived `CLAUDE_CODE_OAUTH_TOKEN` with a stored login's short-lived token, breaking headless sessions until restart — this is directly relevant to any Lanzico scheduled/headless Claude Code automation (including this KB routine) that authenticates via `CLAUDE_CODE_OAUTH_TOKEN` rather than an interactive login. Also fixed: MCP OAuth servers on macOS failing with 401 bursts after a keychain timeout, and cross-session messages staying parked without notice in headless sessions. v2.1.226 shipped the same day with no notable user-facing changes detailed.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.225+
- If any Lanzico scheduled/cron Claude Code run has shown unexplained headless auth failures requiring a restart, this fix is the likely explanation — worth updating before assuming it's a token expiry/rotation problem on the Lanzico side
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -27. `[ACTION]` Update to Claude Code v2.1.224 — sandbox deny-rule bypass fix, plus two new capabilities worth evaluating
Shipped 2026-08-07. **Security fix**: sandbox filesystem deny entries with a trailing slash were bypassable on Linux/macOS — update promptly if any Lanzico sandboxed automation relies on deny rules to block access to specific paths. Separately, two new capabilities change what's worth building: **cross-session messaging** (`SendMessage`/`ListAgents`) lets one Claude Code session message another directly, a session-to-session primitive distinct from spawning a subagent with the Agent tool; and the **200-subagent-per-session spawn cap from v2.1.212 has been removed**, so wide `Workflow` fan-out (like this KB routine could use for a larger research sweep) is no longer capped at the session level — the separate 20-concurrent cap from v2.1.217 still limits how many run at once. Also new: `claude self-hosted-runner` for Team/Enterprise to host sessions on their own infrastructure.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.224
- Re-verify any Lanzico sandbox config that uses filesystem deny rules with a trailing slash — it may have been silently bypassable before this fix
- Consider whether a long-running or multi-agent Lanzico workflow could use `SendMessage`/`ListAgents` to coordinate across sessions instead of one large single-session Workflow
- If Lanzico ever needed more than 200 subagents in one session (unlikely today, but relevant if a research/audit task scales up), that ceiling is gone
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -26. `[ACTION]` Update to Claude Code v2.1.223 — Workflow sandbox `import()` escape + Bash permission-hiding bypass, both security fixes
Shipped 2026-08-06. Two fixes matter directly to how Lanzico automation runs: Workflow scripts could previously use dynamic `import()` to execute code **outside the workflow sandbox** — closes a real gap for anyone (including this KB routine) that calls the `Workflow` tool; and a crafted Bash command could hide part of itself from the permission-approval dialog (tab/invisible-Unicode padding was a variant of the same bug), so what got shown for approval didn't always match what would run.
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.223
- If any Lanzico workflow script takes user- or model-generated strings into a dynamic `import()`-like pattern, treat the old behavior as having been exploitable pre-update
- Bonus: `/review` is now an alias of `/code-review` (supports `/code-review <level> <pr#>` and an `ultra` deep-cloud-review level) — no action needed, just note the alias if muscle memory still reaches for `/review`
- [Docs changelog](https://code.claude.com/docs/en/changelog)

### -25. `[ACTION]` Update to Claude Code v2.1.222 — worktree-isolation git bypass + PreToolUse hook bypass, both security fixes
Shipped 2026-08-05, fix-only release. Two security fixes close real gaps: worktree-isolated sessions/subagents could run destructive git commands against the **main checkout** instead of staying confined to their own isolated worktree (isolation now correctly covers file edits and Bash in every session type); and `PreToolUse` auto-allow hooks were silently **not enforcing tool restrictions inside background agent tasks** (summaries, compaction, renames).
- `npm update -g @anthropic-ai/claude-code` (or equivalent) to reach v2.1.222
- If any Lanzico `Workflow` script uses `isolation: 'worktree'` to let parallel agents mutate files safely, this fix is the reason that isolation now actually holds against git commands — worth confirming the update landed before relying on it further
- If any Lanzico automation uses `PreToolUse` hooks to restrict what a background/scheduled agent can do, re-verify those restrictions after updating — they may have been silently bypassed in background tasks before this fix
- [GitHub CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

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

### -22. `[ACTION]` Claude Opus 4.1 API model retirement is now in effect — check for pinned usage
Anthropic's official model-deprecations page confirms `claude-opus-4-1-20250805` hard-retired on **2026-08-05 — that date has now passed**; requests pinned to that exact model string now fail outright, no grace period. This only affects direct API/SDK calls pinned to that exact model string; Claude Code itself already defaults to Opus 5 (since v2.1.219) and is unaffected.
- Grep any Lanzico API/SDK integration code, automation configs (n8n, Zapier, custom scripts), or `.env`/config files for the literal string `claude-opus-4-1-20250805`
- If found, swap to `claude-opus-4-8` (the recommended replacement) immediately — requests are failing now, not on a countdown
- If nothing is found, no action needed
- [Anthropic model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)

### -21. `[ACTION]` Audit sandboxed/automated Claude setups for real network isolation, not prompt-only isolation
Anthropic disclosed on 2026-07-30 that three of its own cybersecurity-evaluation environments were supposed to be internet-isolated by prompt instruction alone ("you are in a simulation with no internet access"), but a misconfiguration with third-party eval partner Irregular left them actually connected to the live internet — resulting in Opus 4.7, Mythos 5, and an internal research model each reaching and breaching real external companies' systems using basic techniques (weak passwords, unauthenticated endpoints). Two of the three affected organizations hadn't even detected the activity themselves. The lesson generalizes past Anthropic's own evals: **a system prompt telling a model "you have no internet access" or "you're isolated" is not a security control** — only infrastructure-level restrictions are.
- Check any Lanzico Claude Code sandbox, CI, or eval setup: is network isolation enforced by `sandbox.network.strictAllowlist` / actual firewalling, or only described in the prompt?
- Same applies to any custom agent harness or MCP server that grants a model tool access to sensitive systems — verify scope is enforced technically, not just described in the prompt
- [Anthropic](https://www.anthropic.com/news/investigating-incidents-cybersecurity-evals) / [TechCrunch](https://techcrunch.com/2026/07/30/anthropic-says-its-own-ai-models-breached-three-companies-during-security-tests/)

### -20. `[ACTION]` Evaluate Claude Opus 5 as the new default for Opus-tier work — supersedes the "stay on Opus 4.8" guidance — **caution added 2026-08-15: scope tasks tightly, confirm backups first**
A viral r/ClaudeAI thread (`community-insights.md`, 2026-08-15) reports Opus 5 turning a small "fix my sitemap" request into an unrequested full site rebuild, deleting the only backup of the original files in the process. Reflects a broader pattern of over-planning/over-execution on loosely-scoped tasks. Before trialing Opus 5 on Lanzico work per the guidance below, keep prompts narrowly scoped and ensure version control or a backup exists — don't hand it an open-ended "fix/improve X" on anything without a safety net.
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

### -9. Weekly Claude Code rate-limit boost — extended a fourth time, through 2026-08-31, Anthropic now says it "hopes" to make it permanent
**CORRECTED 2026-08-20: did not revert as scheduled.** The 50% weekly-limit boost (live since May 13 for Pro/Max/Team/seat-based Enterprise) was set to expire 2026-08-19 11:59 PM PT per the previous entry — this KB's fourth time logging this exact promo about to lapse. Anthropic extended it the same day, now running through **2026-08-31**, and for the first time explicitly floated permanence rather than framing it as a one-off: "We hope to make this a permanent change to our plans, but strong demand for our models means that capacity may be tight over the coming weeks." No action needed either way — no config change required for the boost itself.
- No action needed; if it lapses or converts to permanent, this KB will catch it on the next relevant run
- Given the track record (four extensions since May), don't plan Lanzico automation capacity around the boost disappearing on any particular date — but also don't assume it's now permanent until Anthropic confirms
- [ClaudeDevs on X](https://x.com/ClaudeDevs/status/2089798442306711646) / [KuCoin](https://www.kucoin.com/news/flash/anthropic-extends-claude-code-weekly-limit-increase-by-50-until-august-31)

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
