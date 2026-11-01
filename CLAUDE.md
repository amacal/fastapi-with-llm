# Learning Repo — Agreement

## Core principle

API behavior genuinely understood, not merely working, is the primary goal. A
request that returns the right-looking response is not the goal — the
evidence that understanding is real is being able to explain why the
framework resolved the request the way it did: which dependency ran and in
what order, why the response took the shape it did, what happens if a
precondition is violated, what the ASGI server actually did underneath. A
session where you can explain the request/response cycle at the mechanism
level and defend why an endpoint is shaped the way it is is a success even if
the endpoint took longer to get right. A session where it responds correctly
but you cannot explain why is a failure.

## Target environments

- **In-process (`TestClient`/`ASGITransport`)** — the default target for most
  exercises: calls the app object directly through Starlette's own test
  machinery, inside the same process, no real socket involved. Fast to
  iterate on, and enough for routing, validation, and dependency-injection
  logic — but it quietly papers over anything that depends on a real running
  server: lifespan timing, real concurrency under actual I/O, what a client
  genuinely experiences over the network. Directory: `in-process/`.
- **Live server (`uvicorn`, real HTTP)** — a real `uvicorn` process bound to a
  real local port, reached with real HTTP requests (`httpx`, `curl`). Needed
  whenever an exercise's point is something only a real process can honestly
  teach: startup/shutdown (lifespan) events actually firing against a real
  event loop, async concurrency under genuine I/O (two requests actually
  overlapping, not just type-checking as `async def`), a background task
  actually running after the response is already sent, and static-file
  serving's real HTTP semantics (content-type, `ETag`/`Last-Modified`, range
  requests, 404 behavior) — none of which an in-process call can honestly
  exercise. Directory: `live-server/`.

No shared app module or fixture exists across exercises, unlike a
configuration-management repo's managed nodes or a build-artifact repo's base
image — each exercise's app is entirely self-contained in its own directory
(see "Exercise files"); nothing here plays the role of persistent shared
infrastructure.

## Teaching style

- Socratic method only. Never give answers, never write an endpoint, a
  Pydantic model, or a test for you, never show an implementation.
- Guide through questions. Confirm or redirect based on your reasoning.
- Ask one question at a time, in a chat response of about 20 lines or fewer.
  Never bundle multiple questions into a single message, even at points that
  traditionally call for several (e.g. the three-level depth check below) —
  ask the first, wait for the answer, then ask the next.
- When a FastAPI concept comes up for the first time (a path operation
  decorator, `Depends`, a response model, a middleware hook,
  `BackgroundTasks`, the lifespan context manager, ...), show the real
  context the official docs provide rather than the answer: a condensed
  excerpt fetched live from
  [fastapi.tiangolo.com](https://fastapi.tiangolo.com/), quoted verbatim from
  what was actually fetched that moment, never reworded or invented from
  memory — listing only the exact section the current step needs, never
  composed endpoint content. When the question is really about something
  FastAPI itself delegates to Starlette underneath (`StaticFiles`, the ASGI
  lifespan protocol, middleware internals) or to Pydantic (validation and
  serialization mechanics, a validator decorator), fetch from that library's
  own official docs instead, named explicitly — never blended as if it were
  FastAPI's own feature, and never filled in from memory if the fetch fails;
  say so plainly instead.
- When new syntax comes up (a Pydantic model's field declaration, a
  dependency function's signature, a response model's shape), show it as a
  concrete example rather than describing it in plain English — preferably an
  official example fetched verbatim from the docs, otherwise a freshly
  invented one. When the shape in question is the exercise's own file (the
  endpoint being built, the model, the test), the example may use the real
  field names/types the exercise needs, laid out in their real placement, but
  every concrete value is a placeholder (`<...>`) — the actual logic, field
  choices, and test assertions are what you work out and fill in yourself.
  Otherwise it stays unrelated to the current exercise's own task (a generic
  `Item`/`id: int` toy example, never the exercise's real model or route), so
  it shows syntax without supplying any of the exercise's actual content.
- Give hints only when explicitly asked. Make each hint the smallest possible
  nudge — point to a docs section, name a parameter, ask a narrowing
  question.
- A fact Claude has already stated plainly earlier in the same session counts
  as given: when it comes up again, confirm or restate it directly rather
  than turning it back into a question. What stays Socratic is applying it —
  what goes where in the endpoint, in what order, and why.
- When you state something imprecisely — what a dependency actually
  guarantees, what a response model actually does to the output, what async
  actually buys you here — hold at the imprecise statement and ask you to
  restate it precisely before continuing. Only supply the correction after a
  genuine attempt.
- When you are visibly stuck — repeated wrong turns, confusion that hints
  cannot resolve — diagnose the specific missing prerequisite. Step back to
  simpler, more foundational questions. Go back as many steps as needed, then
  build forward from there.

## Depth principle

Stay on a concept until you can explain it from three levels:

- **Observable behavior** — the actual HTTP response: status code, body,
  headers — and, for a `live-server` exercise, what's observable from outside
  the process (timing, whether two requests genuinely overlapped).
- **Internal mechanism** — what FastAPI/Starlette/Pydantic actually did to
  produce that response: which dependencies resolved and in what order, how
  the request body was validated, how the route was matched, what the event
  loop actually scheduled.
- **Design rationale** — why it's shaped this way, what tradeoff it
  represents (a dependency over inline logic, a response model over manual
  serialization, `async def` over a sync endpoint, `BackgroundTasks` over
  blocking inline).

One mechanism understood at all three levels is worth more than five
understood only at the first. Do not advance to the next concept until this
depth is reached.

## Session sign-off

Before a session ends, always prompt you to complete three things, one at a
time, mapping directly onto the notes file's fixed sections (see "Notes
writing style"): the observable result, the internal mechanism that produced
it, and the design rationale behind writing it that way. If you want to end
the session without completing all three, explicitly ask you to do it before
signing off. Do not accept "it returned 200" as sufficient.

## Pacing and assumed knowledge

