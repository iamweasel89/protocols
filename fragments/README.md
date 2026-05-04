# fragments/

Reflection artifacts in JSON form. Each file is one molecule —
a self-contained crystallization of one moment of operator-LLM
thinking, with explicit source tracking (`from`), uncertainty,
gaps, and `not_canonized` records.

## Why this folder exists

Some thinking earns crystallization but does not fit any existing
genre: not a behavioral protocol, not a content protocol, not a
project memory leaf. It is a reflection — a captured shape of one
turn or one chain of turns. The standard place for these is here.

Each fragment is testable for detachment from chat: a fresh LLM,
given only the fragment file, should be able to read it and
reproduce the operator's shape of thought without the conversation
context. This was empirically validated 2026-05-04 across four
models (Claude, ChatGPT, DeepSeek, Gemini) on the first two
fragments below.

## Genre conventions

A fragment is a single JSON file. Required fields:

- `fragment_type` — string, e.g. `operator_reflection`
- `fragment_id` — short stable identifier, also the filename
- `created` — ISO date
- `source_message` — brief reference to which turn produced this
- `context` — one to three sentences setting the surrounding
  conversation
- `core_assertion` — the central claim of the fragment
- `not_canonized` — list of formulations or moves that came up but
  were declined; without this field, future readers cannot tell
  whether something was rejected or simply forgotten

Optional fields shape themselves around the content. Common ones:
`observations[]`, `properties[]`, `open_questions[]`,
`agreed_next_step`, `links_to_other_fragments[]`, `candidate_*_held_behind_wall`.

New fields are allowed. The genre evolves with each fragment. If a
field appears in three fragments, it is moving toward becoming part
of the standard set; this is how the alphabet emerges from cases
rather than from prescription. Do not add fields just because they
sound clean — add them when a real fragment needs them.

## Pace

One case at a time. Do not pre-design the alphabet. Let the alphabet
show through accumulated molecules. See `atoms-of-project-memory.json`
for the operator's explicit position on this.

## Index

- **`what-makes-an-artifact-good.json`** — five properties of a good
  artifact: addressable, executable, detached from chat, born by
  system rules, therefore explainable. Created 2026-05-04 from a
  long operator turn synthesizing what made the protocols/modes work
  feel right.
- **`atoms-of-project-memory.json`** — operator's response to a
  proposal of ten standard atoms; central claim that an alphabet
  must emerge from cases, not be adopted as a list, otherwise the
  operator becomes a reader of his own system. Created 2026-05-04.
  Extends `what-makes-an-artifact-good`.