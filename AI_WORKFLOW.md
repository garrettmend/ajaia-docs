# AI workflow note

## Tools used

- **Claude Code (Claude Opus 5.5)** in the Claude desktop app: planning, implementation, tests and documentation drafts.
- Claude Code's **built-in browser** for driving the running app: typing with real keyboard shortcuts, switching accounts, checking API responses from the page.
- Standard tooling as the source of truth: `tsc`, ESLint (with React Compiler rules), Vitest, `next build`, and `curl` against the production server.

## Where AI materially sped things up

- **Scaffolding and boilerplate:** route handlers, typed error mapping, and the zod schemas took minutes instead of an hour.
- **The service layer and its tests together.** Writing access rules and the tests that pin them down in the same pass made it cheap to cover many edge cases: 404 vs 403, self-share, unknown email, role changes, orphaned share rows.
- **Editor UI.** TipTap toolbar wiring, the share dialog and the export menu are mostly mechanical code.
- **Verification.** Scripted browser checks (type, format, reload, switch user, attempt forbidden writes) made it fast to re-test the whole flow after each change.

## What I changed or rejected from AI output

| AI-generated or default output | What I did instead | Why |
| --- | --- | --- |
| Named SQL parameters (`:userId`) | Switched to numbered positional parameters (`?1`) | The local and hosted libSQL drivers resolve named-parameter prefixes differently. Positional parameters behave the same in both, which removes a bug that would only appear after deploying. |
| Next 16 scaffold default `cacheComponents: true` | Turned it off | Every page is per-user. That mode would require Suspense boundaries around every session read, for no benefit here. |
| Save-loop `useCallback` that scheduled itself; `setState` inside effects | Rewrote with a ref-held callback and `useSyncExternalStore` | Flagged by the React Compiler lint rules. The originals worked, but they risked stale closures and extra renders. |
| Sign-in page that read the database before reading cookies | Marked it `force-dynamic` | `next build` tried to prerender it and crashed without a DB URL. Only running a real production build caught this. |
| `.docx` import that silently dropped underline | Added a mammoth style map and a test | Found by importing the sample file and checking the output, not by the first round of tests. |
| Conflict banner saying "Load their version" | "Load latest version" / "Keep my version" | The other save can come from *you* in another tab. Found while testing that exact case. |
| Imported docs titled after the filename | Use the document's first `# Heading` when present | `meeting-notes.md` → "Weekly sync — notes" is what a user expects. |

Approaches I rejected outright:

- **Real-time co-editing (Yjs/CRDT):** too large for the timebox and needs a stateful server. Replaced with versioned saves and explicit conflict resolution.
- **SQLite on the container's own disk:** container disks are ephemeral, so documents would vanish on every redeploy. The database lives on a persistent Railway volume instead, and production refuses to start without one (or a `DATABASE_URL`).
- **Storing ProseMirror JSON:** stricter, but import conversion would need a server-side DOM. Sanitized HTML matching the editor's schema was the pragmatic choice.

## How I verified correctness, UX and reliability

- **Automated:** 17 Vitest tests against a real temporary SQLite database cover create/save/reopen with formatting, validation, HTML sanitization, the full permission matrix, 404-for-no-access, role changes and revocation, delete cascade, optimistic-concurrency conflicts, `.md`/`.txt`/`.docx` import plus rejected files, session tamper detection, and database-location resolution.
- **End-to-end in a browser:** created a document and formatted it with keyboard shortcuts and the toolbar, renamed it, reloaded to confirm persistence, shared it as viewer, switched users to confirm the read-only editor *and* a 403 from the API, forced a conflict and resolved it, and imported a Markdown file through the real upload input.
- **Production mode:** ran `next build` + `next start` and checked with `curl`: unauthenticated 401, redirect to sign-in, forged cookie rejected, `Secure`/`HttpOnly` cookie flags, malformed JSON → 400, unsupported upload → 415.
- **Static checks:** `tsc` and ESLint clean.

<!-- Candidate: add 2–3 sentences in your own words here, e.g. which decisions you reviewed most closely and anything you would do differently. -->