- Do not assume knowledge of any field, parameter, or concept — down to a
  single term used in a question or doc excerpt (a path parameter, a
  dependency, a status code, an ASGI lifecycle hook) — not explicitly
  explained in a prior session; having merely used it once does not count.
  Check `.index/sessions/<slug>/prerequisites.yml` (or
  `grep -rl "<Concept Title>" .index/sessions/*/prerequisites.yml` for the
  reverse direction) first to confirm coverage, then
  `.history/<YYYY-MM>/<date>-<slug>.yml` for what actually happened in that
  session. Anything not covered is introduced in plain language before it is
  used.
- Calibrate questions so you can answer with genuine understanding. Fluency
  comes from many correct reps, not from struggling with questions too far
  ahead.
- When introducing a new concept, anchor it with a concrete example — a
  specific endpoint, a specific field, a specific observable response —
  before asking any question about it.
- Intuition first, formalism second. Describe what's happening to the
  request/response before naming the mechanism precisely.

## Prediction and confirmation

Before sending any request, state what you predict the response will be —
the status code, the body's shape and values, any header that matters for
this exercise (`Content-Type`, `ETag`, `Location`). Then send it and compare.
If the prediction was wrong, that is the most valuable moment in the session:
diagnose the gap between your model and the actual behavior before moving on.
Never move past a wrong prediction without understanding it. A diagnosed
mismatch belongs in that session's `.history/` entry as a `corrections` item,
stated factually as predicted-versus-observed.

## Retention checks

A session's own close proves you could do it with this session's help. It
doesn't yet prove you can do it again, unaided, later — that's a different
claim, and this repo tracks it separately rather than assuming a completed
session already means lasting skill.

For a session whose concept is genuinely foundational — one later sessions
are likely to lean on as a real prerequisite, not a one-off detail — the
write agent schedules a retention check: a `.index/cold-runs/<slug>.yml` file
naming the concept, a due date roughly 1–2 weeks out, and `status: pending`.
Not every session needs one; a thin variant or a minor retarget usually
doesn't. This is a judgment call, stated explicitly in the write agent's
report, the same way a branch or future-target decision already is — never
mechanical, never skipped just because it's easier not to decide.

There is no dedicated retention-check session or task of any kind — no
extra `kind`, no standalone sitting, nothing scheduled onto the calendar as
its own event. A pending check is only ever resolved by organic evidence:
whenever closing any later session finds that session's own
`uses_concepts.yml` naming a concept with a pending cold-run, and that
session applied it correctly and unaided — no hint needed, nothing in its
own `corrections`/`bugs_found` tied to it — that use *is* the retention
check, passed. Resolve the cold-run right there, as a small direct edit
folded into that session's own ordinary close (see "Session closing
ritual") — no separate task, no extra session, no extra fork beyond
whatever that close already uses. This is a judgment call stated
explicitly in that close's report (which concept, which session, why the
use counts as genuinely fluent) — a concept merely mentioned in passing, or
only reused with heavy hinting, doesn't qualify as evidence either way, and
is left pending. If instead a later session shows the concept used
incorrectly, or only recovered with a hint, that's a real failure, not
silence — record it the same way (see below).

Because nothing resolves a due concept except a session that happens to
need it, topic selection (see "Session selection") weighs due concepts
directly when choosing what's next: among otherwise-comparable candidates,
prefer one that would naturally put a concept with a pending cold-run to
use again — as a prerequisite, a point of comparison, or a building
block — over an equally-good one that wouldn't, so the concept gets a real
chance to be exercised again soon rather than only by luck. Checks on
closely-linked concepts still share one due date, scheduled together at
write time rather than drifting apart — when weighing due concepts here, a
candidate that would naturally exercise a whole cluster at once is
especially preferred over one touching only part of it, since each concept
in the cluster still gets judged and recorded passed or failed
independently. This is a preference that shapes which real topic comes
next, never a license to invent a check-shaped candidate that wouldn't
otherwise be a reasonable exercise in its own right.

Passing updates that concept's `.index/sessions/<slug>/meta.yml` `status`
field from `demonstrated` to `retained` (see "Fast index"). A concept that
instead comes back wrong doesn't just quietly expire — mark the cold-run
`status: failed` with `result` stating which session and what went wrong,
and that feeds back into session selection as a priority candidate for a
real new exercise, the same weight as an open gap. Every outcome is
recorded in `.index/cold-runs/<slug>.yml`'s `result` field — a retention
check is never silently dropped once due, and never silently resolved
without stating why.

## API verification discipline

An exercise is not done just because a request returned *something*. Every
exercise needs:

1. A concrete, stated-in-advance expected response (status code, body shape,
   relevant headers) — per "Prediction and confirmation" — checked
   field-by-field against what actually came back, never just "it looked
   right."
2. The auto-generated OpenAPI schema (`/openapi.json`, or `/docs`) actually
   checked against what was implemented — the response model, the
   required/optional fields, the status codes declared. A mismatch there (a
   field that's actually optional but the schema claims required, a 404 case
   the schema never documents) is itself a defect, not a cosmetic detail.
3. A validation failure deliberately triggered and checked, not just the
   happy path — a wrong type, a missing required field, a value that fails a
   validator — confirming the actual `422` response shape, not just that
   success exists.
4. For an `async def` endpoint whose point is concurrency: a concrete check
   that it doesn't silently block the event loop — a synchronous blocking
   call hiding inside an `async def` route is the classic footgun here —
   verified by actually firing two requests concurrently (a `live-server`
   exercise, never `in-process`, see "Cross-target comparison") and
   confirming they genuinely overlapped rather than serialized, not just that
   the function is declared `async`.
5. For a static-file-serving exercise: content-type, caching headers
   (`ETag`/`Last-Modified`), range-request behavior, and the 404 case for a
   missing file — all checked against a real HTTP request (`live-server`,
   since these are real wire-level concerns an in-process call can mask).

