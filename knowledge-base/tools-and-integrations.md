# Tools & Integrations

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## API & SDK

- **`fallbacks` parameter (beta)** (2026-06-15) — Pass a `fallbacks` list in the Messages API to have the server automatically retry a Fable 5 refusal on another model. Available in beta on the Claude API and Claude Platform on AWS. `[ACTION]` Add to any Lanzico integration that calls Fable 5 directly. [Fallback docs](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)
- **SDK middleware (refusal handling)** (2026-06-15) — TypeScript, Python, Go, Java, and C# SDKs now include middleware for client-side fallback on Fable 5 refusals. Works on any platform. [SDK middleware docs](https://platform.claude.com/docs/en/cli-sdks-libraries/middleware)
- **`task-budgets-2026-03-13` header** (2026-06-15) — Beta header for Fable 5 and Mythos 5; sets a token/time cap on a task so subagent fan-out can't run away. [Task budgets docs](https://platform.claude.com/docs/en/build-with-claude/task-budgets)
- **Fallback credit** (2026-06-15) — When retrying a Fable 5 refusal on another model, the prompt-cache cost of the original call is refunded so you don't pay for caching twice. [Fallback credit docs](https://platform.claude.com/docs/en/build-with-claude/fallback-credit)
- **Context editing (beta)** (2026-06-15) — `context-management-2025-06-27` header on Fable 5 enables tool-result clearing through context editing; useful for long-running agentic sessions that accumulate large tool outputs. [Context editing docs](https://platform.claude.com/docs/en/build-with-claude/context-editing)

## Claude Managed Agents

- **Self-hosted sandboxes** (2026-06-15, public beta) — Anthropic orchestrates, you run the tool execution environment. Keeps code, filesystem, and network egress inside your perimeter. Supported infrastructure: Cloudflare Workers, Daytona, Modal, Vercel. `[ACTION]` Evaluate for Lanzico workflows touching private customer data. [Docs](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes) · [Blog](https://claude.com/blog/claude-managed-agents-updates)
- **MCP tunnels** (2026-06-15, research preview) — connect Managed Agents to private, on-prem MCP servers via encrypted tunnels. Request access through your Anthropic account team. [Blog](https://claude.com/blog/claude-managed-agents-updates)

## Claude Code integrations

- **VSCode Account & usage dialog** (2026-06-15, new in 2.1.174) — shows cache-miss rate, long-context events, and per-skill/agent/plugin token breakdowns. `[ACTION]` Review before the June 15 billing split to understand where tokens are going.
- **`enforceAvailableModels` managed setting** (2026-06-15, new in 2.1.175) — org admins can lock the Default model and prevent users/projects from expanding the model allowlist. Useful for cost governance.
- **`footerLinksRegexes` setting** (2026-06-15, new in 2.1.176) — configure regex patterns to add link badges to the footer; useful for project-specific links or dashboards.
- **Improved Bedrock credential caching** (2026-06-15, new in 2.1.176) — credentials now cached until expiration instead of 1-hour cap; removes frequent re-auth interruptions in long sessions.

## MCP servers

- **MCP servers for external tools** (2026-06-12) — connect Notion, Figma, databases, issue trackers, and monitoring so Claude Code can act on them directly. `[ACTION]` Map which Lanzico tools (Notion, Slack, Gmail, etc.) are already connected and which are missing. [TrueFoundry guide](https://www.truefoundry.com/blog/claude-code-workflow-guide)

## Community workflow packs

- **Production-ready workflow/agent collections** (2026-06-12) — worth evaluating: [shinpr/claude-code-workflows](https://github.com/shinpr/claude-code-workflows), [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
