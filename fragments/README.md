# fragments/

## Карта папки

Кратко для оператора. Машинам и без этого хорошо — этот блок нужен,
чтобы человеку было видно, что в папке появилось и как этим
пользоваться, не залезая в JSON-ы.

**Что здесь лежит.** JSON-фрагменты двух типов. **Рефлексии** —
фиксации одного хода или цепочки ходов размышления, операторская
позиция плюс контекст. **Мета-спеки** — порождающие документы,
которые описывают, как строить класс артефактов; читаются ЛЛМ,
не оператором.

**Что с ними делать оператору.** В рефлексии можно заглядывать как
в записную книжку — вернуться, перечитать, развить. В мета-спеки
оператору заглядывать в норме не нужно: они для ЛЛМ, оператор
видит результат — конкретную машину или произведённый артефакт.

**С чего начинать чтение.** `what-makes-an-artifact-good.json` —
фундамент: пять свойств, по которым артефакт оценивается. Дальше по
нарастающей: `atoms-of-project-memory.json` про темп освоения,
`how-an-artifact-is-born.json` про процесс рождения,
`machine-spec-meta.json` про машины. Каждый следующий опирается на
предыдущие через `links_to_other_fragments`.

**Когда добавлять новый фрагмент.** Когда сегодняшний разговор
прошёл проверку отчуждения — то есть когда из него можно собрать
самостоятельный JSON, понятный свежей ЛЛМ без контекста чата. Если
не прошёл — оставить как разговор, не коммитить. Преждевременная
фиксация ломает жанр.

**Когда строить машину.** Только если задача проходит четыре
applicability_conditions из `machine-spec-meta.json`. Иначе машина
будет работать слабее, чем ручной процесс, и затея проиграет.


Reflection artifacts in JSON form. Each file is one molecule —
a self-contained crystallization of one moment of operator-LLM
thinking, with explicit source tracking (`from`), uncertainty,
gaps, and `not_canonized` records.

## Why this folder exists

Some thinking earns crystallization but does not fit any existing
genre: not a behavioral protocol, not a content protocol, not a
project memory leaf. It is a reflection — a captured shape of one
turn or one chain of turns. The standard place for these is here.

Each fragment is testable for detachment from chat: a fresh LLM,
given only the fragment file, should be able to read it and
reproduce the operator's shape of thought without the conversation
context. This was empirically validated 2026-05-04 across four
models (Claude, ChatGPT, DeepSeek, Gemini) on the first two
fragments below.

## Genre conventions

Fragments come in types, distinguished by `fragment_type`. The first
two are: `operator_reflection` (most fragments — captured shape of
one turn or chain of turns) and `meta_spec` (introduced 2026-05-04 —
JSON contract that defines how to construct a class of artifacts;
see `machine-spec-meta.json`).

A fragment is a single JSON file. Required fields:

- `fragment_type` — string, e.g. `operator_reflection`
- `fragment_id` — short stable identifier, also the filename
- `created` — ISO date
- `source_message` — brief reference to which turn produced this
- `context` — one to three sentences setting the surrounding
  conversation
- `core_assertion` — the central claim of the fragment
- `not_canonized` — list of formulations or moves that came up but
  were declined; without this field, future readers cannot tell
  whether something was rejected or simply forgotten

Optional fields shape themselves around the content. Common ones:
`observations[]`, `properties[]`, `open_questions[]`,
`agreed_next_step`, `links_to_other_fragments[]`, `candidate_*_held_behind_wall`.

New fields are allowed. The genre evolves with each fragment. If a
field appears in three fragments, it is moving toward becoming part
of the standard set; this is how the alphabet emerges from cases
rather than from prescription. Do not add fields just because they
sound clean — add them when a real fragment needs them.

## Pace

One case at a time. Do not pre-design the alphabet. Let the alphabet
show through accumulated molecules. See `atoms-of-project-memory.json`
for the operator's explicit position on this.

## Index

- **`what-makes-an-artifact-good.json`** — five properties of a good
  artifact: addressable, executable, detached from chat, born by
  system rules, therefore explainable. Created 2026-05-04 from a
  long operator turn synthesizing what made the protocols/modes work
  feel right.
- **`atoms-of-project-memory.json`** — operator's response to a
  proposal of ten standard atoms; central claim that an alphabet
  must emerge from cases, not be adopted as a list, otherwise the
  operator becomes a reader of his own system. Created 2026-05-04.
  Extends `what-makes-an-artifact-good`.
- **`how-an-artifact-is-born.json`** — operator proposed a formula
  for how an artifact is born; this fragment unrolls it from a sum
  into a trace through inputs, LLM, candidate, operator, application,
  system change. Created 2026-05-04. Extends `what-makes-an-artifact-good`.
- **`machine-spec-meta.json`** (v2) — first meta-spec in the system.
  Defines what a machine is (JSON contract with input/process/output)
  and how to build one. New fragment_type: `meta_spec`. v2 adds the
  input-vs-process discipline rule after empirical testing across
  three LLMs. Created 2026-05-04.