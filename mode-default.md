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

## How experimental modes resolve

An experimental mode ends in one of two ways:

1. **Merge into default.** The experiment succeeded. The mode's rules
   are folded into the relevant base protocol via launch block. The
   mode file is then deleted. This is the success path.
2. **Discard.** The experiment showed the change does not work. The
   mode file is deleted, the Active mode pointer returns to default,
   nothing is folded in. This is also a valid outcome — knowing what
   does not work is a result.