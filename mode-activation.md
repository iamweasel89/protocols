# Mode activation

Specification of the contract that must hold for a mode pointer in
`README.md` to actually take effect in a fresh LLM session. Referenced
by `mode-default.md` and by any individual `mode-<name>.md` file.

Empirically established 2026-05-04 across two independent modes
(`mode-numbering`, `mode-json`) covering both a simple format rule
(prefix) and a complex one (full JSON schema with typed fields). Both
modes failed under the naive activation pattern and succeeded under
the contract below — establishing the contract, not the mode files,
as the load-bearing element.

Note on activation thresholds across models. Empirically (2026-05-04),
Claude activates a mode under either of the two conditions alone in
this document; DeepSeek requires both conditions plus a content task
in the first message — bare URL alone, even with cache-bust, did not
trigger activation in DeepSeek (instant and expert modes). The mode
mechanism itself works on both models; what differs is the threshold
at which the mode file is treated as a binding contract rather than
descriptive text. To minimize cross-model divergence, design mode
files so that any reference to them in the first message activates the
mode, not only the bare-URL form.

## Two conditions

### 1. Cache-bust on the entry URL

The operator opens the fresh session with a URL of the form:

```
https://github.com/<owner>/<repo>?nocache=<unique-string>
```

The query string is ignored by GitHub's rendering — only Fastly's
cache layer cares about it. The string can be any value unique enough
to miss the cache key; a date stamp such as `20260504` is sufficient.
A version increment (`20260504a`, `20260504b`) keeps successive tests
on the same day from sharing a key.

Without this, the fresh session may receive a Fastly-cached README
from hours or a day earlier, missing the current Active mode pointer
entirely. The mode contract is then never seen by the LLM and the
experiment fails silently — looking like an architectural failure
when it is actually a stale-cache failure.

See `fetching.md`, "Operator-side rule", for the broader context on
CDN-mediated staleness.

### 2. Two-phase activation

The first message in the fresh session contains **only** the entry-
point URL (with cache-bust, per condition 1). No task, no preamble,
no additional instructions. The LLM's first response acknowledges
the active mode — typically by following the "Required first response"
procedure that each mode file specifies.

Only in the **second** message does the operator pose the actual task.

Without this separation, the LLM tends to answer the task in default
style without operationalizing the mode rules — the task collapses
the activation phase. With separation, the mode is acknowledged in
the first response, which forces the LLM to read and process the mode
file before any task-related reasoning competes for attention.

## What individual mode files must include

For the contract to work, each `mode-<name>.md` file must specify:

1. **The format rule itself** — what the LLM must do differently from
   default behavior.
2. **A "Required first response" section** — the exact form (or close
   to exact) of what the LLM should reply when the operator's first
   message is just the entry URL. This is what makes the two-phase
   activation testable.
3. **Recognition criteria for success and failure** — observable
   outcomes, not subjective ones.
4. **A Resolution section** — initially empty, filled in when the mode
   ends, recording whether the experiment succeeded, failed, or was
   discarded for other reasons.

## What the contract does not promise

- That the LLM will sustain the mode rule across very long sessions.
- That the LLM will resist explicit operator overrides.
- That every model will operationalize a read mode file. See caveat
  at top of this document.

## History

Earlier on 2026-05-04, three iterations of `mode-json` and one of
`mode-numbering` were run without the contract above and all failed.
The Resolution at that time concluded that file-pointer-based mode
switching could not change response format. That conclusion was wrong:
subsequent retests with cache-bust + two-phase activation succeeded
on both modes (Claude). DeepSeek subsequently failed to activate the
JSON mode under the same contract — added to the caveat section.