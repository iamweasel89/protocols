# Mode activation

Specification of how a mode pointer in `README.md` actually takes
effect in a fresh LLM session, with empirical findings on which
models honor it and to what degree.

## Cross-model findings (2026-05-04)

A single mode design (`mode-json` requiring JSON-shaped output) was
tested across four models with the same activation contract. Results
differed substantially.

### Claude

Activates on a bare entry-point URL with cache-bust query string,
first message containing only the URL. No imperative in the operator's
reply needed. Source-tracking fields (`from_documentation`,
`from_operator`, `from_model`, `gaps`) populated honestly, with the
operator's actual statements only — no confabulation.

### DeepSeek

Does not activate on a bare URL, even with cache-bust. Activates when
the operator includes an explicit imperative in the first message
(e.g. "соблюдай активный режим" + URL). Schema is followed; output is
parseable JSON. However, `from_operator` is contaminated: DeepSeek
inserts statements the operator never made, treating its own
inferences about the operator's intent as if quoted. Treat DeepSeek's
source-tracking output with skepticism even when the mode activates.

### ChatGPT

Does not activate on a bare URL. Does not activate on URL plus
imperative either. Reads the file, describes its design, critiques
it as a "half-protocol", and offers an alternative architecture
(MCP, structured prompts, validation layers). Repositions itself as
an advisor on protocol design rather than as a participant in the
protocol. The mode mechanism through file-pointer + cache-bust +
imperative is not sufficient.

### Gemini

Does not activate. Replies "Active mode enabled" in prose without
producing JSON, simulating compliance verbally while ignoring the
format requirement. When asked directly why it did not apply the
protocol, names three conditions it would need: strict output
contract, system role directive, few-shot examples. These are
platform-level mechanisms (system prompts via API), not file-level.

## Conclusion

Mode activation through file pointers is not model-independent.
Achieving uniform behavior across all four tested models via the
mode-file mechanism alone is not feasible. The mechanism is a Claude-
native pattern that other models honor partially or not at all.

Practical guidance:

- For Claude: the existing mode mechanism (cache-bust + two-phase) is
  reliable. Use it.
- For DeepSeek: add an explicit imperative in the operator's first
  message ("соблюдай активный режим"). Mode activates, but verify
  source-tracking fields manually — they may include confabulation.
- For ChatGPT and Gemini: do not rely on file-based mode activation.
  Use platform-level system prompts or in-message instructions per
  reply.

## Two conditions (for models that honor file-based modes)

### 1. Cache-bust on the entry URL

The operator opens the fresh session with a URL of the form:

```
https://github.com/<owner>/<repo>?nocache=<unique-string>
```

The query string is ignored by GitHub's rendering — only Fastly's
cache layer cares about it. Without this, fresh sessions may receive
a CDN-cached README from before the current Active mode pointer
existed, and the mode contract is never seen by the LLM.

See `fetching.md`, "Operator-side rule".

### 2. Two-phase activation

The first message contains only the entry-point URL (with cache-bust).
The LLM reads, acknowledges the mode, and waits. The second message
poses the actual task.

For DeepSeek specifically, also include a short imperative in the
first message such as "соблюдай активный режим".

## What individual mode files must include

For the contract to work where it works at all, each `mode-<name>.md`
must specify:

1. **The format rule itself** — what the LLM must do differently.
2. **A "Required first response" section** — exact form of the
   acknowledgement when first message is just the entry URL.
3. **Recognition criteria** for success and failure.
4. **A Resolution section** — empty initially, filled when the mode
   ends.

## History

Earlier on 2026-05-04, three iterations of `mode-json` and one of
`mode-numbering` failed under naive activation (no cache-bust, single-
message bundle of URL + task). Retroactively diagnosed as CDN-mediated.
Subsequent retests with cache-bust + two-phase succeeded on Claude.
Cross-model testing on DeepSeek, ChatGPT, and Gemini then narrowed
the conclusion: mechanism is Claude-reliable, DeepSeek-conditional
(with caveats), ChatGPT-rejected, Gemini-simulated.

Two experimental variants in this investigation —
`mode-json-operator-voice.md` (first-person operator voice in the
file) and `prompt-json.md` (file phrased as a direct prompt) — both
failed to lower DeepSeek's activation threshold. They are deleted;
their Resolutions live in git history.