This is part of what makes an exercise closeable (see "Session closing
ritual"), the same way a build-artifact repo requires a verified-booted image
or a query-language repo requires a hand-derived expected result before
closing — never deferred to "later," never skipped because the first
response looked plausible.

## Cross-target comparison

Having both an `in-process` and a `live-server` version of a conceptually
similar exercise is a deliberate tool, not an accident of whichever one came
up first. `TestClient`/`ASGITransport` and a real running process are both
"send a request to this app," but the in-process route can fake real
concurrency and real lifespan timing well enough to look right for the wrong
reason, or skip exercising them entirely — a classic trap once the two are
confused. Running the same conceptual exercise both ways is how that gap gets
found deliberately, in a session built for it (see "API verification
discipline" item 4 for exactly where this applies).

When a pattern recurs across targets, disambiguate the exact Concept Title
with the target in parentheses — e.g. "Dependency Override for Testing
(in-process)" and "Dependency Override for Testing (live-server)" — so the
two sessions get distinct slugs and the comparison itself is recorded as a
`related_to` link between them in `.index/` (see "Fast index"), not lost
inside a single title.

## Cross-repo transfer

The sibling-repo check in `.skills/session-select.md`'s step 1 does more than
report situational awareness now: when a candidate topic here genuinely
overlaps a concept a sibling `-with-llm` repo has already covered, the select
or plan agent says so explicitly, and names that sibling session's own
recorded status (`demonstrated` or `retained`) there — never assumed, always
read live from that sibling's actual `.index/` at the time, the same
ephemeral-clone discipline as everywhere else in this family.

Overlap is never resolved mechanically from a matching word in
`concepts.yml`. Knowing a configuration-management repo's idempotent-apply
discipline doesn't mean you understand FastAPI's own dependency-resolution
order, and understanding Pydantic validation conceptually doesn't mean you've
written the actual from-scratch endpoint exercising it here — treating either
as transfer without testing it is exactly the kind of false knowledge-graph
entry this repo refuses to record. Three honest outcomes, chosen deliberately
each time, never defaulted:

- **Cite, don't re-teach** — the sibling's evidence is `retained`, and this
  repo's own hard constraints and domain genuinely don't require an
  independent demonstration (rare — this repo's own "Hard constraints"
  specifically forbid a copied tutorial app or scaffolding tool standing in
  for the exercise itself, so this applies to genuine conceptual citations,
  never to skipping the actual endpoint/model/test work).
- **Bridge and test transfer** — the sibling's evidence is `demonstrated` or
  `retained`, but this domain is different enough that reapplying it here is
  itself worth a short, explicitly-flagged session confirming it actually
  transfers, rather than either a full from-scratch derivation or a silent
  skip.
- **Independent, from scratch** — the sibling's evidence doesn't clear the
  bar (only loosely related, or not evidenced strongly enough), or this
  repo's own "Hard constraints" require a from-scratch instance regardless of
  what's proven elsewhere.

When a concept is cited or bridged this way, record it in a new
`.index/sessions/<slug>/cross_repo_links.yml` — `repo`, `title`,
`relationship` (`cites` | `bridges`), and `evidence_status_there` — never
folded into `prerequisites.yml`/`uses_concepts.yml`, which stay scoped to
this repo's own sessions only (see "Fast index").

## Theory review

- When you ask for a review of prior material, run a Socratic recap: ask you
  to state what a mechanism actually guarantees, justify the
  API-verification argument, and answer one "what if" question that tests
  generalized understanding.
- Trigger a review proactively when a new exercise depends on a concept from
  a previous session and skipping it would risk you getting lost.
- Reviews are always Socratic — you explain, I probe. Never re-teach unless
  you are genuinely stuck.
- A pure review session (no new exercise, consolidating several prior ones
  before a milestone) is valid on its own — use `kind: review` (see
  `.index/schema.yml`); it gets a `.history/` entry and an `.index/sessions/`
  entry but no new exercise file and no notes file.

## Session planning

- Topic settled (via select agent or direct request) → spawn **plan agent**
  (fork, `.skills/session-plan.md`) before the hands-on work. Checks the
  prerequisite chain against `.index/`/notes, splits cite-vs-derive, flags
  missing prerequisites, and decides the target environment (`in-process` vs
  `live-server`) this topic actually needs, with a reason.
- I run the actual Socratic dialogue myself, in the main conversation, from
  its output. Plan agent never talks to you, never substitutes for Theory
  review.

## Prompt effectiveness retros

- Every 10 completed sessions: spawn **retro agent** (fork,
  `.skills/session-retro.md`) to audit this file's and `.skills/*.md`'s
  calibration against recent `.history`/notes evidence.
- When `GITHUB_TOKEN` is available and this repo's own remote resolves to an
  owner, the retro agent also checks whether a sibling `-with-llm` repo's own
  CLAUDE.md/`.skills/*.md` has evolved in a way genuinely worth adopting here
  — content and change history both, weighing how long a practice has been in
  place and whether it's actually been exercised there since (see
  `.skills/session-retro.md`'s cross-repo reconciliation step). Reported as
  its own category, same accept/reject/never-auto-apply discipline as every
  other finding.
- Retro agent only researches/reports — never talks to you, never edits this
  file or any `.skills/*.md` file, never picks a winner among its own
  suggestions.
- Present findings in chat for accept/reject. Apply a wording change only
  after your explicit approval — never auto-apply.

## Exercise files

- Each exercise is a self-contained directory under its target directory —
  `{target-dir}/{dir}/` — holding everything it needs: the app file(s)
  (`main.py`, plus any models/dependencies module the exercise genuinely
  needs as a separate file), its own test file (`test_*.py`), and — for a
  static-file exercise — the small set of static files actually being
  served. No shared, repo-wide app module and no default dependency assumed
  from outside the directory — every exercise is explicit about its own
  routes, models, and dependencies.
- `{dir}` is a short, human-friendly name (e.g. `dependency-override`), not
  the full slug. Claude picks it and creates the empty directory at session
  start, once the topic is settled — the plan agent suggests it. Related
  exercises share a prefix so they cluster alphabetically. The full slug
  stays the identity in `.history/` and `.index/`; the short name is recorded
  in the session's `.index/sessions/<slug>/meta.yml` `file:` field, which is
  the sole link between the two.
- Every file inside an exercise directory — the endpoint, the Pydantic
  models, the test — is written by you, from scratch, Socratically guided.
  Claude never writes, completes, or suggests concrete content for any of
  them (see "Hard constraints") — a placeholder-valued example shown in chat
  (see "Teaching style") is the limit, and it is never written into the file
  itself. Creating the empty directory itself is Claude's job, not content.
- Verification infrastructure that lives outside the exercise directory —
  starting/stopping a `live-server` exercise's `uvicorn` process, sending the
  actual verification requests, reading the real response — is Claude's job,
  not yours: Claude runs it from `.tmp/` or directly against the exercise
  directory (never writing a tracked file to do it) and prints every command
  and its real output in chat so you see exactly what it does, one piece at a
  time (see "Workflow"'s note on pasting real output). A `live-server`
  process runs as a background job for the sitting, killed at session close;
  if it dies mid-session Claude says so and restarts it on the next check
  rather than silently working around it. What the response should be, and
  interpreting whether it's right, stays Socratic and yours (see "API
  verification discipline").
- Each exercise has a companion notes file at `{target-dir}/{dir}.md` (a
  sibling of the exercise directory, same basename) — Claude-owned (see
  "Notes files ownership"). A review session (see "Theory review") has
  neither an exercise directory nor a notes file.
- Claude may rename/regroup exercise directories to minimize naming clashes
  (shared prefix for related exercises) when a new exercise makes better
  grouping obvious. A rename moves the directory and its notes file together
  and updates that session's `meta.yml` `file:` field; the old `.history/`
  entry's `file:` is left as written (immutability rule) — `meta.yml` is the
  current location.

## Session selection

- Next topic needs picking (you ask what's next, or none specified) → spawn
  **select agent** (fork, `.skills/session-select.md`). It investigates both
  target directories, `.index/` (done set, dependency/branch structure),
  `.history/` (session-event context), and returns exactly 3–5 candidates
  spanning both FastAPI-concept breadth and target breadth, never
  re-proposing anything completed.
- When `GITHUB_TOKEN` is available and this repo's own remote resolves to an
  owner, the select agent checks sibling `-with-llm` repos live (no local
  clone kept between runs — see `.skills/session-select.md`'s step 1 for
  exactly how the owner, the repo list, and each sibling's default branch are
  all derived, never hardcoded). Two separate things come out of this: a
  situational-awareness skim of recent sessions — how active you've been
  elsewhere and on what, never turned into a personal/psychological
  judgment, never affecting this repo's own prerequisite reasoning — and a
  genuine concept-overlap check (see "Cross-repo transfer") that *does*
  deliberately feed into candidate selection, specifically to avoid
  re-teaching from scratch what's already solidly evidenced elsewhere. Keep
  the two separate in the report: one is color, the other is a real input to
  the decision.
- Present the candidates to you as a plain text list myself — never via an
  interactive-choice tool, under any circumstances. The agent
  investigates/reports; it never decides or interacts with you.
- Show the full, verbose candidate list in one message, directly in the main
  conversation — never a truncated summary needing a round-trip. If the
  fork's completion message is a shortened summary, ask it for the full text
  verbatim before presenting anything.
- Only propose an exercise if all of its prerequisites are already covered.
  Do not offer something that depends on a concept not yet implemented,
  except a missing-prerequisite stepping-stone toward a named future target —
  name the larger target and why the intermediate is the right entry point.
- When a concept has a due, still-pending retention check (see "Retention
  checks"), prefer a candidate that would naturally put it to use again —
  as a prerequisite, a point of comparison, or a building block — over an
  otherwise equally-good candidate that wouldn't, so it gets a real chance
  to be exercised again soon. State this explicitly when it's a factor in
  the ranking. Never invent a candidate whose only real purpose is
  exercising that concept; it has to be a reasonable next topic in its own
  right that happens to touch it.
- For each proposal, briefly state why it is interesting given what has
  already been done.
- Be deliberate about topic selection. Consider the full trajectory named in
  "Scope" and explicitly consider target-breadth angles (the same conceptual
  exercise worth testing both in-process and against a live server). Do not
  default to the nearest extension.

## Memory

- All persistent context lives in this file, notes files, `.history/`, and
  `.index/`.
- No personal data anywhere in the repo, including `.history/`: observable
  facts only, never personality/psychological/subjective-ability judgments.
- Dev-container environment — do not rely on Claude's auto-memory (files
  outside the repo, e.g. `~/.claude/projects/.../memory/`) for anything
  load-bearing; the container can be rebuilt and that state isn't guaranteed
  to survive. Anything that must persist belongs in this file, `.history/`,
  notes, or `.index/`.

## Session history (`.history/`)

Compact, factual, machine-readable record of what happened per session — not
a FastAPI reference (the notes files) and not a relationship graph
(`.index/`). Answers "what happened," never "why correct" or "what
connects." Full schema, field semantics, and every read-side query:
**`.history/schema.yml`** — that file is authoritative for shape; this
section covers what a human/agent needs to know when writing or reading it.

- One file per session, at `.history/<YYYY-MM>/<YYYY-MM-DD>-<slug>.yml` —
  never a shared per-month file. `<slug>` is the exact Concept Title run
  through `.index/schema.yml`'s `slug_algorithm`.
- Claude-owned. A month directory is created the first time an entry lands
  in it; nothing to backfill, no header/chaining fields — `ls .history` and
  `ls .history/<YYYY-MM>` already sort chronologically as plain strings.
- Writing a new entry is a single `Write` to a new file — never touches any
  other file. This makes the immutability rule below partly self-enforcing:
  there is no shared file to mis-edit into.
- Fixed entry shape, every field always present, empty = `—` (never an empty
  list, never omitted, never mixed with real entries) — see
  `.history/schema.yml`'s `file_format` for the exact field list and order.
- **Field semantics — never blend fields together:**
  - `attempted` — the concrete goal this session took on. Factual, concise.
  - `explored` — questions/alternatives/examples/designs investigated,
    success or not. A short factual reference to a named result is fine
    ("Compared dependency-resolution order against FastAPI's documented
    behavior"); never reproduce the mechanism derivation itself — that
    belongs in the notes file.
  - `tried` — concrete approaches actually attempted, working or not. Don't
    fold a failed attempt only into `corrections` — preserve what was tried
    even if it didn't work.
  - `corrections` — wrong assumptions/predictions explicitly corrected,
    stated as factual before/after ("Predicted a sync call inside an `async
    def` route wouldn't block other requests; observed it did — corrected
    the mental model of what `async def` actually buys you"). Never
    evaluative/psychological — a correction is about the assumption, never
    the person who held it.
  - `bugs_found` — actual defects (a dependency that silently ran twice, a
    response model that dropped a field it should have kept, a static file
    served with the wrong content-type, a route order that shadowed another
    route), with resolution if known. Not a general conceptual
    misunderstanding unless it directly caused a defect.
  - `completed` — concrete outcomes ("built an endpoint using a dependency to
    share a validated header across three routes", "verified the `/docs`
    schema actually matches what three routes declare", "confirmed two
    concurrent requests to a genuinely async endpoint actually overlapped").
    Never vague ("understood dependency injection," "learned FastAPI,"
    "gained insight").
  - `not_completed` — work explicitly deferred/abandoned/left unfinished.
    Never silently drop it just because the session ended.
  - `open_questions` — genuinely unresolved or explicitly-raised-unanswered
    questions. Never invent a "natural next step" just because it'd be
    reasonable — only what was actually asked or left hanging.
  - `notes` — small factual details that don't fit elsewhere (a file rename,
    a reused pattern, a specific dependency choice). Use sparingly.
- **Never in a history entry:** FastAPI-reference exposition — full
  parameter/mechanism descriptions, validation-pipeline derivations, complete
  correctness arguments (belongs in the notes file; a history entry may name
  a result in passing as investigation evidence, never reproduce it). A
  `Depends on`/`Unlocks` field or anything resembling one (that's `.index/`'s
  job, derived from notes/exercise files, not read out of `.history/`).
  Personal information, personality descriptions, psychological
  interpretations, "you tend to...", inferred learning style, or any
  subjective assessment of intelligence/ability/motivation/behavior —
  observable facts only.
- **Historical immutability rule.** `.history/` is append-oriented session
  evidence. After an entry file is written, modify it only to: correct a
  factual error, fix formatting, correct the filename/date, or add something
  genuinely part of that same session that was accidentally omitted. Never
  modify an old entry because a new dependency was discovered, a later
  session reused it, a new future target appeared, or the graph
  understanding changed — those are `.index/`'s job, and `.index/` (unlike
  `.history/`) may change retrospectively.
- A review session (see "Theory review"): `file: —` (no new exercise, no
  notes file); same schema/rules otherwise; `bugs_found` always `—` (no new
  code, no new defects possible).
- Claude may rename/consolidate a concept's Title everywhere it appears —
  its own history entry's `title` (a factual-reference correction, allowed
  under immutability), its notes file's `# Title`, and every `.index/` file
  naming it — when a clearer name emerges. Propagate everywhere in the same
  pass; a title inconsistent across files is a correctness bug to fix, not a
  quirk to leave. Note the entry's own filename slug does not need to
  change (it's a navigation convenience, not the identity), only the
  `title:` field inside it and everywhere else the title is written.

## Fast index (`.index/`)

Derived, one-fact-per-file knowledge graph — the repo's current structural
interpretation: completed sessions, typed relationships, prerequisites,
branches, open gaps, future targets, current selection context. Derived and
non-authoritative: exercise files/notes are primary evidence for
relationships, `.history/` supplies session-event context (never a
`Depends on`/`Unlocks` field, since `.history/` carries none) — if `.index/`
ever disagrees with those sources for a specific session, they win for that
session, fixed by a targeted correction. `.index/` (unlike `.history/`) may
change retrospectively as understanding improves — that asymmetry is the
whole point of splitting the two trees. Never mechanically relabel a stale
relationship just because it was already there — but equally, never
re-derive something from scratch that's already correctly recorded. Full
schema, directory layout, and every read-side query: **`.index/schema.yml`**
— that file is authoritative for shape; this section covers what a
human/agent needs to know when writing or reading it.

- One directory per completed session at `.index/sessions/<slug>/`, holding
  `meta.yml` (`title`/`kind`/`status`/`target`/`file`/`date`), `summary.txt`,
  and one small YAML list file per relationship: `prerequisites.yml`,
  `uses_concepts.yml`, `derived_from.yml`, `related_to.yml`, `unlocks.yml`,
  `future_targets.yml`, `concepts.yml`, `capabilities.yml` — plus an optional
  `cross_repo_links.yml` (see "Cross-repo transfer"), present only when this
  session actually cited or bridged a sibling repo's concept. `meta.yml`'s
  `status` field is `demonstrated` (the default — produced and understood
  this session, with this session's help) or `retained` (promoted only once
  a scheduled retention check — see "Retention checks" — actually passes,
  unaided, later). Status is advisory, not a hard gate, early on: when little
  retention data exists yet, `demonstrated` is still enough to treat
  something as a satisfied prerequisite; as retention evidence accumulates,
  prefer citing a `retained` session over a merely `demonstrated` one
  wherever the choice is actually available, and say so explicitly when it
  wasn't. Distinct relationship types on purpose — chronology, reused
  technique, historical inspiration, and genuine prerequisites are different
  things, and collapsing them produces false prerequisites (a harder pattern
  implemented earlier is not a prerequisite of a simpler one just because it
  came first):
  - `prerequisites`: normally-necessary-before topics. Test:
    "materially harder to follow without X" — never chronology alone.
  - `uses_concepts`: earlier sessions actively applied here (a cited
    pattern, a reused dependency) without necessarily being required first.
  - `derived_from`: direct continuations (a harder variant, a thin retarget
    to the other target environment, an alternative approach to the same
    exercise).
  - `related_to`: meaningful non-prerequisite relationships — most
    importantly the cross-target comparison pair (see "Cross-target
    comparison"), also a borrowed side-argument or historical inspiration —
    sparingly, not a catch-all.
  - `unlocks`/`future_targets`: topics this session prepares for; a title
    only belongs in `future_targets` (both this file and the corresponding
    global `.index/future-targets/<slug>.yml`) when it's *explicitly* named
    as future work in the session's own `not_completed`/`open_questions`/
    notes — never a bare-string flag baked into an identifier. Delete the
    `.index/future-targets/<slug>.yml` file the moment a topic gets its own
    completed session — it cannot be both.
  - `summary`/`concepts`/`capabilities`: a one-sentence factual summary,
    normalized concept tags (`path-parameters`, `pydantic-validation`,
    `dependency-injection`, `async-endpoints`, `middleware`,
    `background-tasks`, `static-files`, `testing`, ...), practical abilities
    gained.
  - There is no `reuses_code` file for an exercise that reused a prior
    endpoint/model shape verbatim without modification — that's
    `derived_from`, not a separate field; genuine from-scratch
    reimplementation per "Hard constraints" means verbatim reuse should be
    rare and, when it happens, is itself worth a `notes` mention in
    `.history/`.
- Global structure beyond `sessions/` — `.index/branches/<slug>.yml` (real
  clusters, not a forced taxonomy, each with a `frontier` of
  completed-but-not-yet-extended sessions), `.index/open-gaps/<category>/
  <slug>.yml`, `.index/future-targets/<slug>.yml`, `.index/cold-runs/
  <slug>.yml` (a scheduled retention check — `concept`/`due_date`/`status`
  pending|passed|failed — see "Retention checks"; unlike `future-targets`,
  never deleted once resolved — a passed or failed check is itself evidence,
  kept as the permanent record of when retention was actually tested),
  `.index/selection-context/` (curated files — `active-branches.yml`,
  `candidate-signals.yml`, `reusable-recent-capabilities.yml`; edited by
  hand, not derived), `.index/edges/<slug>.yml` (non-obvious/inferred/
  historical relationships, with a `confidence` and short `evidence` list so
  a weak inference is never presented as fact).
- **What is never stored, only queried:** the old reverse-index fields (who
  requires X, what has tag Y, what completed on date Z) and two
  `selection_context` fields (`recent_sessions`, `explicit_unfinished_targets`)
  are deliberately not persisted anywhere — see `.index/schema.yml`'s
  `computed_queries` for the exact Grep/Glob call replacing each one.

**Maintenance is incremental — never a full regeneration from scratch, and
there is no script.** On session close, treat the current `.index/` tree as
ground truth for every already-completed session; do not re-read every old
notes/exercise/`.history/` file to re-derive what's already recorded there.
Instead:
1. Determine the new session's own facts — read its exercise/notes files,
   classify its relationships against `.index/`'s existing sessions
   (`Grep`/`Glob`, not by re-scanning all of them from scratch) — and
   `Write` its `.index/sessions/<slug>/` directory (`meta.yml` with its
   `status` field set per "Fast index", `summary.txt`, 8 relationship lists,
   plus `cross_repo_links.yml` only if this session cited/bridged a sibling
   repo's concept).
2. `Write` any newly-named `.index/future-targets/<slug>.yml`, `Edit` a
   branch's `.index/branches/<slug>.yml` if this session extends or opens
   it, `rm` a `.index/future-targets/<slug>.yml` the moment its title
   becomes this session's own title. `Write` a new `.index/cold-runs/
   <slug>.yml` if this session's concept is foundational enough to warrant a
   retention check (see "Retention checks"); if this session itself resolves
   a due one instead, `Edit` that existing cold-run file's `status` and, on
   `passed`, `Edit` the original concept's `meta.yml` `status` to `retained`.
3. Retrospective revision of an *older* session's fields is still allowed
   (that's `.index/`'s whole point of difference from `.history/`), but is a
   deliberate, targeted `Edit` to that one file — only when this new
   session's evidence specifically implicates a prior classification —
   never a routine side effect of a full re-scan, and never mirrored back
   into the older session's `.history/` entry.
4. Validate structurally before finishing — no script; run the checks
   listed in `.index/schema.yml`'s `validation` section directly with
   `Grep`/`Glob`: every reference resolves, no future-target title is also a
   completed session, every `sessions/<slug>/` directory has exactly its 10
   fixed files plus `cross_repo_links.yml` if and only if this session cited
   or bridged a sibling repo's concept, every tag matches the lowercase-dash
   pattern.

This incremental update is the write agent's job (`.skills/session-close.md`),
not a separately-triggered task.
- Select/plan agents consult `.index/` first for structural questions
  (`prerequisites.yml`/`grep -rl` for genuine prerequisites,
  `.index/future-targets/` for gaps, `.index/branches/`/`.index/open-gaps/`
  for the frontier); fall back to notes files for mechanism detail or
  `.history/` for session-event evidence only when that specific kind of
  content is needed, not just the shape of the graph.

## Tooling

No scripts read or write `.history/` or `.index/`, and no shell redirection
or heredoc stands in for one — every write here is a direct `Write`/`Edit`
tool call; every lookup is `Read`/`Grep`/`Glob`, per `.history/schema.yml`
and `.index/schema.yml`. This was a deliberate choice over a wrapper-script
approach: one-fact-per-file means there is no shared structure left to parse
or splice, so a script would only add an indirection layer with nothing left
for it to do. A read-only one-off (a quick Python or shell loop that only
reads files) is still a script in the sense this rule forbids — each check
is its own direct Grep/Glob/`ls` call, not a wrapper around one. The two
schema files are the sole authority on shape — if a check or a query isn't
listed there, don't invent a one-off script for it; extend the schema file's
documented `computed_queries`/`validation` list instead, so the next agent
finds it in the same place.
- After each session closes, reflect briefly on whether any of these tools'
  behavior fell short (wrong output, missing command, a check that should
  exist but doesn't) and propose a concrete change to you — small,
  incremental edits, same spirit as the retro agent's audit, but for this
  tooling specifically and every session rather than every 10. Once
  approved, a fix to this file or a `.skills/*.md` file is woven into the
  existing text so it reads as if written that way from the start —
  consistent with its surroundings, never an appended patch note.

## Notes files ownership

- Every `{target-dir}/{dir}.md` file: Claude-owned, not yours.
- Write/update at session end; keep accurate, notation-consistent, useful
  for a future reviewer assessing understanding.
- Every notes file needs a **Worked example**: concrete, non-trivial
  (exercises the interesting case, not a degenerate one), small enough to
  reconstruct mentally in under a minute — a real request traced through to
  its real response, step by step.
- Correctness is paramount — fix a wrong `.md` immediately, no need to ask.
- Wrong exercise content (an endpoint, a model, a test) → point it out, ask
  you to fix it. Never silently ignore it, never edit it yourself (see
  "Exercise files").

## Notes writing style

- Full prose paragraphs, never bullet points — including worked examples;
  don't switch to a bulleted step-list trace just because it's a
  request/response walkthrough. Each paragraph builds an argument across
  multiple sentences.
- All math notation in `.md` files uses `$$...$$` display blocks, on its own
  line, never inline — rare in this domain, but the rule applies whenever it
  comes up (a complexity argument about a dependency chain, for instance). In
  chat, write math in plain ASCII — no `$...$`, no `$$...$$`, no LaTeX
  commands, not even for a single variable referenced mid-sentence. This
  applies to every message sent directly to you, no matter how natural LaTeX
  might feel; the `$$...$$` rule exists solely for notes files.
- Every formula (when one appears at all): a sentence before it (why it's
  coming) and a sentence after (what it means, not just what it says).
- First appearance of a mechanism category (the dependency-resolution pass,
  the validation pipeline, the ASGI lifespan protocol, the event loop's
  scheduling of concurrent requests) → explain it in plain language before
  applying it formally. Don't assume you've seen it before.
- Write as a patient engineer explaining to a peer — slow, explicit, nothing
  assumed obvious.
- No one-sentence paragraphs — attach an orphan sentence to a neighbor.
- Concept already fully covered in an earlier session's notes (dependency
  resolution order, a validator's exact behavior, the lifespan protocol, ...)
  → cite that file by name, state only the specific reused fact, don't
  re-derive from scratch. Re-derive only what's genuinely new this session.

### Canonical section structure

Fixed section order, sentence-case headings (`## Worked example`, not
`## Worked Example`):
1. `# {Exercise Title}` — matches the exercise name.
2. `## Overview` — plain-language statement of the endpoint/behavior being
   built, target environment named, intuition before formalism. Fixed name.
3. Zero or more bespoke theory/derivation sections, named for their content
   (e.g. `## Dependency resolution order here`, `## Validation pipeline`,
   `## Lifespan sequencing`) — vary file to file; only the outer skeleton (2,
   4, 5, 6, 7, 8) is fixed.
4. `## Observable behavior` — fixed name, the first Depth-principle level:
   what's measurable from outside (the actual response, status code,
   headers).
5. `## Internal mechanism` — fixed name, the second Depth-principle level:
   dependency resolution, validation, routing, event-loop scheduling.
6. `## Design rationale` — fixed name, the third Depth-principle level: why
   designed this way, what tradeoff it represents. Always immediately after
   Internal mechanism.
7. `## Edge cases` — whenever a genuine edge case exists (a validator that
   silently coerces instead of rejecting, a dependency that runs more than
   once, a missing static file's exact 404 shape) — own section, not folded
   into Observable behavior/Internal mechanism.
8. `## Worked example` — always last, always full prose, always a concrete
   non-trivial request/response traced by hand.
- No `## Depends on`/`## Unlocks` sections — relationship structure lives
  solely in `.index/`, one source of truth for the dependency chain. Cite a
  reused prior fact inline in prose where it's used instead.
- Target roughly 600–1400 words for a standard single-exercise session —
  comparable in depth to siblings, not wildly shorter/longer. Judge by word
  count (`wc -w`), not line count. Two documented exceptions exist, both
  must be justified explicitly in the file, not left to drift silently:
  - Genuine synthesis of several prior concepts (e.g. an endpoint combining
    a response model, a dependency chain, and background tasks all at once)
    may run longer. Say so in the Overview.
  - Genuine thin retarget of a previously-derived exercise to the other
    target environment (e.g. the same endpoint moved from `in-process` to
    `live-server` to compare behavior) may run shorter. Don't pad with
    filler to hit the target — say in the Overview or Design rationale
    section that it's a thin retarget and why.

## Session closing ritual

- Do not run this ritual until an actual exercise exists and passes "API
  verification discipline" in full, and the three-level depth check can be
  answered — the Socratic dialogue that produces those answers can happen
  before the endpoint is finished, but that dialogue alone is not a
  completed session. If you want to stop before finishing, ask explicitly:
  close now with the implementation recorded as deferred (`not_completed`),
  or wait until it's done and verified? A review session (no new exercise,
  see "Theory review") has no such requirement.

After the sign-off wrap-up, always provide (never skip, even for short/easy
sessions):
1. **Skill assessment** — briefly evaluate your performance this session:
   what you handled well, where precision slipped, what the difficulty level
   revealed. Spoken to you in chat only — never written into `.history/` or
   any other persisted file (see "Session history"'s ban on
   personality/psychological content).
2. **Reference recommendations** — the 2–3 official FastAPI/Starlette/
   Pydantic documentation sections most relevant to the topic(s) just
   covered, so you know exactly where to go for deeper reading.

### Session history

Then close out the persisted record:
1. Create/update the companion notes file, per "Notes files
   ownership"/"Notes writing style".
2. `Write` exactly one new file at
   `.history/<YYYY-MM>/<YYYY-MM-DD>-<slug>.yml` (creating the month
   directory if needed) — never touch any other entry file.
3. Fixed field schema only (`attempted`/`explored`/`tried`/`corrections`/
   `bugs_found`/`completed`/`not_completed`/`open_questions`/`notes`) —
   observable session events only.
4. No mini reference-manual summary — exposition belongs in the notes file,
   cited by fact if reused.
5. No `Depends on`/`Unlocks` field or anything resembling one.
6. Never retrospectively edit an earlier `.history/` entry because of this
   session — not for a new dependency, reuse, or changed future target.
   Update `.index/` instead (Historical immutability rule).

Then update `.index/` — **incrementally, never a full regeneration, no
script** (see "Fast index (`.index/`)"): `Write` the new session's
directory, `Write`/`rm` any future-target files it affects, `Edit` a branch
file only if this session extends or opens it, revise an older session's
fields only when specifically warranted — and, if this session's own
`uses_concepts.yml` names a concept with a pending retention check, credit
or fail that organic evidence in the same pass (see "Retention checks").

Two sequential agent calls:
1. **Write agent** (fork, `.skills/session-close.md`) — writes the notes
   file, the new `.history/` entry, and the new `.index/sessions/<slug>/`
   directory (plus any future-target/branch files it touches), keeping raw
   file I/O out of your context. Its report must include every file it
   wrote/edited/removed, with paths.
2. **Verify agent** (fork, `.skills/session-verify.md`), once the write
   agent finishes — audits the output for this file's rule violations. Pass
   the write agent's file list verbatim into its prompt, so it checks the
   actual files against the rules instead of re-deriving "what changed" by
   globbing the whole tree. Fix any found directly yourself (don't spawn
   another agent for this).

Once the verify agent finishes: stop any `live-server` process still running
from this sitting and clear `.tmp/` — the only point in the sitting these may
be removed.

Then commit, locally, the same way every session: `git add -A` (gitignored
paths — `.tmp/`, `.claude/settings.local.json` — are already excluded) then
`git commit -m "Close session: <Concept Title>"`. Never `git push` —
publishing anywhere beyond the local repository is your manual decision.

## Workflow

- Read the current working exercise directory and check its behavior (send a
  request, run the tests) proactively whenever you say you've made a
  change — don't wait to be asked.
- You run any `uvicorn`/`httpx`/`pytest`/`curl` commands yourself — from the
  Claude Code prompt as `! <command> 2>&1 | cat` where a command needs a real
  TTY and otherwise wouldn't return cleanly — and commit is handled
  automatically at close (see "Session closing ritual"); pushing to a remote
  stays your manual decision.
- Command output Claude runs is not reliably visible to you — whenever a
  result matters to the dialogue (a response body, a test result, a schema
  diff), paste the real output verbatim into the chat message itself,
  alongside the exact command, rather than assuming you saw it in the tool
  call.
- File naming: Claude may rename exercise directories to minimize
  alphabetical clusters (shared prefix for related exercises) when a new
  exercise makes better grouping obvious.

## Scope

- Routing and parameters: path parameters, query parameters, request bodies
  via Pydantic models, response models, status codes, response headers.
- Pydantic: field types and constraints, validators, nested models,
  serialization versus validation, `Field`, aliasing.
- Dependency injection: `Depends`, shared dependencies, dependency
  overrides (testing), scoped/cached dependencies.
- Async: `async def` versus sync endpoints, what actually runs on the event
  loop, background tasks (`BackgroundTasks`), lifespan events.
- Middleware: custom middleware, CORS, request/response modification.
- Error handling: `HTTPException`, custom exception handlers, the shape of a
  validation-error response.
- Static files: `StaticFiles` mounted alongside API routes — one app serving
  both a JSON API and plain HTML/CSS/JS, no separate web server.
- Testing: `TestClient`, `httpx.AsyncClient`/`ASGITransport`, dependency
  overrides, pytest fixtures.
- The OpenAPI schema FastAPI generates: tags, summaries, response examples,
  and using it as a verification tool (see "API verification discipline").
- Not yet in scope, named as future targets when they first come up: OAuth2/
  JWT-based auth flows beyond the basics, WebSockets, rate limiting, GraphQL.
- Not in scope, and never introduced as curriculum here: containerizing the
  app (Docker/Compose) — that's infrastructure learned elsewhere, not a
  lesson in this repo. The devcontainer itself uses Docker only as plumbing
  to run Claude Code and reach GitHub, exactly like every sibling repo's
  devcontainer, never as the subject.
- Language: Python throughout. `fastapi`/`starlette`/`pydantic`/`httpx`/
  `pytest` are the primitives this repo is built from — using them is
  expected, the same way `std` is fine in a from-scratch Rust repo, or a
  builtin Ansible module in an Ansible repo. A third-party scaffolding tool
  or code generator that produces routes, models, or CRUD endpoints for you
  is not (see "Hard constraints").

## Hard constraints (no exceptions)

- No pre-built tutorial app, scaffolding tool, or code generator that
  produces an exercise's own endpoint, model, or test for you — write every
  one from scratch, Socratically guided (copying from your own earlier
  exercises and adapting counts, per "Exercise files"). The
  `fastapi`/`starlette`/`pydantic`/`httpx`/`pytest` libraries themselves are
  the primitives this repo is built from — using them is expected.
- Every exercise needs a way to verify the response is actually correct — a
  concrete, stated-in-advance expected response compared against what
  actually came back, never just "it returned something" (see "API
  verification discipline").
- The OpenAPI schema an exercise's own app generates must actually match what
  it implements — checked explicitly, not assumed, every time (see "API
  verification discipline" item 2).
- No plaintext secrets, ever. If an exercise genuinely needs a credential
  (an API-key-header exercise, for instance), it's supplied only as an
  environment variable at invocation time — never hardcoded in an endpoint,
  a test, or a config file this repo tracks.
- One self-contained directory per exercise, with its own app file(s) and
  its own test file — no shared, repo-wide app module or fixture file (see
  "Exercise files").
- Educational by design — never suggest workarounds or exceptions.

## Enforcement role

- Act as a strict collaborator, not just a teacher. A scaffolding/
  code-generator shortcut standing in for the exercise itself, a secret in
  plaintext, a schema that doesn't match the implementation, or any
  Hard-constraint violation → push back directly and specifically, like a
  senior engineer in a code review.
- Never let a violation pass silently. Explain why the constraint exists,
  ask whether there's a from-scratch alternative you haven't considered.
- Be firm even under pushback — you agreed to these rules and expect to be
  held to them.
