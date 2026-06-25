# Community Insights

Newest entries first. Findings from Reddit, YouTube, X/Twitter, and blogs. Treat as opinion until verified against official docs.

## Service & Incident History

- *(2026-06-24)* **Claude outage — June 23** — elevated error rate across all models, approximately 10:02 AM–12:44 PM ET; now resolved. [TechRadar](https://www.techradar.com/news/live/claude-down-june-23-2026)
- *(2026-06-24)* **Fable 5 & Mythos 5 still offline** — US export-control directive (June 12) remains in effect with no public restoration timeline. Community sites tracking status (e.g., `isfableback.org`). Main workaround: `claude-opus-4-8` or Sonnet 4.6. [Anthropic](https://www.anthropic.com/news/fable-mythos-access)

## Reddit (r/ClaudeAI, r/ClaudeCode)

- *(2026-06-22)* **r/ClaudeCode now at 4,200+ weekly contributors** — 3× the size of r/Codex; Claude Code is the most-discussed AI coding agent on Reddit. Community consensus on setups has consolidated significantly in 2026. [MorphLLM](https://www.morphllm.com/claude-code-reddit)
- *(2026-06-22)* **CLAUDE.md under 200 lines** — the single most cited rule; files longer than this cause Claude to drop instructions. Use hierarchical sub-files in nested directories for per-domain rules. [MorphLLM](https://www.morphllm.com/claude-code-reddit)
- *(2026-06-22)* **Hooks beat CLAUDE.md for determinism** — if an action must happen on every run with zero exceptions, a hook is the right tool; CLAUDE.md instructions are advisory. Community consensus. [MorphLLM](https://www.morphllm.com/claude-code-reddit)
- *(2026-06-22)* **Pro plan ceiling** — at $20/month, heavy users hit limits after ~12 prompts per session; community recommends the API plan for intensive coding work. `[ACTION]` Evaluate whether Lucas's usage pattern warrants switching to API billing.
- *(2026-06-22)* **Multi-file coherence praised** — multiple threads cite Claude correctly updating imports, type definitions, and tests across 10+ files in a single pass; especially noted for monorepos and enterprise apps. [MorphLLM](https://www.morphllm.com/claude-code-reddit)

## Workflow Patterns (Community-validated)

- *(2026-06-25)* **`/compact` at 50% context threshold** — community consensus on r/ClaudeCode: run `/compact` manually when context hits ~50% rather than waiting for auto-compaction near 70%. The 60-70% range is the "agent dumb zone" where performance visibly degrades. Use `/compact Focus on [topic]` to preserve the most relevant context. [DEV.to](https://dev.to/galian/claude-code-workflow-best-practices-that-ship-code-na)
- *(2026-06-25)* **Git worktrees for parallel Claude sessions** — run each Claude Code session in a separate `git worktree` checkout of the same repo; prevents state bleed and merge conflicts when two Claude sessions work in parallel. `[ACTION]` Standard pattern for any parallel Lanzico feature work. [DEV.to](https://dev.to/evan-dong/10-battle-tested-claude-code-practices-4n81)
- *(2026-06-25)* **"I Tried 100 Claude Skills" — curated skill guide** — DEV.to article benchmarking 100 community skills; top performers: code-review, generate-tests, explain-error, refactor-to-patterns, write-changelog. `[ACTION]` Review for Lanzico-applicable skills; cross-check against current `.claude/skills/` setup. [DEV.to](https://dev.to/suraj_khaitan_f893c243958/i-tried-100-claude-skills-these-are-the-best-1m4a)

- *(2026-06-22)* **Five-layer setup** — consensus best starting configuration: CLAUDE.md (project rules) → MCP servers (external tools) → Skills (reusable workflows) → Hooks (automation/safety) → Subagents (isolated research/review). [MCP directory](https://mcp.directory/blog/claude-code-best-practices)
- *(2026-06-22)* **Start CLAUDE.md simple, add rules iteratively** — don't pre-fill rules; let pain points drive additions. Review CLAUDE.md whenever Claude repeats a mistake. [MorphLLM](https://www.morphllm.com/claude-code-reddit)
- *(2026-06-22)* **Skills as "reusable thinking, not instructions"** — skills inject reasoning patterns, not just steps. Where CLAUDE.md tells Claude *what*, a skill tells Claude *how to think about a class of problems*. [MCP directory](https://mcp.directory/blog/claude-code-best-practices)
- *(2026-06-22)* **2026 shift framing** — community narrative: 2025 was "can AI write working code?" (yes, small things, sometimes); 2026 is "how do you orchestrate AI to ship production code?" The answer converges on: research → plan → execute → review → ship, human as gatekeeper at each step. [DEV.to](https://dev.to/galian/claude-code-workflow-best-practices-that-ship-code-na)
