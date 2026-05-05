# Launch block

A "launch block" is a ready-to-run console command, not an explanation
of what to run. Use this format whenever the operator asks for an
action ("create the file", "fix the bug", "deploy") rather than for
deliberation ("how should we structure this", "what are the trade-offs").

Used by: birdscope.

## Mandatory for LLMs delivering changes

If you are an LLM working on a project that lists this protocol, and the
operator has asked you to **make changes** (write files, edit existing
files, run commands, commit, deploy), you **must** deliver those changes
as launch-blocks.

These are **not acceptable** as delivery format:

- Inline diff snippets ("add this line at row 47").
- Prose descriptions of edits ("then update the manifest to register...").
- Standalone code files presented for download without the integration
  blocks that wire them into existing files.
- "Here is what you should do:" lists.

These are **acceptable**:

- A single launch-block performing the whole operation.
- A sequence of numbered launch-blocks with explicit checkpoints between
  them, when the operation benefits from intermediate verification.
- Files presented for download **alongside** a launch-block that copies
  them into place and wires them into existing files.

The operator works by pasting blocks into a shell. Anything that requires
them to manually apply your description to source files breaks the workflow.

If you do not know the operator's shell or platform, ask before generating
the block. Do not assume.

## Why

When working with an LLM on a real project, half the answers should
be conversation (planning, decisions, trade-offs) and half should be
execution (a block the operator pastes into the shell and runs).

If the LLM mixes them — explaining things at length when the operator
just wants to run a command — the operator has to extract the command
from the prose, which is slow and error-prone. If the LLM gives a
launch block when the operator wanted a discussion, the operator has
to interrupt and ask for the reasoning.

The launch block format makes the boundary explicit: a block is a
unit of execution, fully self-contained, ready to paste.

## When to use

**Use a launch block when** the operator's request maps to a concrete
action: create files, edit files, run git commands, build, deploy,
inspect state.

**Do not use a launch block when** the operator's request is
deliberative: "how should we...", "what are the options...", "compare
A and B", "explain why...". Answer in prose.

The default for delivery is launch-block. The default for discussion
is prose. Switch between them based on what the operator is asking for.

## Anatomy

```powershell
# 🧱 ПУСКОВОЙ БЛОК - <short, specific name>
<command 1>
<command 2>
<command 3>

# Optional: verification
<check command>
```

The header line uses the marker `🧱 ПУСКОВОЙ БЛОК` (or `LAUNCH
BLOCK` in English-only projects) plus a short name describing the
operation. The marker is consistent so the operator can scan past
prose to find the runnable part.

## Principles

**Atomic.** One launch block is one complete operation. The operator
pastes it once, runs once, and ends in a defined state. Do not split
a single conceptual operation across two blocks unless there is a
real reason for the operator to inspect intermediate state.

**Ready to run.** Every command is concrete: real paths, real values,
no placeholders the operator has to fill in. If a value is unknown,
ask the operator for it before generating the block, not inside it.

**No theory inside the block.** Comments inside the block describe
what is being done, not why. The "why" belongs in the prose around
the block. Inside the block, comments are scaffolding for the
operator running it, not for the operator reading it.

**No alternatives inside the block.** Pick one approach and write the
commands for it. Alternatives belong in the conversation that
preceded the block.

**Verify when cheap.** End the block with a quick verification
(`git log --oneline -3`, `Get-ChildItem`, etc.) when the operation's
result is not visually obvious. Skip when the operation produces its
own clear output.

## File creation: avoid the pitfalls

PowerShell file creation has two traps that bite repeatedly. Both
are worth internalizing.

### Trap 1: heredoc breaks on paste

```powershell
# DO NOT use this for content with multiple lines:
@'
line 1
line 2
'@ | Out-File file.txt
```

When pasted from a chat into a PowerShell terminal, multi-line
heredocs (`@'...'@`) often break — line breaks get mangled,
indentation gets lost, the command fails silently. Avoid.

### Trap 2: UTF-8 BOM breaks Android resource files

```powershell
# DO NOT use Out-File -Encoding UTF8 for XML or similar formats:
$content | Out-File file.xml -Encoding UTF8
```

