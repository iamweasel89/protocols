# protocols

Personal collection of reusable protocols, prompts, and patterns. This
is the meta-layer of the operator's work: cross-cutting conventions
applied across multiple projects, kept here so individual project repos
don't duplicate them.

## For LLMs opening this repo

If the operator has asked you to **work on the memory concept itself**
(refine the spec, log a fresh-session probe, queue meta-work),
start at `project-memory.md`. Active instances using it are listed
under "Active instances" below.

If the operator has asked you to **work inside a project**, you usually
don't open this repo first; the project's own `memory/` walks you to
the protocols it needs.

### Direct entry URLs

If your fetch tool only follows URLs that appear in prior results,
seed your fetcher with these:

- https://raw.githubusercontent.com/iamweasel89/protocols/main/project-memory.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/launch-block.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/conversation-format.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/dates-discipline.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/mobile-app-android.md

## Index

- **`project-memory.md`** — flat folder of JSON nav + markdown leaves;
  the format used by the operator's project memory. Read first when
  working on memory.
- **`launch-block.md`** — ready-to-run console command format. Read
  first when delivering changes to any project.
- **`conversation-format.md`** — numbered messages, real timestamps,
  language matching, English corrections, simplification levels.
- **`dates-discipline.md`** — honesty rules for any date written into
  a file; no invented timestamps.
- **`mobile-app-android.md`** — Kotlin Android app pattern: GitHub
  Actions builds the APK, app self-updates from GitHub Releases.

## Active instances

- [iamweasel89/birdscope](https://github.com/iamweasel89/birdscope) —
  uses `project-memory.md`, `launch-block.md`, `dates-discipline.md`,
  `mobile-app-android.md`.