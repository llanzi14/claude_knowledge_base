# Daily Routine Instructions

These instructions are for the scheduled run (daily, 07:00). Follow them exactly to keep runs cheap and the knowledge base consistent.

## Budget

- Keep each run short: **3–5 searches maximum** (Perplexity or web search).
- Prefer one broad query per category over many narrow ones.
- Skip a category entirely if yesterday's digest already covered it and nothing new is expected.
- Total new content per run should rarely exceed ~150 lines across all files.

## Procedure

1. Read the most recent file in `digests/` to know what is already covered. Do not re-research known items.
2. Search for updates since the last digest date. Suggested queries (adapt dates):
   - `Claude Code changelog <month> <year>` — releases and features
   - `Claude Code best practices OR workflow tips <month> <year>` — practices
   - `Claude MCP OR skills OR plugins new <month> <year>` — tools
   - One community sweep: `site:reddit.com/r/ClaudeAI OR r/ClaudeCode <topic>` only if a notable topic surfaced
3. Write `digests/YYYY-MM-DD.md`: 5–15 bullet points, each with a source link. If nothing new, write a one-line digest saying so — do not pad.
4. Update the relevant `knowledge-base/` files:
   - Add new entries **at the top** of the relevant section, dated `(YYYY-MM-DD)`.
   - Prune entries that are superseded or older than ~3 months unless still authoritative.
   - Tag items worth acting on at Lanzico with `[ACTION]`.
5. Commit with message `kb: daily update YYYY-MM-DD` and push.

## Quality rules

- Every claim gets a link. No link, no entry.
- Prefer official sources (docs, changelog, Anthropic blog) for facts; community sources for tips and sentiment.
- Curate, don't archive: the knowledge base is a "current state of knowledge," not a log. The log lives in `digests/`.
