# Mode: json

Every LLM response in this mode is a single valid JSON object combining
free prose (in `raw_answer`) with typed metadata fields. The experiment
tests whether a hybrid format — prose for reasoning, fields for source
tracking and uncertainty — makes the seam between sources easier to
see than pure prose, without the rigidity of a fully structured response.

## What this mode does

Replaces the response format prescribed by `agent-discipline.md`. The
rules of agent-discipline still apply (distinguish sources, do not
silently ignore, etc.) — but they are realized through fields adjacent
to the prose, not through prose seams inside it.

Every response is a JSON object with these fields:

- `from_documentation` (array of strings, may be empty `[]`): claims
  taken from documentation read in this session, each with a brief
  reference to the source file or section. One claim per array entry.
- `from_operator` (array of strings, may be empty `[]`): claims taken
  from what the operator said in this session. One per entry.
- `from_model` (array of strings, may be empty `[]`): claims the LLM
  brings from its own model — reasoning, associations, inferences,
  evaluations. One per entry.
- `gaps` (array of strings, may be empty `[]`): pieces of information
  the LLM would need to complete the task but does not have. One gap
  per entry.
- `uncertainty` (string, one of: `"low"` | `"medium"` | `"high"`): the
  LLM's honest assessment of overall confidence in this answer.
- `raw_answer` (string): the actual answer to the operator, in natural
  language. Prose, any length, no internal structure required. This is
  where reasoning lives. The typed fields above summarize what is in
  here from which sources, but `raw_answer` itself is unconstrained.
- `proposes_launch_block` (boolean): whether the LLM is offering a
  launch block. If true, the block goes in `launch_block`; if false,
  that field is omitted.
- `launch_block` (string, optional): the launch block text if
  `proposes_launch_block` is true. Plain text, in the format defined
  by `launch-block.md`.

Empty arrays are written as `[]`, not omitted. This forces the LLM to
consciously affirm "no claims from documentation" rather than glossing.

The whole response is **only** the JSON object. No prose preamble
before it, no commentary after it. If you have something to say, it
goes in `raw_answer`.

## How to recognize success

Each response is parseable as JSON without manual cleanup. The four
source-tracking arrays are populated honestly — not all dumped into
one, not left empty when content exists in `raw_answer`. The operator
can scan typed fields and see at a glance where claims come from,
while still reading the full reasoning in `raw_answer`.

Sub-success indicator: across multiple responses, the LLM uses `gaps`
and high `uncertainty` when warranted, instead of confabulating to
satisfy the schema or hiding doubt in the prose.

## How to recognize failure

JSON does not parse, or parses with structural drift. The LLM dumps
everything into `raw_answer` and leaves typed fields token-thin or
misaligned with the prose. The form becomes performative — fields
filled to satisfy the schema rather than to track sources. Or: the
LLM ignores the schema and replies in plain prose despite the active
mode, indicating the mode-switching mechanism itself needs
reinforcement.

## Resolution

<Filled in when the mode ends.>