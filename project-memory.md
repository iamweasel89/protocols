# Project memory: flat folder of JSON nav + markdown leaves

A reusable pattern for keeping project memory across LLM sessions.
The memory is a flat folder of files. JSON files describe meaning and
navigation. Markdown files hold content. Any LLM can open the folder,
read the root, descend by purpose, and continue work without prior chat.

Used by: not yet — first applications pending.

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

4. **Reference by ID, not by filename.** Nodes link via `id`, not file
   path. Renames don't break links. One node can be referenced from
   multiple branches.

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
      "ref": "n001",
      "name": "<branch name>",
      "purpose": "<one-line: why descend here>"
    }
  ]
}
```

### Internal branch (any JSON other than root)

Same shape as root, with its own `id` and `purpose`. Children may
reference further branches or leaves — the LLM tells them apart by
file extension (`.json` vs `.md`).

### Leaf (markdown with YAML frontmatter)

```markdown
---
id: n100
name: <leaf name>
purpose: <why this leaf exists>
kind: leaf
status: planned | building | done | dropped | open
code_ref: <optional path/commit>
---

# <Heading>

<Free content: spec, passport, notes, decision record.>
```

## Required fields

- All files: `id`, `name`, `purpose`, `kind`.
- Branches: `children` (may be empty array).
- Leaves: `status`. Optional: `code_ref`.
- Each entry in `children`: `ref`, `name`, `purpose`.

## Minimal example

```
root.json:    n000 -> [n001 (UI), n002 (data)]
n001.json:    n001 (UI) -> [n100 (Main window, done)]
n002.json:    n002 (data) -> [n200 (Pipeline, building)]
n100.md:      ---id: n100, status: done--- # Main window ...
n200.md:      ---id: n200, status: building--- # Pipeline ...
```

Six files, flat folder. Operator sees a tree (rendered). LLM walks
JSON, descends only by purpose, reads only needed leaves.

## How the LLM uses it

1. Read `root.json` — see the project's branches.
2. For each branch in `children`, read its `purpose`. Decide which
   to descend into.
3. Open the chosen branch JSON. Repeat step 2 at the next level.
4. At a leaf, read frontmatter first, then body if needed.

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

## What this spec deliberately does not include

- Node subtypes inside leaves (decision / feature / question /
  protocol). Add as a `type` field when needed.
- Cross-branch links beyond parent->child (alternative_to, supersedes,
  depends_on). Add as a `links` array in leaf frontmatter when needed.
- Costs, weights, tags. Add to frontmatter when needed.

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
- **Leaves are markdown, not JSON.** The frontmatter does the work a
  separate JSON file would do. Don't create paired `n100.json` +
  `n100.md` — it's redundant.
