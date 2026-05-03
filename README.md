# protocols

Personal collection of reusable protocols, prompts, and patterns. This
is the meta-layer of the operator's work: cross-cutting conventions
applied across multiple projects, kept here so individual project
repos don't duplicate them.

## For LLMs opening this repo

If the operator has asked you to **work on the memory concept itself**
(refine the spec, log a fresh-session probe, queue meta-work),
start at `project-memory.md`. Active instances using it are listed
under "Active instances" below.

If the operator has asked you to **work inside a project**, you
usually don't open this repo first; the project's own `memory/`
walks you to the protocols it needs.

If a fetch fails (404, "URL not allowed", or similar), see
`fetching.md` before reporting failure.

### Direct entry URLs

If your fetch tool only follows URLs that appear in prior results,
seed your fetcher with these:

- https://raw.githubusercontent.com/iamweasel89/protocols/main/project-memory.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/launch-block.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/conversation-format.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/dates-discipline.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/mobile-app-android.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-landscape.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-scaling.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-queue.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/fetching.md?nocache=1

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
- **`memory-landscape.md`** — survey of similar approaches in the
  field (typed knowledge graphs, hierarchical memory, lazy context).
  The space moves fast; refresh periodically.
- **`memory-scaling.md`** — deferred reference for what to do as a
  project memory grows past ~30 nodes (staleness, duplicates,
  hidden content).
- **`memory-queue.md`** — cross-project queue of pending refinements
  to the memory concept itself.
- **`fetching.md`** — recovery techniques when GitHub fetch fails
  (Fastly cache, URL allowlists, blob-HTML extraction).

## Active instances

- [iamweasel89/birdscope](https://github.com/iamweasel89/birdscope) —
  uses `project-memory.md`, `launch-block.md`, `dates-discipline.md`,
  `mobile-app-android.md`, `fetching.md`.