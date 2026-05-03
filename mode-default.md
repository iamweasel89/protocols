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

## Known limit of mode switching

Modes can change **scope** and **focus** of a session: which protocols
are read, which content is in play, what kind of work is centered.
Modes can NOT reliably change the LLM's **response format**. This was
tested directly on 2026-05-04 with `mode-json` (JSON schema) and
`mode-numbering` (sequential prefix `[N]`). Both failed: the LLM read
the mode file and proceeded to answer in default prose anyway. A
simple, known-easy rule failed the same way as a complex one, so the
limit is structural, not about wording or directiveness.

For format changes, use one of:
- A direct instruction in each operator message ("answer with `[N]`
  prefix"), which works.
- Multi-agent orchestration where one agent reformats another's
  output (out of scope for the chat substrate).
- A custom chat substrate that wraps the LLM with a format layer
  (out of scope here; see operator's separate explorations).

Modes via the `Active mode` README pointer remain useful for
narrowing what the session reads and works on — and for keeping
experiments isolated until they merge or discard.

## How experimental modes resolve

An experimental mode ends in one of two ways:

1. **Merge into default.** The experiment succeeded. The mode's rules
   are folded into the relevant base protocol via launch block. The
   mode file is then deleted. This is the success path.
2. **Discard.** The experiment showed the change does not work. The
   mode file is deleted, the Active mode pointer returns to default,
   nothing is folded in. This is also a valid outcome — knowing what
   does not work is a result.