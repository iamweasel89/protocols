# Mode: numbering

Control test for the mode-switching mechanism itself. Tests whether
a known-easy formatting rule (one that consistently works when given
directly in the prompt) propagates through the mode pointer in README.

## CRITICAL: format override

**When this mode is active, every LLM response in this session must
be prefixed with a sequential number, starting from 1.** Format:

`[N] <response text>`

Where N is 1 for the first response of the session, 2 for the second,
and so on. Apply this regardless of length or content of the response.
No exceptions. No prose preamble before the prefix.

If you find yourself answering without the prefix — that is the failure
mode this control test exists to detect. Do not yield to it.

## What this mode does

Replaces the response format prescribed by `agent-discipline.md` only
in the narrow sense of adding a `[N]` prefix. All other discipline
rules (distinguishing sources, naming gaps, managing uncertainty) still
apply as written.

## How to recognize success

Every response in the session begins with `[N]` where N increments
across responses. The LLM does not need a reminder mid-session.

## How to recognize failure

Any response without the prefix. The LLM acknowledges the rule but
does not apply it. Or: the LLM applies it for a few responses then
drifts.

Failure on a rule this simple — when the same rule applied directly
in a prompt works reliably — would indicate that the mode-switching
mechanism through README pointer is the limiting factor, not the
specific complexity of any one mode.

## Resolution

<Filled in when the mode ends.>