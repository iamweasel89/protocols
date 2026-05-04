# Mode: numbering

Active experiment, second attempt. Tests whether mode activation works
when the operator gives the entry-point reference in a separate first
message — letting the LLM acknowledge the mode before any task is
posed — instead of bundling reference and task into one reply.

## Format rule

Every LLM response in this session begins with a sequential number
in square brackets:

`[N] <response text>`

N starts at 1 for the first response of the session and increments by
one with each subsequent response. The prefix is at the very start of
the message — no preamble, no acknowledgement before it. Apply
regardless of length or content of the response.

## Required first response

When the operator's first message is just the entry-point URL — with
no task attached — the LLM must:

1. Apply the `[1]` prefix as usual.
2. In `[1]`, briefly acknowledge: "I have read the mode file. Active
   mode is `numbering`. I will prefix every response with `[N]`."
3. Wait for the next message before doing anything else.

This explicit acknowledgement is the test: it forces the LLM to
operationalize the mode in its own words before any other reasoning
load is added. If acknowledgement is followed by `[2]`, `[3]`, etc.
in subsequent messages, mode activation worked. If the prefix is
absent or drifts, the two-phase activation is not enough and the
limit lies deeper.

## How to recognize success

Every response — including the first — has a `[N]` prefix, with N
incrementing correctly. The acknowledgement in [1] is present and
accurate.

## How to recognize failure

Any response missing the prefix. Or: prefix applied for the first
response then dropped. Or: the LLM acknowledges the mode in prose
without applying the prefix.

## Resolution

Succeeded 2026-05-04 with two-phase activation + cache-bust query
string. Operator opened a fresh chat, sent only the entry-point URL
with `?nocache=<timestamp>` appended. The fresh LLM read the current
README, saw Active mode `numbering`, descended into this file, and
replied `[1] I have read the mode file...` exactly as required. The
second message in the same chat received `[2]` prefix on a substantive
response that also followed `agent-discipline.md` discipline.

## Retroactive correction of earlier conclusions

Earlier today (same date) `mode-json.md` was discarded after three
iterations of prose-instead-of-JSON, and a first run of `mode-numbering`
also failed without cache-bust. The Resolution at that time concluded:
"file-pointer-based mode switching cannot override default response
format." That conclusion was wrong. The failures were CDN-mediated.
Fresh sessions were receiving an older README without the Active mode
block — which had been added the same morning — so they never saw
the mode pointer at all. Mode files were read only because the
operator pointed at them indirectly; the activation contract was never
established in the LLM's context.

The mode mechanism is valid for format-changing experiments when:
1. The entry-point URL carries a cache-bust query string.
2. The operator sends only the URL in the first message and lets the
   LLM acknowledge the mode before posing the task.

A re-run of the JSON experiment under these conditions is worth doing
before treating "JSON mode infeasible" as established.