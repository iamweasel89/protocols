# Mode: default

Standard configuration. Active when no experiment is in progress.

## What this mode does

Nothing special. The full set of protocols listed in `README.md`
applies as written: agent-discipline governs behavior, project-memory
governs memory format, launch-block governs delivery, etc. No
overrides, no narrowing, no extra rules.

## When to switch out of this mode

Switch to a named experimental mode when introducing a behavior or
format change that needs to be tested in isolation before becoming
part of the default. Examples of work that warrants a mode:

- A new response format (numbered messages, JSON-shaped answers,
  token-counted replies).
- A new behavioral protocol that overrides parts of agent-discipline
  to test alternatives.
- A focused work session on a specific subset of protocols, where
  reading the full set would dilute the experiment.

## When to NOT switch modes

Do not switch modes for routine project work. The default is the
default for a reason — it is the integrated, tested, current
configuration. Modes are for experiments, not for daily work.

## Activation requirements

Mode activation requires two conditions in the operator's opening
of a session:

1. **Cache-bust on the entry URL.** Append a query string such as
   `?nocache=<timestamp>` to the protocols-repo URL. Without this,
   fresh sessions may receive a CDN-cached README from before the
   current Active mode pointer existed, and the mode contract is
   never seen by the LLM. See `fetching.md`, "Operator-side rule".

2. **Two-phase activation.** Send the entry URL alone in the
   first message, with no task attached. Let the LLM read the
   mode file, acknowledge it, and only then post the task in a
   second message. Bundling URL + task in one reply does not
   reliably activate the mode — the LLM tends to answer the task
   in default style without operationalizing the mode rules.

Both conditions were tested on 2026-05-04 with `mode-numbering`
(simple `[N]` prefix rule). Without them, three iterations of a
`mode-json` experiment and one of `mode-numbering` failed. With
them, `mode-numbering` succeeded immediately and persisted across
multiple turns.

Modes can change scope, focus, AND response format — earlier
belief that format changes were architecturally impossible was a
misdiagnosis caused by CDN-cached READMEs. See
`mode-numbering.md` Resolution for the retroactive correction.

## How experimental modes resolve

An experimental mode ends in one of two ways:

1. **Merge into default.** The experiment succeeded. The mode's rules
   are folded into the relevant base protocol via launch block. The
   mode file is then deleted. This is the success path.
2. **Discard.** The experiment showed the change does not work. The
   mode file is deleted, the Active mode pointer returns to default,
   nothing is folded in. This is also a valid outcome — knowing what
   does not work is a result.