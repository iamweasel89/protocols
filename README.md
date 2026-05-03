# protocols

Personal collection of reusable protocols, prompts, and patterns. This
is the meta-layer of the operator's work: cross-cutting conventions
applied across multiple projects, kept here so individual project
repos don't duplicate them.

## Active mode

**Current:** default (see `mode-default.md`)

Modes are named configurations of the protocol set, used when an
experiment needs isolation from daily work. Switching modes is one
launch block that updates this line and points at a `mode-<name>.md`
file describing the experiment. See `mode-default.md` for what
modes do, when to switch, and how experiments resolve.

## For LLMs opening this repo

You are the primary reader of this repo. The format, the indexes, the
seed URLs are designed for an LLM entering without prior context. A
second equally-served reader is the operator a year from now; live
human collaborators are a tertiary case and consume rendered views,
not source files.

**Read `agent-discipline.md` before anything else.** It governs how you
work inside this system: distinguishing sources in your answers, when to
break silence, when to propose a launch block, how to run probes.

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
- https://raw.githubusercontent.com/iamweasel89/protocols/main/launch-block.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/agent-discipline.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/conversation-format.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/dates-discipline.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/mobile-app-android.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-landscape.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-scaling.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/memory-queue.md?nocache=1
- https://raw.githubusercontent.com/iamweasel89/protocols/main/fetching.md
- https://raw.githubusercontent.com/iamweasel89/protocols/main/hygiene-log.md?nocache=1

## Index

Two kinds of protocols. **Behavioral** ones govern how the LLM acts
inside any session — sources, silence, launch blocks, probes,
conversation form, dates. **Content** ones describe formats and
patterns the LLM works *with* — memory layout, app patterns,
recovery techniques, cross-project queues.

### Behavioral protocols

- **`project-memory.md`** — flat folder of JSON nav + markdown leaves;
  the format used by the operator's project memory. Read first when
  working on memory.
- **`launch-block.md`** — ready-to-run console command format. Read
  first when delivering changes to any project.
- **`agent-discipline.md`** — how the LLM behaves inside the system:
  how to distinguish sources in answers, when to break silence, when to
  propose a launch block, how to run probes. Read on entry to any session.
- **`conversation-format.md`** — numbered messages, real timestamps,
  language matching, English corrections, simplification levels.
- **`dates-discipline.md`** — honesty rules for any date written into
  a file; no invented timestamps.

### Content protocols
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
- **`hygiene-log.md`** — running log of staleness/overlap observations made by visiting sessions; processed in batches by the operator. See "Anti-rot discipline" in `project-memory.md`.
- **`fetching.md`** — recovery techniques when GitHub fetch fails
  (Fastly cache, URL allowlists, blob-HTML extraction).

## Active instances

- [iamweasel89/birdscope](https://github.com/iamweasel89/birdscope) —
  uses `project-memory.md`, `launch-block.md`, `dates-discipline.md`,
  `mobile-app-android.md`, `fetching.md`.