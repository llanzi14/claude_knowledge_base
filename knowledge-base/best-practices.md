# Best Practices

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## Fable 5 specific

- **Use `effort` parameter to control cost on Fable 5** (2026-06-15) — Fable 5 has adaptive thinking always on; the `effort` parameter is the only lever for thinking depth and therefore cost. Set lower effort for simple tasks, higher for complex reasoning. `[ACTION]` Apply to any Lanzico Fable 5 API calls. [Effort docs](https://platform.claude.com/docs/en/build-with-claude/effort)
- **Set `thinking.display: "summarized"` for reasoning transparency** (2026-06-15) — Fable 5 never returns raw chain-of-thought; `"summarized"` returns a readable summary in thinking blocks instead of an empty field. Pass thinking blocks back unchanged in multi-turn conversations. [Adaptive thinking docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- **Gate subagent fan-out with a budget** (2026-06-15) — community reports of Max-20x users burning 5-hour windows in <10 minutes running 1,000+ subagents. Use `task-budgets-2026-03-13` header to set token/time budgets on Fable 5 tasks. `[ACTION]` Add budget parameters to any Lanzico dynamic workflow or nested subagent pipelines.
- **Handle `stop_reason: "refusal"` explicitly** (2026-06-15) — Fable 5 safety refusals return HTTP 200 with `stop_reason: "refusal"`, not an error. Add refusal handling and use the `fallbacks` parameter (or SDK middleware) for automatic retry on Opus 4.8. Not billing you for refused requests is the upside. [Refusals docs](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)

## General Claude Code

- **`!` prefix for instant bash injection** (2026-06-15) — prefixing a line with `!` in Claude Code runs it as shell and injects output into context without waiting for Claude to request it. Saves round-trips for status checks and quick lookups. [DEV Community tips](https://dev.to/oikon/24-claude-code-tips-claudecodeadventcalendar-52b5)
- **Hooks for lifecycle automation** (2026-06-15) — configure hooks in `.claude/settings.json` to run shell commands at specific Claude Code lifecycle events (pre-tool, post-tool, session-start, session-end). Use cases: block dangerous commands, auto-format after edits, auto-notify on completion. Available via `/hooks` command. [Official docs](https://code.claude.com/docs/en/best-practices)
- **`claude --dangerously-skip-permissions`** (2026-06-15) — auto-approves all actions; appropriate for fully trusted, isolated, repeatable workflows (e.g., CI). Never use on untrusted repos or production. [DEV Community tips](https://dev.to/oikon/24-claude-code-tips-claudecodeadventcalendar-52b5)
- **Plan before implementing** (2026-06-12) — use plan mode to separate research/exploration from execution; reviewing a spec before coding raises success rates substantially. [Official best practices](https://code.claude.com/docs/en/best-practices)
- **Demand evidence, not assertions** (2026-06-12) — have Claude show test output, command results, or screenshots instead of claiming success; reviewing evidence is faster than re-verifying yourself. [Medium — Effective Claude Code Workflows in 2026](https://medium.com/data-science-collective/effective-claude-code-workflows-in-2026-what-changed-and-what-works-now-c93ebc6f8f50)
- **Maintain CLAUDE.md per project** (2026-06-12) — encode conventions, do/don't patterns, and key file references so every session starts informed. `[ACTION]` Audit Lanzico repos for missing/stale CLAUDE.md files. [Official best practices](https://code.claude.com/docs/en/best-practices)
- **Save repeatable workflows as Skills** (2026-06-12) — refine a task once, save it as a skill, and Claude applies it consistently without re-explaining. [Firecrawl — Best Claude Code Skills](https://www.firecrawl.dev/blog/best-claude-code-skills)
- **Lean on the harness, not manual context juggling** (2026-06-12) — the 2026 shift: plan mode, parallel exploration, persistent project memory, and long-session continuity replace hand-managed context. [Medium](https://medium.com/data-science-collective/effective-claude-code-workflows-in-2026-what-changed-and-what-works-now-c93ebc6f8f50)

## Suggested new workflows for Lucas

- **Fable 5 for code review** (2026-06-15) `[ACTION]` — With ~80% SWE-Bench Pro, Fable 5 is significantly stronger than Opus 4.8 for complex code review. Create a `/code-review` skill that targets `claude-fable-5` explicitly with `effort: "high"` while it's free on the plan (before June 22).
- **Anthropic's official "Claude Code in Action" course** (2026-06-15) `[ACTION]` — Free course on Anthropic's learning platform; covers plan mode, sub-agents, MCP, and skills in depth. Recommended for any Lanzico team members onboarding to Claude Code. [Scrimba article](https://scrimba.com/articles/best-claude-code-tutorials-and-courses-in-2026/)
- **VSCode Account & usage dialog** (2026-06-15) — New in 2.1.174. Check this dialog to understand cache-hit rates and per-skill token consumption before the billing split makes those numbers cost real money.
