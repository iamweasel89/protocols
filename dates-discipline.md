# Dates discipline

Honesty rules for any date written into a file by an LLM or an operator.
Apply wherever a date appears in a repo: README, memory leaves, code
comments, session logs, decision records.

Used by: birdscope.

## Why

LLMs hallucinate dates with confidence. Operators forget to update them.
Once a wrong or stale date is in a file, every later reader treats it
as fact. The only defense is discipline at the moment of writing.

The cost of getting this wrong is asymmetric: a wrong date can mislead
indefinitely, while admitting "unknown" is briefly awkward and then
self-correcting.

## Three rules

### R1. Never invent a date

If the real date is not known at the moment of writing, write
`unknown` or `pending`. Do not write a "probably right" date.
Do not approximate. Do not pattern-match from training data.

This applies to year, month, and day independently. "2026" alone is
better than "2026-05-15" if the day is a guess.

### R2. The source must be checkable

Before writing a date, fetch it from a verifiable source:

- `date` in bash, or `Get-Date` in PowerShell — for the current date
  on the operator's machine.
- `git log -1 --format=%cs <file>` — for the date a file was last
  committed.
- The user's explicit message ("today is X") — when in conversation.

Do not write a date based on memory of when something happened.
Memory is not a source.

### R3. Stale dates need a plan

A date written into a file is a snapshot. If staleness matters
(e.g. `last_verified`, `surveyed_on`), pair the date with one of:

- An explicit `review_after: YYYY-MM-DD` field, so the staleness has
  a deadline.
- A note in the surrounding text saying when to re-check.
- An accepted assumption that the date is final and will not be
  updated (e.g. for historical events).

Otherwise dates accumulate and quietly rot. Files end up with
`last_verified: 2024-something` that nobody believes anymore.

## Where dates appear

In a typical project, dates show up in:

- **Memory leaves** — `last_verified`, `review_after`, free-text
  references like "surveyed 2026-05-02".
- **README** — sometimes a "last updated" line, or links to changelogs.
- **Code comments** — TODO blocks with deadlines, deprecation notices.
- **Session logs** — when sessions are recorded.
- **Decision records / ADRs** — date of the decision.

Each of these is a candidate for the rules above. Git commit dates
are exempt — they are automatic and trustworthy.

## Common gotchas

- **LLM with no time tool.** If the LLM has no access to the current
  date, it must say so when asked, and refuse to write a current date
  into a file. Use `unknown` or ask the operator.
- **Copy-pasted dates.** When ingesting external content, dates inside
  it belong to the source, not to the project. Do not treat them as
  the project's own timestamps.
- **Approximate dates from memory.** "Last summer" is not a date.
  Either pin it down with a source, or write the qualitative phrase
  ("last summer") rather than fake precision.
- **Time zones in single-day precision.** Most project files use day
  precision and ignore time zones. This is fine. When sub-day precision
  matters, write the time zone explicitly (`2026-05-02 14:30 CEST`).
