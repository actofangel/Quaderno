# Quaderno

**Quaderno** is a local Markdown research notebook, built for research on Leonardo da Vinci.
Every note is a plain `.md` file, so the same folder also opens in Obsidian or any text editor.
Behind the page, a sky follows the sun, moon and weather at a chosen place (Florence by default), and it can switch to a historical scene, such as an evening over Milan in 1495.

---

## Features

| Area | What it does |
|---|---|
| Notes | Edit and read modes, `[[wiki links]]`, `#tags` (nested), pasted and dropped images, Obsidian-style callouts, `==highlight==` |
| Files | File name follows the first `# Heading`, links are rewritten on rename, archive, trash (`.trash/`), .zip export |
| Tasks | Every `- [ ]` line in any note is gathered under Tasks. Obsidian Tasks syntax for 📅 due dates, ⏫ priorities and ✅ completion dates. Quick add (`due:tomorrow !!`) |
| Projects | Folders under `Projects/` become projects on an Idea / Active / On hold / Done board |
| Capture | Ideas (quick drafts), Scraps (unverified clippings and AI sessions), a link inbox and a link maker that writes citations |
| Atlas | People, Places, Glossary and Library notes with templates and cross references |
| Corpus | Catalogue of Leonardo's paintings, drawings and codices, with summaries and images from Wikipedia |
| Reference | Embedded Leonardo chronology, places, works and a list of research sites |
| Sky | Sun and moon position and phase, current weather from Open-Meteo, a historical mode (Julian calendar, local mean time), Italian hours, Florentine and Venetian year styles |
| Tools | Calculator with a log, sky stamp, `index.md` generator, launcher for research apps |
| Domains | Split one vault into areas such as Leonardo and Workbench (`quaderno.json`) |

## Getting started

### Single-file build (simplest)

1. Download [`dist/Quaderno.html`](dist/Quaderno.html).
2. Open it in Chrome, Edge or another Chromium browser.
3. Choose where to keep the notes.

| Option | What happens | Browsers |
|---|---|---|
| **Open a folder on this computer** | Reads and writes `.md` files in the folder directly. An Obsidian vault works as is | Chromium (File System Access API) |
| **Open a vault folder in this browser** | Copies the folder into the browser. Bring changes back with a .zip export | Browsers without that API |
| **Use the browser library** | Stores everything in the browser (IndexedDB). Works offline | All |

If the chosen folder holds both `index.html` and `vault/`, Quaderno opens `vault/` as the vault.

### Split build (`src/`)

`src/app.html`, `app.js` and `app.css` are the same app as separate files, laid out for a Chrome extension. When `location.protocol` is `chrome-extension:`, the chronology is shown through `chrono.html`.
These files, which the split build refers to, are not yet in this repository:

- Icons such as `icons/32.png`
- `chrono.html` (the page that shows the chronology inside the extension)
- `manifest.json`

## Keyboard shortcuts

| Keys | Action |
|---|---|
| Ctrl/Cmd + N | New note |
| Ctrl/Cmd + Shift + I | New idea in Ideas |
| Ctrl/Cmd + K (or O) | Jump to note |
| Ctrl/Cmd + Shift + D | Today's note |
| Ctrl/Cmd + E | Read or edit |
| Ctrl/Cmd + S | Save now |
| Ctrl/Cmd + Enter | Toggle the task on the line |
| Ctrl/Cmd + B / I | Bold / italic |
| Ctrl/Cmd + Shift + H | Highlight |
| Ctrl/Cmd + \\ | Focus mode |
| Ctrl/Cmd + Shift + F | Search |
| `[[` | Link a note (autocomplete) |
| Option/Alt + click | Place the caret in a link instead of opening it |

## Network use

Quaderno runs locally. It goes online only in these cases:

| Host | Purpose | How to avoid it |
|---|---|---|
| `api.open-meteo.com` | Current weather for the sky (cached for 15 minutes) | Use the historical scene |
| `geocoding-api.open-meteo.com` | Finding a sky place by name | Do not search |
| `api.microlink.io` | Page titles when adding links | Settings → Page titles → "Do not go online" |
| `*.wikipedia.org` | Summaries and images for Corpus and Atlas | Do not use those buttons |

## Repository layout

```
.
├── README.md            This file
├── dist/
│   └── Quaderno.html    Single-file build (HTML, CSS and JS inlined)
├── src/
│   ├── app.html         Split build HTML (extension)
│   ├── app.css          Styles (minified)
│   └── app.js           Application (minified)
└── docs/
    ├── SPEC.md          Functional specification
    ├── DATA-FORMAT.md   Vault, file and front matter formats
    └── ARCHITECTURE.md  Code structure
```

The `<script>` and `<style>` in `dist/Quaderno.html` are identical to `src/app.js` and `src/app.css`.

## Documentation

- [Functional specification (docs/SPEC.md)](docs/SPEC.md)
- [Data formats (docs/DATA-FORMAT.md)](docs/DATA-FORMAT.md)
- [Architecture (docs/ARCHITECTURE.md)](docs/ARCHITECTURE.md)

## Current limitations

- `src/app.js` and `src/app.css` are minified; the original source is not yet in the repository.
- There is no build step yet for producing the single-file build from the split build.
- No license has been chosen.
