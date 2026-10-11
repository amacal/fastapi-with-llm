# in-process exercises

Shared infrastructure note, not an exercise itself. Every `in-process/<dir>/`
exercise calls its own app object directly through Starlette's test
machinery — `fastapi.testclient.TestClient` (sync-looking, wraps `httpx`) or
`httpx.AsyncClient` with `httpx.ASGITransport(app=app)` (for genuinely async
test code) — never over a real socket, never via a running `uvicorn` process.

This is fast and sufficient for most exercises: routing, path/query
parameters, request/response validation, dependency resolution, error
handling. It is not sufficient, and never used as a stand-in, for anything
CLAUDE.md's "Target environments" names as `live-server`'s job: real
concurrency under actual I/O, lifespan events firing against a real event
loop, a background task's timing after the response is already sent, or
static-file serving's real HTTP semantics. An exercise that needs any of
those lives in `live-server/` instead, decided by the plan agent at
session-planning time (see CLAUDE.md's "Session planning").

Everything inside an `in-process/<dir>/` exercise directory — the app file,
the models, the test — is written by you, guided Socratically, from scratch
or adapted from one of your own earlier exercises (see CLAUDE.md's "Exercise
files"). Claude never writes or completes any of it (see CLAUDE.md's "Hard
constraints"). Running the test file and reading its actual output is
Claude's job when verification infrastructure is needed, per CLAUDE.md's
"Workflow" note on pasting real output.
