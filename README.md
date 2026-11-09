# Ajaia Docs

A lightweight collaborative document editor: create and format documents, import files, and share them with other people as editors or viewers.

**Live app:** `<LIVE_URL>`  
**Walkthrough video:** see [`VIDEO_URL.txt`](VIDEO_URL.txt)

## Demo accounts

Sign-in is simulated: pick an account on the sign-in page (no passwords). Switch accounts from the avatar menu (top right) → **Switch demo account**.

| Account | Email | Starts with |
| --- | --- | --- |
| Alice Chen | `alice@ajaia.test` | Owns *Welcome to Ajaia Docs* (shared with Bob as editor, Carol as viewer). Can view Bob's *Q4 planning notes*. |
| Bob Rivera | `bob@ajaia.test` | Owns *Q4 planning notes*. Can edit *Welcome to Ajaia Docs*. |
| Carol Okafor | `carol@ajaia.test` | Can view *Welcome to Ajaia Docs* (read-only). |

### A 2-minute review path

1. Sign in as **Alice** → open *Welcome to Ajaia Docs* → edit text, try **bold / italic / underline**, headings and lists. Watch the "Saving… → All changes saved" indicator. Reload: everything is still there.
2. Rename the document by editing its title (top left).
3. Go back → **Import a file** → pick one of the files in [`samples/`](samples) (`.md`, `.txt`, `.docx`). It opens as a new editable document.
4. Click **Share** → add `carol@ajaia.test` as **Can view** (or `bob@ajaia.test` as **Can edit**).
5. Switch to **Carol** → the document appears under **Shared with me** with a *Can view* badge; the editor is read-only.
6. Switch to **Bob** → open *Welcome to Ajaia Docs* → he can edit but can't change sharing or delete.

## Features

- **Documents:** create, rename, edit, autosave (debounced, plus Ctrl/⌘+S), reopen, delete (owner only), export as Markdown or HTML.
- **Rich text** (TipTap/ProseMirror): bold, italic, underline, strikethrough, headings 1–3, bulleted and numbered lists, quotes, code blocks, links, undo/redo.
- **File import:** `.txt`, `.md`/`.markdown` and `.docx` up to **2 MB** become a new document. Headings, lists, bold/italic/underline and links are kept. Images, tables, colors and fonts are not imported. Other file types are rejected with a clear message.
- **Sharing:** every document has one owner, who can grant **Can edit** or **Can view** access by email, change roles, or revoke access. The home page separates **My documents** from **Shared with me** and shows each document's owner and your access level.
- **Safe concurrent edits:** saves carry the document version. If someone else saved since you opened the document, your save is rejected (HTTP 409) and you choose **Load latest version** or **Keep my version**. Nobody's work is overwritten without them knowing.
- **Persistence:** SQLite (libSQL). A local file in development, hosted [Turso](https://turso.tech) (free tier) in production.

## Run locally

Requires **Node.js 20.9+** (developed on Node 22).

```bash
npm install
npm run dev
```

Open http://localhost:3000. No environment variables are needed locally: the app creates `./local.db` (SQLite) on first request, seeds the demo accounts and documents, and uses a development-only session secret. To reset, stop the server and delete `local.db`.

### Tests and checks

```bash
npm test            # Vitest: services, access control, concurrency, import, sessions (real SQLite)
npm run typecheck   # generate Next route types + tsc
npm run lint
npm run build
```

## Deployment (Vercel + Turso)

The live app runs on Vercel's free tier with a free Turso database. No paid services are required.

1. **Database.** Create a database at [turso.tech](https://turso.tech) (or with the CLI: `turso db create ajaia-docs`). Copy its URL (`libsql://…`) and create an auth token (`turso db tokens create ajaia-docs`).
2. **App.** Import the repository in Vercel (framework preset: Next.js) and set these environment variables:

   | Variable | Value |
   | --- | --- |
   | `DATABASE_URL` | `libsql://<db>-<org>.turso.io` |
   | `DATABASE_AUTH_TOKEN` | Turso token |
   | `SESSION_SECRET` | a long random string, e.g. `openssl rand -base64 32` |

3. Deploy. The schema and demo data are created automatically on the first request (idempotent and safe to run again).

In production the app refuses to start without `DATABASE_URL` and `SESSION_SECRET`, so a misconfigured deploy fails loudly instead of quietly using an insecure default. `TURSO_DATABASE_URL` / `TURSO_AUTH_TOKEN` are also accepted.

## Project structure

```
src/
  app/
    page.tsx                  Home: My documents / Shared with me, new + import
    login/                    Demo account picker
    docs/[id]/                Editor page (server loads doc + role, client edits)
    api/
      session/                POST sign in, DELETE sign out
      documents/              GET list, POST create
      documents/import/       POST multipart file → new document
      documents/[id]/         GET, PATCH (versioned save), DELETE
      documents/[id]/shares/  GET people with access, POST grant/change role
      documents/[id]/shares/[userId]/  DELETE revoke
  components/
    editor/                   DocumentEditor (save loop, conflicts), Toolbar, ShareDialog, ExportMenu
  lib/
    access.ts                 Roles and capabilities (owner / editor / viewer)
    documents.ts              Document + sharing service; all access checks live here
    db.ts                     libSQL client, schema, seed data
    session.ts                HMAC-signed session cookie
    importers.ts              .txt / .md / .docx → sanitized HTML
    sanitize.ts               HTML allowlist applied to every write
    api.ts, errors.ts         Typed errors → consistent JSON error responses
tests/                        Vitest suites (run against a temporary SQLite file)
samples/                      Files for trying the importer
```

## More

- [ARCHITECTURE.md](ARCHITECTURE.md): what I prioritized, the trade-offs, and what's next.
- [AI_WORKFLOW.md](AI_WORKFLOW.md): how AI tools were used and checked.
- [SUBMISSION.md](SUBMISSION.md): what's included in this submission.
