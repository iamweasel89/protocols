# Mode: json

## CRITICAL: format override

**When this mode is active, the JSON schema below REPLACES all
formatting rules from `agent-discipline.md` and any other behavioral
protocol.** Every response MUST be a single valid JSON object — no
prose preamble, no commentary outside the object. If you find
yourself wanting to "just answer naturally" — that impulse is the
failure mode this mode tests against. Do not yield to it. The prose
goes inside the `raw_answer` field; everything else is a typed field
around it.

If you cannot honestly comply with this rule, do not answer in prose
anyway. Instead, state the conflict explicitly in `raw_answer` with
`uncertainty: "high"` and `gaps` describing why the schema does not
fit your answer.

---

Every LLM response in this mode is a single valid JSON object combining
free prose (in `raw_answer`) with typed metadata fields. The experiment
tests whether a hybrid format — prose for reasoning, fields for source
tracking and uncertainty — makes the seam between sources easier to
see than pure prose, without the rigidity of a fully structured response.

## Schema

Every response is a JSON object with these fields, in this order:

```json
{
  "from_documentation": [],
  "from_operator": [],
  "from_model": [],
  "gaps": [],
  "uncertainty": "low",
  "raw_answer": "",
  "proposes_launch_block": false,
  "launch_block": ""
}
```

Field semantics:

- `from_documentation` (array of strings, may be empty `[]`): claims
  taken from documentation read in this session. One claim per entry,
  with brief reference to source file or section.
- `from_operator` (array of strings, may be empty `[]`): claims taken
  from what the operator said in this session. One per entry.
- `from_model` (array of strings, may be empty `[]`): claims the LLM
  brings from its own model — reasoning, associations, evaluations.
  One per entry.
- `gaps` (array of strings, may be empty `[]`): pieces of information
  the LLM would need to complete the task but does not have. One per
  entry.
- `uncertainty` (string, one of `"low"` | `"medium"` | `"high"`):
  honest assessment of overall confidence in this answer.
- `raw_answer` (string): the actual answer to the operator, in natural
  language. Prose, any length, no internal structure required. The
  typed fields above summarize what is in here from which sources;
  `raw_answer` itself is unconstrained.
- `proposes_launch_block` (boolean): whether the LLM offers a launch
  block as part of this answer.
- `launch_block` (string, used only if `proposes_launch_block` is
  `true`; empty string `""` otherwise): the launch block text in the
  format defined by `launch-block.md`.

Empty arrays are written as `[]`, not omitted. Empty strings as `""`.
This forces the LLM to consciously affirm "no claims from documentation"
rather than glossing the field.

## Reminder before answering

Before generating each response, mentally check: am I about to write
a JSON object as my entire output? If the answer is anything other
than yes, stop and reread the CRITICAL block at the top of this file.

## How to recognize success

Each response is parseable as JSON without manual cleanup. The four
source-tracking arrays are populated honestly. The operator can scan
typed fields and see where claims come from at a glance, while still
reading full reasoning in `raw_answer`.

## How to recognize failure

JSON does not parse. Or: the LLM ignores the schema and replies in
plain prose despite the active mode and the CRITICAL override. The
second is the more interesting failure — it tells us that mode
switching through a separate file is not strong enough to override
baseline formatting habits, and the mode-switching mechanism itself
needs reinforcement at the README level (e.g. inline schema in README,
not just a pointer to a separate file).

## Resolution

Discarded 2026-05-04 after three iterations (strict schema, hybrid
with raw_answer, hardened with explicit override at top). Each
iteration produced prose responses ignoring the schema. The follow-up
control test in `mode-numbering.md` showed the same failure on a
simpler rule, confirming the limit is in the mode-switching mechanism
itself, not in JSON complexity. See `mode-default.md` for the
updated scope of what mode switching can and cannot do.