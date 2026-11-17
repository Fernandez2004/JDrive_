# JDrive — Personal Cloud (Enhanced)

A polished, local-first personal cloud demo. Everything lives in your browser via
IndexedDB — no server, no account, no uploads leave your device.

This is an enhanced, fully interactive rebuild of the original single-file
`index.html.html` concept attached in `C:\Users\REC\Downloads`.

## Run it

The app is plain HTML/CSS/JS with no build step.

**Recommended** — serve the folder locally:

```powershell
cd "C:\Users\REC\Documents\Default Project\jdrive"
python -m http.server 4173 --bind 127.0.0.1
```

Then open <http://127.0.0.1:4173/>.

You can also double-click `index.html`, but a local server is more reliable for
browser storage behavior and is **required** to load the shared library. An
internet connection is needed for the CDN-hosted Tailwind, Lucide icons, JSZip
and Inter font.

## Host on GitHub Pages

This is a static site, so it runs on GitHub Pages with no backend. The `.nojekyll`
file is already included, and all asset paths are relative, so a project site at
`https://<username>.github.io/<repo>/` works.

1. Create a new **public** repo on GitHub (e.g. `jdrive`).
2. On the empty repo page, choose **uploading an existing file**.
3. Drag in the **contents** of this `jdrive` folder: `index.html`, `README.md`,
   `.nojekyll`, the `app` folder, and the `shared` folder. Commit the changes.
4. In the repo, go to **Settings → Pages**.
5. Under **Build and deployment**, set Source to **Deploy from a branch**,
   branch **main**, folder **/ (root)**, then **Save**.
6. Wait about a minute, then open `https://<username>.github.io/jdrive/`.

Notes:
- GitHub Pages sites are **public**, even when the repo is private (private Pages
  requires GitHub Enterprise Cloud). Anyone with the link can open the app.
- Files are stored per browser and per site origin, so data from your local
  `127.0.0.1` copy won't appear on the hosted version — it seeds fresh demo data.
  A visitor gets their own separate vault and cannot see your files.
- Share links remain illustrative; there is no server behind them.
- If you use Git instead, `git init` inside this folder, commit, and push to the
  repo's `main` branch.

## Shared library (password-protected downloads)

The shared library is the one part of the app that is **shared between visitors**.
It is a folder of encrypted files committed to the repository. Visitors can see
the file list, but must enter the library password to decrypt and download.

### Publish or replace the library (owner only)

Only someone with write access to the repository can change the live library,
because changing it means committing new files. The publish tool merely creates a
ZIP on your computer; it does not upload anything by itself.

1. Open the site and sign in to your local vault.
2. Go to **Settings → Shared library → Publish shared library** (or use the
   **Publish library** button on the Shared Library page).
3. Tick every item that should be in the library, choose a **library password**
   (separate from your master password), and confirm it.
4. Click **Encrypt & download .zip**. The app encrypts each file with
   AES-256-GCM and downloads `jdrive-shared-library.zip`.
5. **Delete the existing `shared` folder** in the project so old encrypted files
   are removed.
6. Extract the ZIP into the project root (the folder containing `index.html`).
   It creates `shared/manifest.json` and `shared/files/*.bin`.
7. Commit and push. GitHub Pages serves the new library within about a minute.

Refreshing the site reloads the manifest. Visitors are locked out until they
enter the new password; the old password stops working immediately.

### How encryption works

- Key derivation: **PBKDF2-HMAC-SHA256**, 600,000 iterations, random 16-byte salt
  per publish.
- Encryption: **AES-256-GCM** with a random 12-byte IV per file.
- Password check: an encrypted verifier value (`JDRIVE-SHARED-LIBRARY-V1`); a
  wrong password fails GCM authentication and is rejected.
- The password is never stored in the repository or in browser storage. It is
  only held in memory after unlocking, and **Lock** or a page reload clears it.
- Folders are zipped in the browser, then encrypted like any other file. A
  folder appears in the library as `<folder>.zip`.
- File names, sizes, MIME types and publish date are visible in
  `shared/manifest.json`; file **contents** are encrypted.

