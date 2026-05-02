# Project memory: flat folder of JSON nav + markdown leaves

A reusable pattern for keeping project memory across LLM sessions.
The memory is a flat folder of files. JSON files describe meaning and
navigation. Markdown files hold content. Any LLM can open the folder,
read the root, descend by purpose, and continue work without prior chat.

Used by: birdscope.

## Why

LLM chat is linear and forgets. Project state — what was decided,
what is in code, what is deferred, what was dropped — leaks out of
the chat over time. By the next session it has to be re-explained.

Standard fixes (single README, scattered ADRs, an Obsidian vault)
either bloat into one unreadable file or scatter without navigation.
This pattern keeps storage flat, navigation cheap, and content
descriptive — and makes the navigation tell the LLM **why** to read
each file before reading it.

## Principles

1. **Flat storage.** All files in one folder. No subfolders. Hierarchy
   is meaning-based, not path-based.

2. **IDs as filenames, not meanings.** Files are named `n001.json`,
   `n023.md`, etc. Meaning lives in fields inside the file. This keeps
   the spec portable across projects: any project's branches have
   different names, but the structure stays the same.

3. **Two file kinds.** JSON files are **branches** (metadata + links
   to other nodes). Markdown files are **leaves** (actual content,
   with YAML frontmatter holding metadata).

4. **Reference by ID with explicit extension.** Each `ref` value
   includes the file extension (`n023.json` or `n100.md`), not just
   the bare ID. Renames don't break links because the ID part stays
   stable, but the extension is required so any LLM fetching the file
   over HTTP gets it on the first try, without guessing whether to
   try `.json` or `.md`.

5. **Every reference carries a purpose.** Each link inside a JSON
   includes a `purpose` field — a short description of why this child
   exists. The LLM reads the purpose and decides whether to descend,
   without opening the child file. This is the laziness mechanism.

6. **Source is machine-readable, view is human-readable.** The files
   are for the LLM. The operator looks at a **rendered view** (graph,
   tree, index) generated from the files. Names in the view come from
   the `name` field, not from filenames.

7. **Single root.** The folder always contains exactly one
   `root.json` with `id: n000`. This is the entry point for any LLM
   opening the project.

8. **README points to memory.** The repo's main README must tell any
   LLM that project state lives in `memory/`, with `memory/root.json`
   as the entry point. Without this pointer, the memory folder is
   discoverable only by guessing.

## File shapes

### root.json

```json
{
  "id": "n000",
  "name": "<project name>",
  "purpose": "root node; entry point for any LLM",
  "kind": "branch",
  "children": [
    {
      "ref": "n001.json",
      "name": "<branch name>",
      "purpose": "<one-line: why descend here>"
    }
  ]
}
```

### Internal branch (any JSON other than root)

Same shape as root, with its own `id` and `purpose`. Children may
reference further branches (`.json`) or leaves (`.md`); the extension
in `ref` tells the reader which.

### Leaf (markdown with YAML frontmatter)

```markdown
---
id: n100
name: <leaf name>
purpose: <why this leaf exists>
kind: leaf
status: planned | building | done | dropped | open | deferred
code_ref: <optional: path/commit>
last_verified: <optional: YYYY-MM-DD>
review_after: <optional: YYYY-MM-DD>
review_when: <optional: free-form trigger condition>
---

# <Heading>

<Free content: spec, passport, notes, decision record.>
```

## Required fields

- All files: `id`, `name`, `purpose`, `kind`.
- Branches: `children` (may be empty array).
- Leaves: `status`.
- Each entry in `children`: `ref` (with extension), `name`, `purpose`.

## Optional leaf fields

- `code_ref` — path, commit hash, build tag, or GitHub URL pointing
  to the realized code. Format is project-local; document the choice
  in a project-side leaf (see "Project-local rules" below).
- `last_verified` — date the leaf was last checked against reality
  (code, decisions, the world). Lets the operator find stale leaves
  without reading their bodies. Recommended once a project crosses
  ~30 leaves; optional below that.
- `review_after` — a date after which the leaf should be revisited.
  When the date passes, the leaf is treated as "due for review."
  Useful for deferred decisions and reference material that may go
  out of date.
