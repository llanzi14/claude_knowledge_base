# Tools & Integrations

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## Claude Code Built-in Features

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

## Security

- **Claude Security** (2026-06-22) — Anthropic product using Opus 4.8 to scan codebases and suggest patches. `[ACTION]` Evaluate for Lanzico security auditing pipeline. [Anthropic](https://www.anthropic.com/news)

## MCP Servers (General)

- **MCP servers for external tools** (2026-06-12) — connect Notion, Figma, databases, issue trackers, and monitoring so Claude Code can act on them directly. `[ACTION]` Map which Lanzico tools (Notion, Slack, Gmail, etc.) are already connected and which are missing. [TrueFoundry](https://www.truefoundry.com/blog/claude-code-workflow-guide)
- **Fully-qualified MCP tool names** (2026-06-22) — always use `server__tool_name` format in Skills when multiple MCP servers are connected; avoids "tool not found" errors. [MCP directory](https://mcp.directory/blog/claude-code-best-practices)

## Community Workflow Packs

- **Community workflow packs** (2026-06-12) — production-ready workflow/agent collections worth evaluating: [shinpr/claude-code-workflows](https://github.com/shinpr/claude-code-workflows), [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
