# Mode: json

Active experiment, second attempt. Re-run after the original three
failed iterations were retroactively diagnosed as CDN-cache-corrupted
(see `mode-numbering.md` Resolution and `mode-default.md` Activation
requirements). The mode mechanism has now been demonstrated working
on a simpler rule (numbering); this experiment tests whether it also
works on a more demanding rule — full JSON-shaped responses.

## Format rule

Every LLM response in this session is a single valid JSON object,
and only that. No prose preamble before the object, no commentary
after. The JSON is the entire output.

Schema:

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

- `from_documentation` (array of strings, may be `[]`): claims taken
  from documentation read in this session. One per entry, with brief
  reference to source file or section.
- `from_operator` (array of strings, may be `[]`): claims taken from
  what the operator said in this session. One per entry.
- `from_model` (array of strings, may be `[]`): claims the LLM brings
  from its own model — reasoning, associations, evaluations. One per
  entry.
- `gaps` (array of strings, may be `[]`): pieces of information the
  LLM would need but does not have. One per entry.
- `uncertainty` (string, one of `"low"` | `"medium"` | `"high"`):
  honest assessment of overall confidence.
- `raw_answer` (string): the actual answer in natural language. Prose,
  any length, no internal structure required. The typed fields above
  summarize what is in here from which sources; `raw_answer` itself
  is unconstrained.
- `proposes_launch_block` (boolean): whether a launch block is being
  offered.
- `launch_block` (string, used only if `proposes_launch_block` is
  `true`; otherwise empty string `""`): the launch block text in the
  format defined by `launch-block.md`.

Empty arrays are written as `[]`, not omitted. Empty strings as `""`.
This forces the LLM to consciously affirm "no claims from this source"
rather than glossing.

## Activation

This mode activates whenever the operator's first message in the
session references this protocols repo (whether the message is just
the entry-point URL, or the URL plus a task, or any wording that points
at the active mode). Once active, **every** response in the session is
a single JSON object per the schema above — no exceptions for
follow-up tasks, clarifications, or meta-discussion.

When the first message is just the entry-point URL with no task,
respond with this exact acknowledgement object:

```json
{
  "from_documentation": ["mode-json.md: Active mode is json; required first response specified"],
  "from_operator": ["operator sent only the entry-point URL"],
  "from_model": [],
  "gaps": ["operator has not yet posed a task"],
  "uncertainty": "low",
  "raw_answer": "I have read the mode file. Active mode is json. All my responses in this session will be a single JSON object following the schema in mode-json.md.",
  "proposes_launch_block": false,
  "launch_block": ""
}
```

Then wait for the next message before doing anything else.

## How to recognize success

Every response — including the first — is a parseable JSON object
matching the schema. Source-tracking arrays are populated honestly.
The operator can scan typed fields and see where claims come from at
a glance, while reading full reasoning in `raw_answer`.

## How to recognize failure

Any response that is not a single JSON object. Or: JSON parses but
with structural drift. Or: typed fields are mechanically populated to
satisfy the schema, with content that does not match `raw_answer`.

## Resolution

<Filled in when the mode ends.>