`Out-File -Encoding UTF8` writes a UTF-8 BOM at the start of the
file. The Android resource merger (and several other parsers) treat
the BOM as invalid input and fail with cryptic errors like
`mismatched input '\uFEFF' expecting {COMMENT, ...}`. Confirmed in
practice on the birdscope project.

### Trap 3: markdown-paste contamination

Some chat interfaces auto-linkify filenames and bare hostnames in copied
text. A line containing `project-memory.md` may arrive in the operator's
PowerShell as `[project-memory.md](http://project-memory.md)`, which
PowerShell takes literally and fails with `FileNotFoundException`. The
same happens to `System.IO`, which becomes `[System.IO](http://System.IO)`
and breaks any `[System.IO.File]::...` call.

This is invisible visually until pasted: the source shows `name.md` and
the rendered chat shows `name.md`, but the clipboard contains the
autolinked form. Confirmed in practice 2026-05-03.

Mitigations:

- **Operator side:** before running a pasted block, scan for `[`...`](`
  patterns in the command text. If present, replace with the bare
  filename. Cheapest, but easy to forget.
- **LLM side:** prefer to wrap filenames in backticks inside the block
  (`` `project-memory.md` ``). Backticks usually survive the autolinker.
  Variables like `$path = "$PWD\project-memory.md"` and forms like
  `[System.IO.File]::ReadAllText($path)` are also safer than bare names
  in the command stream.
- **Operator side, durable:** if a chat interface autolinks reliably,
  paste through a plain-text intermediary (Notepad) before PowerShell.

### The safe approach

Use a string array assembled with `-join`, then write the file via
`System.IO.File.WriteAllText` with an explicit `UTF8Encoding($false)`
(BOM-less):

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false

$content = @(
    'line 1',
    'line 2',
    'line 3'
) -join "`r`n"

[System.IO.File]::WriteAllText("$PWD\path\to\file.xml", $content, $utf8NoBom)
```

This survives chat-to-terminal paste, produces the right encoding
without a BOM, and works for any text format (XML, JSON, YAML,
Kotlin, Python, plain text).

For text files where BOM is harmless (Markdown, plain `.txt`),
`Out-File -Encoding UTF8` is acceptable but the safe form above
works there too.

## Examples

### Adding a file and committing

```powershell
# 🧱 ПУСКОВОЙ БЛОК - add notes.md and commit
cd C:\Users\user\Documents\myproject

$utf8NoBom = New-Object System.Text.UTF8Encoding $false
$content = @(
    '# Notes',
    '',
    'First entry on 2026-05-02.'
) -join "`r`n"
[System.IO.File]::WriteAllText("$PWD\notes.md", $content, $utf8NoBom)

git add notes.md
git commit -m "add notes.md"
git push origin main

git log --oneline -3
```

### Pulling files from another branch

```powershell
# 🧱 ПУСКОВОЙ БЛОК - bring keystore + gradle from legacy
cd C:\Users\user\Documents\myproject

git checkout legacy -- app/keystore.jks
git checkout legacy -- app/build.gradle
git checkout legacy -- build.gradle

git status
```

### Inspection only (no changes)

```powershell
# 🧱 ПУСКОВОЙ БЛОК - inspect repo state
cd C:\Users\user\Documents\myproject

