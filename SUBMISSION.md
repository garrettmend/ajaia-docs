# Submission: Ajaia Docs

## Links

- **Live app:** `<LIVE_URL>`
- **Walkthrough video:** `<VIDEO_URL>` (also in `VIDEO_URL.txt`)
- **Source:** this folder (also at https://github.com/garrettmend/ajaia-docs, a private repo; access on request)

## Test accounts

No passwords: choose an account on the sign-in page, and switch from the avatar menu.

- `alice@ajaia.test`: owns *Welcome to Ajaia Docs* (shared with Bob as editor and Carol as viewer)
- `bob@ajaia.test`: owns *Q4 planning notes* (shared with Alice as viewer)
- `carol@ajaia.test`: view-only access to *Welcome to Ajaia Docs*

Sample files for the import feature are in `samples/`.

## What's included

| Item | Location |
| --- | --- |
| Source code | `src/` (app, API routes, components, services), `tests/`, config files |
| Setup and run instructions | `README.md` |
| Architecture note | `ARCHITECTURE.md` |
| AI workflow note | `AI_WORKFLOW.md` |
| This list | `SUBMISSION.md` |
| Walkthrough video link | `VIDEO_URL.txt` |
| Automated tests | `tests/` (run with `npm test`) |
| Import samples | `samples/meeting-notes.md`, `samples/plain-notes.txt`, `samples/project-brief.docx` |
| Example environment file | `.env.example` |

`node_modules/`, `.next/` and local database files are excluded. Run `npm install` to restore dependencies.

## Status

### Working end to end

- Create, rename, edit, autosave, reopen and delete documents
- Rich text: bold, italic, underline, strikethrough, headings 1–3, bulleted and numbered lists, quotes, code blocks, links, undo/redo
- Import `.txt`, `.md` and `.docx` (≤ 2 MB) as new documents; other types are rejected with a clear message
- Sharing by email with **Can edit** / **Can view** roles, role changes and revocation; owner-only share and delete
- Separate **My documents** and **Shared with me** lists, with owner and access badges
- Server-enforced permissions (viewers get a read-only editor *and* 403s from the API; users without access get 404)
- Conflict detection on concurrent saves, with a choice to load the latest version or keep your own
- Persistence in SQLite (libSQL) locally and Turso in production
- Export to Markdown and HTML
- 15 automated tests

### Intentionally out of scope

- Real-time co-editing and presence (replaced by conflict detection; see `ARCHITECTURE.md`)
- Real authentication (simulated with seeded accounts and a signed session cookie)
- Comments, version history, images and tables in documents

### Next 2–4 hours

1. Version history with restore
2. A live "someone else is editing" indicator (poll or SSE on the document version)
3. Comments anchored to text
4. Playwright end-to-end tests for sharing and conflict flows
