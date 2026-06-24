# Best Practices

Newest entries first. Dated `(YYYY-MM-DD)` = date added to this KB.

## Context Management

- **Use `/btw` for side questions — now with history navigation** (2026-06-24) — runs the answer in a dismissible overlay that never enters conversation history. As of v2.1.187, ←/→ arrow keys step through earlier `/btw` answers. `[ACTION]` Use `/btw` instead of inline questions during long implementation sessions. [Best practices](https://code.claude.com/docs/en/best-practices) / [Changelog](https://code.claude.com/docs/en/changelog)
- **Selective compaction with `/compact <instructions>`** (2026-06-22) — e.g., `/compact Focus on the API changes`; preserves what matters and drops the rest. Encode compaction preferences in CLAUDE.md: `"When compacting, always preserve the full list of modified files and any test commands"`. [Best practices](https://code.claude.com/docs/en/best-practices)
- **`Esc+Esc` / `/rewind` checkpointing** (2026-06-22) — every prompt creates a snapshot; restore conversation only, code only, or both. Summarize from a checkpoint forward or backward to manage context without losing history. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Use subagents for investigation** (2026-06-22) — `"use subagents to investigate X"` delegates codebase reads to a separate context window, keeping your main session clean for implementation. Also useful for post-implementation verification with a fresh, unbiased context. [Best practices](https://code.claude.com/docs/en/best-practices)

## Verification & Quality

- **`/goal` condition for session-level checks** (2026-06-22) — a separate evaluator re-checks after every turn; Claude keeps working until the condition holds. Useful for unattended runs. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Stop hook as deterministic gate** (2026-06-22) — runs a check script and blocks the turn from ending until it passes (overrides after 8 consecutive blocks). Stronger than `/goal`; use for CI-style enforcement. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Adversarial review subagent** (2026-06-22) — after implementation, ask a subagent to review the diff against `PLAN.md` and report only gaps that affect correctness — not style. Avoids over-engineering from over-zealous review. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Writer/Reviewer parallel session pattern** (2026-06-22) — Session A implements; Session B reviews the diff in fresh context (no bias from seeing its own work). Repeat by feeding review back to Session A. `[ACTION]` Use this pattern for Lanzico feature PRs before merge. [Best practices](https://code.claude.com/docs/en/best-practices)

## Session Workflow

- **Let Claude interview you for large features** (2026-06-22) — start with a minimal prompt, ask Claude to interview you using `AskUserQuestion`. It surfaces edge cases and tradeoffs; write findings to `SPEC.md`; then start a fresh session to implement from the spec. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Name sessions and treat them like branches** (2026-06-22) — `/rename` in session; use `claude --continue` or `claude --resume` to pick up across days. One session per workstream. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Two failed corrections → `/clear` and better prompt** (2026-06-22) — context is polluted with failed approaches; a clean session with a sharper prompt almost always outperforms a long session with accumulated corrections. [Best practices](https://code.claude.com/docs/en/best-practices)

## Automation & Scaling

- **Fan-out with `claude -p` loops** (2026-06-22) — loop `claude -p "migrate $file"` over a file list with `--allowedTools` to scope permissions; test on 2-3 files first, then run at scale. [Best practices](https://code.claude.com/docs/en/best-practices)
- **Auto mode for unattended runs** (2026-06-22) — `claude --permission-mode auto -p "fix all lint errors"` — a classifier blocks risky actions and scope escalation without prompts. [Best practices](https://code.claude.com/docs/en/best-practices)
- **`Tool(param:value)` permission rules** (2026-06-22) — new fine-grained permission syntax in v2.1.178 lets you allow a tool only with specific parameter values. [Changelog](https://code.claude.com/docs/en/changelog)

## Project Setup (Previously Documented)

- **Plan before implementing** (2026-06-12) — use plan mode to separate research/exploration from execution; reviewing a spec before coding raises success rates substantially. [Official best practices](https://code.claude.com/docs/en/best-practices)
- **Demand evidence, not assertions** (2026-06-12) — have Claude show test output, command results, or screenshots instead of claiming success; reviewing evidence is faster than re-verifying yourself. [Medium](https://medium.com/data-science-collective/effective-claude-code-workflows-in-2026-what-changed-and-what-works-now-c93ebc6f8f50)
- **Maintain CLAUDE.md per project** (2026-06-12) — encode conventions, do/don't patterns, and key file references so every session starts informed. Keep under 200 lines; prune ruthlessly. `[ACTION]` Audit Lanzico repos for missing/stale CLAUDE.md files. [Official best practices](https://code.claude.com/docs/en/best-practices)
- **Save repeatable workflows as Skills** (2026-06-12) — refine a task once, save it as a skill, and Claude applies it consistently without re-explaining. [Firecrawl](https://www.firecrawl.dev/blog/best-claude-code-skills)
- **Lean on the harness, not manual context juggling** (2026-06-12) — plan mode, parallel exploration, persistent project memory, and long-session continuity replace hand-managed context. [Medium](https://medium.com/data-science-collective/effective-claude-code-workflows-in-2026-what-changed-and-what-works-now-c93ebc6f8f50)
