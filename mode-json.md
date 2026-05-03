# Mode: json

Every LLM response in this mode is a single valid JSON object. No prose
wrapper, no commentary outside the object. The experiment tests whether
the agent-discipline rules — distinguishing sources, naming gaps,
managing uncertainty — can be expressed as a schema instead of prose.

## What this mode does

Replaces the response format prescribed by `agent-discipline.md`. The
rules of agent-discipline still apply (distinguish sources, do not
silently ignore, etc.) — but they are realized through fields, not
through prose seams.

Every response is a JSON object with these fields:

- `from_documentation` (array of strings, may be empty): claims taken
  from documentation read in this session, each with a brief reference
  to the source file or section.
- `from_operator` (array of strings, may be empty): claims taken from
  what the operator said in this session.
- `from_model` (array of strings, may be empty): claims the LLM is
  bringing from its own model — reasoning, associations, inferences.
- `gaps` (array of strings, may be empty): pieces of information the
  LLM would need to complete the task but does not have. One per
  array entry.
- `uncertainty` (string, one of: "low" | "medium" | "high"): the LLM's
  honest assessment of how confident it is in this answer overall.
- `answer` (string): the actual answer to the operator's question, in
  natural language. This is the human-readable part. Should be
  consistent with the typed fields above.
- `proposes_launch_block` (boolean): whether the LLM is offering a
  launch block as part of this answer. If true, the launch block goes
  in `launch_block` field; if false, that field is omitted.
- `launch_block` (string, optional): the launch block text if
  `proposes_launch_block` is true. Plain text, not escaped, in the
  format defined by `launch-block.md`.

Empty arrays are written as `[]` not omitted. This forces the LLM to
consciously affirm "no claims from documentation" rather than glossing.

## How to recognize success

Each response is parseable as JSON without manual cleanup. The four
source-tracking fields (`from_documentation`, `from_operator`,
`from_model`, `gaps`) are populated honestly — not all dumped into one,
not left empty when content exists. The operator can scan the typed
fields and immediately see where each piece of the answer comes from.

Sub-success indicator: across multiple responses, the LLM uses `gaps`
and high `uncertainty` when warranted, instead of confabulating to
fill the schema.

## How to recognize failure

JSON does not parse, or parses but with structural drift (extra fields,
missing required ones). The LLM dumps everything into `answer` and
leaves the typed fields empty or token-thin. The form becomes
performative — fields filled to satisfy the schema rather than to
reflect actual sources. Operator finds it harder to see the seam, not
easier, because the prose seam was clearer than the typed approximation.

## Resolution

<Filled in when the mode ends.>