- `review_when` — free-form trigger condition (e.g. "memory has 30+
  nodes", "after build-20"). Read by the LLM, evaluated heuristically.

`last_verified` answers "is this still true?" `review_after` and
`review_when` answer "should I look at this again?" They serve
different purposes and can coexist.

**All date fields follow `dates-discipline.md`** (separate protocol):
never invent, source must be checkable, pair with a review plan if
staleness matters.

## Minimal example

```
root.json:    n000 -> [n001.json (UI), n002.json (data)]
n001.json:    n001 (UI) -> [n100.md (Main window, done)]
n002.json:    n002 (data) -> [n200.md (Pipeline, building)]
n100.md:      ---id: n100, status: done--- # Main window ...
n200.md:      ---id: n200, status: building--- # Pipeline ...
```

Six files, flat folder. Operator sees a tree (rendered). LLM walks
JSON, descends only by purpose, reads only needed leaves, and uses
the explicit extension in each `ref` to fetch the right file.

## How the LLM uses it

1. Read `root.json` — see the project's branches.
2. For each branch in `children`, read its `purpose`. Decide which
   to descend into.
3. Open the chosen child by `ref` (which includes the extension).
   If the extension is `.json`, repeat step 2; if `.md`, read frontmatter
   and then body if needed.

A small task may need only the root. A focused task — one branch
walk. Full read of all files is rare and usually unnecessary.

## How the operator uses it

The operator does not work with raw files. The operator looks at a
**rendered view** (graph, index, tree) generated from the JSONs.
Nodes are labeled by `name`. Links follow `ref -> child`. Tapping a
node tells the LLM to operate on that `id`. The operator uses
human-readable names; the LLM uses IDs. Each side speaks its own
language; the substrate keeps them in sync.

## Crystallization on demand

Nodes are created when the operator decides to fix something — not
automatically, not on every commit. The LLM does not propose to
crystallize. It is ready to do so quickly when asked. This keeps the
memory thin: only what was worth fixing gets fixed. The rest stays
in code, in chat, in the operator's head — as raw material.

## Purpose discipline

The `purpose` field is the laziness mechanism. The LLM reads it and
decides whether to descend. If purposes are weak, the whole memory
loses its efficiency: the LLM either skips nodes it should read, or
opens nodes it does not need.

Writing good purposes is a skill, not a one-shot task. The LLM cannot
fully enforce it on its own — it can write reasonable text, but it
will drift in style and miss staleness without external pressure.

### Universal rules

These apply across projects.

**Tell when to read, not what is inside.**
A purpose is a trigger condition for the reader, not a label of the
content. Bad: `"FFT analyzer"`. Good: `"FFT pipeline; read when
changing window size or output format"`.

**One sentence, under 20 words.**
Longer purposes drift into content. If more is needed, it belongs
inside the leaf, not in the link.

**Do not duplicate the `name` field.**
The reader already saw `name`. Repeating it in `purpose` wastes the
slot.

**Mention the kind of work, not just the topic.**
The reader filters by what they are doing — designing, deciding,
implementing, debugging — not only by domain.

**Check for duplicates before creating a node (DRY).**
Before creating a leaf about topic X, scan for existing leaves on the
same topic. Two parallel leaves split attention and confuse navigation.

### Project-local rules

Each project develops conventions specific to its domain. These
should live in a leaf node inside the project's memory (e.g.
`n802 — Purpose discipline`), not in this spec. New rules graduate
to the spec only when they prove universal across projects.

## Triggered nodes

A leaf can include `review_after` or `review_when` in its frontmatter,
and the same trigger should be echoed in the parent branch's `purpose`
so an LLM walking the tree sees it without opening the leaf.

Example parent purpose: `"reference for what to do as memory grows;
revisit when memory has 30+ nodes or after 2026-08-31"`.

While only one or two triggered nodes exist, this in-place pattern is
sufficient. When triggers reach three or more — or any becomes critical
— introduce a centralized **trigger registry** near the root
(e.g. `triggers.md`), so any LLM entering memory sees overdue items
at once. Until then, the registry is unnecessary overhead.

## What this spec deliberately does not include

- Node subtypes inside leaves (decision / feature / question /
  protocol). Add as a `type` field when needed.
- Cross-branch links beyond parent->child (alternative_to, supersedes,
  depends_on). Add as a `links` array in leaf frontmatter when needed.
- Costs, weights, tags. Add to frontmatter when needed.
- Renderer. Several formats are valid (Mermaid block, generated
  graph.html, Obsidian Canvas). Out of scope here.
- Trigger registry format. To be specified when first project needs it.

The spec is intentionally minimal. Extensions are introduced from
practice, not anticipated upfront.

## Common gotchas

- **One root only.** Two `root.json` files in the same folder break
  the entry-point assumption.
- **Don't put meaning in filenames.** `main-menu.json` looks helpful
  but ties the spec to a project type. Use `n023.json` and put
  `"name": "Main menu"` inside.
- **Don't omit `purpose` on links.** A link without `purpose` forces
  the LLM to open the child file to know if it's relevant. That breaks
  the laziness mechanism.
- **Don't omit the extension on `ref`.** A `ref` without extension
  forces the reader to guess `.json` vs `.md`. Confirmed in practice
  to cause failed fetches even when the LLM is otherwise reading
  carefully.
- **Leaves are markdown, not JSON.** The frontmatter does the work a
  separate JSON file would do. Don't create paired `n100.json` +
  `n100.md` — it's redundant.
- **README must point to `memory/`.** The repo's top-level README is
  the discovery path. If it doesn't say "state lives in memory/",
  an LLM opening the repo has to guess.
- **Triggered nodes need their trigger in the parent purpose.**
  If the trigger lives only inside the leaf's frontmatter, an LLM
  walking the tree by `purpose` will not see it and will not open
  the leaf when due.
- **Dates inside leaves must follow dates-discipline.md.**
  No invented timestamps; checkable source; staleness needs a plan.
- **UTF-8 BOM breaks Android resource files.** When generating XML
  from PowerShell, write without BOM (`UTF8Encoding($false)`).
  Not specific to memory format, but bites in practice.