### Security caveats

- This protects files **in transit and at rest on GitHub**, not against someone
  who knows the password. Anyone with the password can download and keep copies
  of the decrypted files, so use a strong, unique password and share it only
  with the intended people.
- The site itself is public source code. A visitor can generate their own local
  package with the same tool, but cannot replace the hosted library without
  repository access.
- There is **no password recovery**. If you forget the library password, publish
  a new package with a new password.
- GitHub Pages is not built for large file hosting. Keep each package well under
  GitHub's 100 MB per-file limit (the app enforces 100 MB per package) and note
  the Pages soft limits of about 1 GB per site and 100 GB bandwidth per month.
  The **browser uploader** additionally limits individual files to 25 MB; for
  larger packages, push with Git or GitHub Desktop instead.
- Use HTTPS (GitHub Pages or localhost). The Web Crypto API is unavailable on
  plain `http://` origins other than localhost, and `file://` cannot fetch the
  manifest.

## Features

**Files & folders**
- Upload multiple files or an entire folder (picker or drag & drop)
- Files stored as Blobs in IndexedDB and persist across reloads
- Re-upload the same filename to create a restorable version (last 6 kept)
- Download files; folders export as `.zip` (optional compression)
- Rename, favorite, and manage item details with inline previews
  (images and text/markdown/code)

**Sharing & history**
- Encrypted shared library with a password gate, per-session unlock and Lock
- Local demo share links with copy and revoke
- Version History timeline with restore/download for previous versions
- Recycle bin with restore, permanent delete, and empty bin
- Download history with re-download and clear

**Workspace**
- Dashboard with greeting, storage breakdown by category, recent files
  and activity feed
- Recent, Favorites, Shared Library and My Share Links views
- Search palette (`Ctrl/Cmd + K` or `/`) plus per-list search and sorting
- Notifications with unread badges and mark-all-read
- Dark / light / system theme

**Account & security**
- Editable profile (name, email, avatar initials)
- Master password hashed with SHA-256 + random salt and stored locally
- Password strength meter and validation
- Two-step lock (shows lock screen after the tab is inactive)
- Settings: email alerts (demo), delete confirmation, compression preference,
  default file layout, reset demo data

## Keyboard shortcuts

| Keys | Action |
| --- | --- |
| `Ctrl/Cmd + K` or `/` | Open search palette |
| `Enter` in palette | Filter My Files by the query |
| `Esc` | Close modal, search, popovers or mobile sidebar |

## Data & privacy

- Files: IndexedDB database `jdrive_db`, object store `items`
- Preferences/history: `localStorage` keys under `jdrive.*.v2`
- Clearing site data or using **Settings → Reset demo data** restores the
  original seeded demo vault
- Demo files are seeded on first launch: welcome note, roadmap with an older
  version, CSV, SVG image and a small folder
- The shared library password is never persisted; it exists only in memory while
  the page is unlocked

## Project structure

```
jdrive/
├─ index.html            App shell, modules, modals, overlays
├─ README.md
├─ .nojekyll             Disables Jekyll on GitHub Pages
├─ shared/
│  ├─ manifest.json      Encrypted library manifest (empty placeholder by default)
│  └─ files/*.bin        Encrypted file contents (created when you publish)
└─ app/
   ├─ core.js            Helpers, storage, seed data, theme, notifications, auth
   ├─ views.js           Chrome, dashboard, file list/grid rendering
   ├─ views-modules.js   Share links, downloads, versions, recycle, search, settings
   ├─ actions.js         Navigation, modals, item actions, downloads, shortcuts
   ├─ upload.js          Upload queue, drag & drop, folders, versioning
   ├─ shared.js          Shared library manifest, AES-GCM publish/decrypt, UI
   └─ settings.js        Profile, security, toggles, init, auto-lock
```

## Notes

- This is a front-end demo. "Share links" are illustrative and not published.
- Master passwords are hashed locally for demonstration; this is not production
  authentication or encryption at rest. Only the shared library is encrypted.
- Tested in Chromium. Lighthouse: Accessibility 100, Best Practices 100, SEO 100.
