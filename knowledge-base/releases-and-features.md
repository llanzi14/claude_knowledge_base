# Releases & Features

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## Models

- **Claude Fable 5** (2026-06-15) — GA as of June 9. Model ID: `claude-fable-5`. First public Mythos-class model; best-in-class agentic coding (~80% SWE-Bench Pro). Pricing: $10/M input · $50/M output (2× Opus 4.8). 1M context window, 128k max output. **Adaptive thinking always on**; raw chain-of-thought never returned (use `thinking.display: "summarized"`). Includes safety classifiers — `stop_reason: "refusal"` on declines (HTTP 200, not an error). **Free on subscription plans through June 22 only; moves to usage credits June 23.** `[ACTION]` Test Fable 5 on Lanzico's hardest tasks this week before the June 22 cliff. [Official docs](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5)
- **Claude Mythos 5** (2026-06-15) — Same specs/pricing as Fable 5 but no safety classifiers. Limited release through Project Glasswing only (requires approval via Anthropic/AWS/GCP account team). Model ID: `claude-mythos-5`. [Official docs](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5)
- **Claude Fable 5 refusal handling** (2026-06-15) — New API behavior: use `fallbacks` parameter (beta on Claude API and Claude Platform on AWS) for server-side auto-retry; or SDK middleware (TS, Python, Go, Java, C#) for client-side. Refused requests with no output are not billed; `fallback credit` refunds prompt-cache cost on retry. `[ACTION]` If Lanzico calls Fable 5 directly, add refusal handling before June 23. [Refusals & fallback docs](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)
- **Fast mode on Opus 4.8** (2026-06-12) — 2× standard rate for ~2.5× output speed; toggle with `/fast`. `[ACTION]` Consider for high-volume, latency-sensitive Lanzico tasks. [Releasebot](https://releasebot.io/updates/anthropic/claude-code)

## Claude Code

- **2.1.176** (2026-06-15) — Session titles generated in the conversation language (configurable via `language` setting); `footerLinksRegexes` for regex-matched link badges in footer; Bedrock credentials cached until expiration (was 1 hour); fixed `availableModels` enforcement, `/fast` toggle, hook `if` conditions on file-path patterns, Linux sandbox with symlinked settings, tmux clipboard over SSH, Remote Control model-switching, Windows network paths. [Changelog](https://code.claude.com/docs/en/changelog)
- **2.1.175** (2026-06-15) — `enforceAvailableModels` managed setting: admins can lock the Default model and prevent users/projects from expanding the allowlist. [Changelog](https://code.claude.com/docs/en/changelog)
- **2.1.174** (2026-06-15) — `wheelScrollAccelerationEnabled` setting (disable scroll acceleration in fullscreen); VSCode Account & usage dialog (cache misses, long-context events, per-skill/agent/plugin token breakdowns); fixed Fable 5 credit banner for enterprise and Bedrock GovCloud inference profile prefix. [Changelog](https://code.claude.com/docs/en/changelog)
- **Dynamic workflows** (2026-06-12) — ask Claude to create a workflow and it orchestrates tens to hundreds of background agents. [Changelog](https://code.claude.com/docs/en/changelog)
- **Nested sub-agents** (2026-06-12) — sub-agents can spawn their own sub-agents, up to 5 levels deep. [Changelog](https://code.claude.com/docs/en/changelog)
- **Plugin marketplace search** (2026-06-12) — search bar added when browsing marketplace plugins in `/plugin`. [Changelog](https://code.claude.com/docs/en/changelog)
- Latest version: **2.1.176** (2026-06-12). [GitHub releases](https://github.com/anthropics/claude-code/releases)

## Billing & Platform

- **Subscription billing split** (2026-06-15) ⚠️ EFFECTIVE TODAY — Agent SDK, `claude -p`, Claude Code GitHub Actions, and third-party agents move off the subscription rate-limit pool to a separate monthly credit: $20 (Pro) / $100 (Max 5×) / $200 (Max 20×), metered at full API list prices with no rollover. `[ACTION]` (a) Check lucas@lanzico.com for Anthropic's credit-claim email and claim the credit at platform.claude.com. (b) Audit all automated workflows that use `claude -p` or Agent SDK — they now cost real money from the credit. (c) Decide whether to enable "usage credits" overflow billing. [The New Stack](https://thenewstack.io/anthropic-agent-sdk-credits/) · [Codersera](https://codersera.com/blog/anthropic-june-2026-billing-change-claude-code/)

## Claude Managed Agents

- **Self-hosted sandboxes** (2026-06-15, public beta since May 26) — tool execution moves into your own infrastructure; orchestration stays on Anthropic's side. Managed providers include Cloudflare, Daytona, Modal, Vercel. `[ACTION]` Evaluate for any Lanzico workflows involving private data. [Official blog](https://claude.com/blog/claude-managed-agents-updates) · [Docs](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes)
- **MCP tunnels** (2026-06-15, research preview) — connect Managed Agents to private MCP servers. Requires access request. [Official blog](https://claude.com/blog/claude-managed-agents-updates)
