# Hygiene log

Cross-project log of observations made during sessions: leaves that
look stale, overlap with others, or have grown beyond one topic.
Used by any project that follows `project-memory.md` and has
cross-project relevance. Project-local notes go in the project's
own `memory/hygiene.md` instead.

## How to use (LLM)

When you spot something during another task — do not interrupt the
task. Add one line below in the format:

`YYYY-MM-DD | <priority> | <leaf or file> | <observation in 10-20 words>`

Priority is one character: `!` critical (acts as filter at start-of-session),
`.` normal, `?` uncertain — observation might be a confabulation or might
not warrant action. Default to `.` if unsure; reserve `!` for things that
block ongoing work.

Examples:
- `2026-05-04 | . | n203 | references PhasePortraitView v3 params; code now on v4`
- `2026-05-04 | ! | memory-landscape | duplicate H-MEM section between n801 and memory-landscape after migration`
- `2026-05-05 | ? | n801.md | grew to cover 7 different approaches; split candidate`

## How to use (operator)

At session start, or whenever convenient: ask the LLM to read this
log and propose actions. The LLM will return a short list — what to
fix now, what to defer, what to dismiss. The operator decides; the
LLM applies via launch blocks.

Resolved entries get removed (not crossed out — keep the log light).
If patterns repeat across many entries, that is a signal to revise
the spec, not just the leaves.

## Entries

<!-- entries below this line; newest at top -->
