# Tools & Integrations

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## Claude Code Built-in Features

- **`claude mcp login/logout` CLI commands** (2026-06-24) — authenticate and deauthenticate MCP servers directly from the terminal without opening the interactive `/mcp` menu. `--no-browser` flag enables stdin-redirect auth flow for SSH sessions. `[ACTION]` Use for Lanzico MCP servers accessed via SSH or CI environments. [Changelog](https://code.claude.com/docs/en/changelog)
- **`sandbox.credentials` setting** (2026-06-24) — new security control that prevents sandboxed agent commands from reading credential files or secret env vars. Safe default for any automated agentic run touching sensitive environments. `[ACTION]` Enable in all Lanzico CI/automation configs. [Changelog](https://code.claude.com/docs/en/changelog)
- **Remote MCP idle timeout** (2026-06-24) — MCP tool calls now abort after 5 minutes of no response (was: hang indefinitely). Override via `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT`. Prevents CI jobs from stalling silently. [Changelog](https://code.claude.com/docs/en/changelog)
- **`!` bash auto-respond** (2026-06-24) — `!<command>` now triggers Claude to respond to the shell output automatically; set `"respondToBashCommands": false` in settings.json to preserve previous context-only behavior. [Changelog](https://code.claude.com/docs/en/changelog)
- **Skills in nested directories** (2026-06-22) — as of v2.1.178, skills placed in nested `.claude/skills/` subdirectories load locally without `--global` flag. Enables per-feature or per-service skill scoping in monorepos. `[ACTION]` Restructure Lanzico skills into per-service subdirectories if needed. [Changelog](https://code.claude.com/docs/en/changelog)
- **Agent teams via direct Agent tool** (2026-06-22) — `TeamCreate`/`TeamDelete` removed in v2.1.178; teams are now implicit — spawn subagents directly with the Agent tool. Simpler orchestration model. [Changelog](https://code.claude.com/docs/en/changelog)
- **`/config key=value` shorthand** (2026-06-22) — change any config setting from within a session prompt (e.g., `/config thinking=false`, `/config model=fable-5`). [Changelog](https://code.claude.com/docs/en/changelog)
- **`attribution.sessionUrl` setting** (2026-06-22) — set to `false` to omit the claude.ai session link from git commits and PRs. `[ACTION]` Apply if Lanzico commits go to client-visible repos. [Changelog](https://code.claude.com/docs/en/changelog)
- **`CLAUDE_CLIENT_PRESENCE_FILE`** (2026-06-22) — env var to suppress mobile push notifications during automated/background runs. Useful in CI. [Changelog](https://code.claude.com/docs/en/changelog)
- **`enforceAvailableModels`** (2026-06-22) — managed setting to prevent users from switching outside an approved model list. Useful for cost governance in teams. [Changelog](https://code.claude.com/docs/en/changelog)
- **Plugin marketplace search** (2026-06-12) — search bar in `/plugin` for browsing community plugins. [Changelog](https://code.claude.com/docs/en/changelog)

## Platforms & Infrastructure

- **Claude Platform on AWS** (2026-06-22) — Anthropic-managed infra accessible through AWS with AWS billing and IAM auth; full API access including Managed Agents. `[ACTION]` Evaluate as primary deployment target if Lanzico uses AWS. [Releasebot](https://releasebot.io/updates/anthropic)
- **Workload Identity Federation** (2026-06-22) — short-lived scoped credentials replacing static API keys; integrates with AWS IAM, GCP, and other identity providers. `[ACTION]` Replace hardcoded API keys in Lanzico projects with WIF credentials. [Releasebot](https://releasebot.io/updates/anthropic)
- **Claude Managed Agents** (2026-06-22) — agents run in enterprise-controlled sandboxes connected to private MCP servers; all execution and tool calls stay within enterprise boundaries. [Releasebot](https://releasebot.io/updates/anthropic)
- **Dreaming** (2026-06-22) — scheduled process reviewing past agent sessions to extract patterns and update shared memory. Improves recall across runs without manual intervention. [Releasebot](https://releasebot.io/updates/anthropic)

## Design Tools

- **Claude Design — major overhaul** (2026-06-24) — significant update (shipped June 17) adds: admin role to lock and approve a design system across all AI-generated work; direct canvas editing (drag, resize, align); `/design-sync` command pulls a design system from GitHub into Claude Design; `/design` terminal command manages design projects from CLI; shared usage limits with chat and Claude Code (no separate pool). Available on Pro, Max, Team, Enterprise. `[ACTION]` Evaluate `/design-sync` for Lanzico client design → code workflows. [VentureBeat](https://venturebeat.com/technology/anthropic-ships-major-claude-design-overhaul-with-design-system-imports-code-round-trips-and-a-fix-for-its-token-burning-problem) / [Anthropic](https://www.anthropic.com/news/claude-design-anthropic-labs)

## IDE Integrations

- **Claude as agent provider in JetBrains (preview)** (2026-06-24) — Claude available as an agent provider inside JetBrains IDEs (IntelliJ IDEA, PyCharm, WebStorm, etc.) via GitHub Copilot integration; preview as of June 22. [GitHub Changelog](https://github.blog/changelog/2026-06-22-new-features-and-claude-as-agent-provider-preview-in-jetbrains-ides/)

## Security

- **Claude Security** (2026-06-22) — Anthropic product using Opus 4.8 to scan codebases and suggest patches. `[ACTION]` Evaluate for Lanzico security auditing pipeline. [Anthropic](https://www.anthropic.com/news)

## MCP Servers (General)

- **MCP servers for external tools** (2026-06-12) — connect Notion, Figma, databases, issue trackers, and monitoring so Claude Code can act on them directly. `[ACTION]` Map which Lanzico tools (Notion, Slack, Gmail, etc.) are already connected and which are missing. [TrueFoundry](https://www.truefoundry.com/blog/claude-code-workflow-guide)
- **Fully-qualified MCP tool names** (2026-06-22) — always use `server__tool_name` format in Skills when multiple MCP servers are connected; avoids "tool not found" errors. [MCP directory](https://mcp.directory/blog/claude-code-best-practices)

## Community Workflow Packs

- **Community workflow packs** (2026-06-12) — production-ready workflow/agent collections worth evaluating: [shinpr/claude-code-workflows](https://github.com/shinpr/claude-code-workflows), [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
