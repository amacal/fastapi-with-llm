# live-server exercises

Shared infrastructure note, not an exercise itself. Every `live-server/<dir>/`
exercise is verified against a real `uvicorn` process bound to a real local
port, reached with real HTTP requests — never `TestClient`/`ASGITransport`,
which would quietly skip exactly what this target exists to exercise (see
CLAUDE.md's "Target environments").

Starting and stopping that process is Claude's job, not yours — run from the
repo root as a background job for the sitting, never writing a tracked file
to do it:

```bash
uvicorn main:app --app-dir live-server/<dir> --host 127.0.0.1 --port <port> --no-reload &
```

Pick a port fresh each sitting rather than assuming one is free, confirm the
process is actually up before sending any verification request (a quick
request to the app's own root or an existing route, not a dedicated
health-check endpoint invented just for this), and kill it explicitly at
session close (see CLAUDE.md's "Session closing ritual") — it never survives
past the sitting that started it. If it dies mid-session, say so and restart
it on the next check rather than silently working around the gap.

Verification requests themselves go over real HTTP — `curl` or `httpx`
against `http://127.0.0.1:<port>/...` — and every command and its real
output is pasted into chat (see CLAUDE.md's "Workflow"), since Claude-run
command output isn't otherwise reliably visible to you. What the response
should be, and interpreting whether it's right, stays Socratic and yours
(see CLAUDE.md's "API verification discipline").

Everything inside a `live-server/<dir>/` exercise directory — the app file,
the models, any static files being served, the test — is written by you,
guided Socratically, from scratch or adapted from one of your own earlier
exercises (see CLAUDE.md's "Exercise files"). Claude never writes or
completes any of it (see CLAUDE.md's "Hard constraints").