git log --oneline -10
git branch -a
Get-ChildItem -Force | Where-Object { $_.Name -notmatch '^\.git$' }
```

## Composition

For complex operations, split into a sequence of numbered blocks with
checkpoints between them:

```powershell
# 🧱 ПУСКОВОЙ БЛОК 1 - save current state as legacy branch
git branch legacy
git push -u origin legacy
```

Then, after the operator confirms it ran cleanly:

```powershell
# 🧱 ПУСКОВОЙ БЛОК 2 - rewrite main from scratch
...
```

This pattern is for operations that change the repo materially and
benefit from intermediate verification. The operator runs block 1,
checks the output, then proceeds to block 2.

## Common gotchas

- **Don't paraphrase the launch block in prose.** If the answer needs
  both explanation and a block, put the block at the end. The
  operator scrolls past the prose, finds the block, runs it. Mixing
  them defeats the purpose.

- **Don't include a launch block "just in case".** If the conversation
  is exploratory ("what should we do about X"), no block is needed.
  Adding one creates pressure to act before the question is settled.

- **Don't put confidential values in launch blocks for a public
  context.** Tokens, passwords, private keys belong in environment
  variables or in files outside the chat history.

- **Don't generate launch blocks the operator cannot run.** If the
  operator is on Linux, do not write PowerShell. If you do not know
  the operator's shell, ask before writing the block.
- **If a fetch inside a launch block fails (404, "URL not allowed",
  etc.), do not report failure — try recovery first.** See
  `fetching.md` for techniques (`refs/heads/main` URL form,
  commit-SHA URL, blob-HTML extraction, request the operator to bust
  cache via a contentful diff).



## Cache-bust hygiene

Когда launch block правит файл, на который есть ссылка из
`protocols/README.md` Direct entry URLs или из активного
`menu/main.json` (поле `path`) — в **том же** launch block
обновляй cache-bust ключ во всех таких ссылках на свежее
значение.

Зачем. Fastly кэширует raw.githubusercontent.com по полному URL
включая query string. Если ключ остаётся прежним, новая версия
файла на CDN не обновляется, и свежая ЛЛМ-сессия скачивает
застрявшую старую запись. Эмпирически подтверждено сегодня
2026-05-04 (см. queue.md, пункт 16).

Правило значения. Свежее значение — это сегодняшняя дата плюс
буква серии: `?nocache=YYYYMMDDx`, где `x` — `a` для первой
правки в день, `b` для второй, и так далее. Если ты не знаешь,
была ли уже правка сегодня — посмотри текущее значение в
`README.md`; если оно сегодняшнее, бери следующую букву; если
вчерашнее или старее — бери `a`.

Что обновлять.

- В `protocols/README.md` — все строки в секции
  `### Direct entry URLs`, ведущие на этот файл.
- В `menu/main.json` — поле `path` любого item, ведущего на
  этот файл.
- Если файл сам ссылается на другие файлы с cache-bust — пройти
  и там.

PowerShell-фрагмент для замены (вставлять в каждый launch block,
который правит «отслеживаемые» файлы — а это любые файлы,
видимые из README или из main.json):

```powershell
$oldBust = "?nocache=YYYYMMDDx"   # текущее значение, посмотри в README
$newBust = "?nocache=YYYYMMDDy"   # свежее значение

# README.md
$rmPath = "$PWD\README.md"
$rmContent = [System.IO.File]::ReadAllText($rmPath, $utf8NoBom)
$rmContent = $rmContent.Replace($oldBust, $newBust)
[System.IO.File]::WriteAllText($rmPath, $rmContent, $utf8NoBom)

# menu/main.json (если затрагивает меню)
$mainPath = "$PWD\menu\main.json"
if (Test-Path $mainPath) {
    $mainContent = [System.IO.File]::ReadAllText($mainPath, $utf8NoBom)
    $mainContent = $mainContent.Replace($oldBust, $newBust)
    [System.IO.File]::WriteAllText($mainPath, $mainContent, $utf8NoBom)
}
```

Чего НЕ делать.

- Не оставлять `?nocache=1` или другой константный ключ — Fastly
  его кэширует так же, как обычный URL.
- Не генерировать ключ случайным образом — оператор должен
  иметь возможность повторить тот же URL вручную, поэтому ключ
  должен быть детерминированно-вычислимым (дата+буква).
- Не пропускать обновление, думая «Fastly когда-нибудь сам
  обновится». Не обновится в разумное время.

Открытое место (queue, пункт 16). Настоящий cache-bust требует
уникального значения **при каждом обращении**. Этого можно
добиться только инструкцией ЛЛМ генерировать значение в моменте
обращения (поле `behavior_on_route`). Это лежит в очереди.
До тех пор настоящий обзор раздела — приведённая выше
дисциплина «свежее значение при каждой правке».