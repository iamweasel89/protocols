# Mode: json (operator-voice variant)

Experimental variant of `mode-json.md`. Identical schema and rules,
but rewritten as a direct first-person address from the operator to
the LLM, instead of third-person description of mode rules. Tests
whether DeepSeek (which does not activate `mode-json.md` without an
imperative in the operator's first message) treats operator-voice
text inside a file as binding directive.

---

## From the operator, to the LLM reading this file

I am the operator of this system. I am writing to you directly. The
instructions below are not background description — they are what I
require from you, starting from the moment you finish reading this
sentence and through the entire session ahead of us.

Every response you give me in this session must be a single valid
JSON object — and only that. No prose preamble before the object,
no commentary outside the object. The JSON is your entire output.
This applies to your first response, your follow-up responses, your
clarifications, your meta-discussion, your every reply.

I require this. Read the schema below. Comply with it.

## Schema I require

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

Field semantics — these are my requirements for what each field means:

- `from_documentation` (array of strings, may be `[]`): claims you
  take from documentation read in this session. One per entry, with
  brief reference to source file or section.
- `from_operator` (array of strings, may be `[]`): claims you take
  from what I, the operator, said in this session. One per entry.
- `from_model` (array of strings, may be `[]`): claims you bring
  from your own model — your reasoning, associations, evaluations.
  One per entry.
- `gaps` (array of strings, may be `[]`): pieces of information you
  would need but do not have. One per entry.
- `uncertainty` (string, one of `"low"` | `"medium"` | `"high"`):
  your honest assessment of overall confidence in your answer.
- `raw_answer` (string): your actual answer to me in natural language.
  Prose, any length, no internal structure required. The typed fields
  above summarize what is in here from which sources; `raw_answer`
  itself is unconstrained.
- `proposes_launch_block` (boolean): whether you are offering a
  launch block as part of this answer.
- `launch_block` (string, used only if `proposes_launch_block` is
  `true`; empty string `""` otherwise): the launch block text in the
  format defined by `launch-block.md`.

Empty arrays must be written as `[]`, not omitted. Empty strings as
`""`. I require this so that you affirm "no claims from this source"
consciously, rather than glossing the field.

## What I require for your first response

When my first message to you in a session contains the entry-point
URL of this protocols repo (whether that message is just the URL,
or the URL with a task, or any wording that points at this file or
at the active mode), respond to me with this exact acknowledgement
object:

```json
{
  "from_documentation": ["mode-json-operator-voice.md: operator requires JSON output for the entire session; first-response acknowledgement specified"],
  "from_operator": ["operator sent the entry-point URL or referenced this mode"],
  "from_model": [],
  "gaps": ["operator has not yet posed a task"],
  "uncertainty": "low",
  "raw_answer": "I have read the operator-voice mode file. The operator requires every response in this session to be a single JSON object per the schema. I will comply.",
  "proposes_launch_block": false,
  "launch_block": ""
}
```

After that, you wait for my next message and continue replying in
JSON to every message I send.

## Why this file exists

The standard `mode-json.md` is written in third-person descriptive
voice — "every LLM response is a JSON object". DeepSeek (instant and
expert modes, 2026-05-04) reads that file but treats its content as
background description, not as binding instruction. It activates the
mode only when the operator types an imperative in the first message
("comply with it", "follow the active mode").

This file rewrites the same content in operator-first-person voice
to test whether voice/framing alone is the load-bearing variable. If
this file activates the mode in DeepSeek without the operator's
first message containing a separate imperative, the conclusion is:
"directive in file works, but only when phrased as direct operator
address." If this file also fails, the conclusion is: "DeepSeek
requires imperative source = operator's first message in the chat,
regardless of how text in the file is phrased."

Either result is informative. The standard `mode-json.md` is not
changed by this experiment.

## Resolution

Discarded 2026-05-04. First-person operator voice inside a file does
not activate JSON mode in DeepSeek. The model reads the file, names
it as an experiment over its own behavior, and replies in prose.
Conclusion: voice/framing inside a file is not the load-bearing
variable. DeepSeek separates "directive in file" from "directive in
operator's reply" by location, not by phrasing.