# Quaderno data formats

The files and folders Quaderno reads and writes.
Notes are UTF-8 Markdown, so they stay readable without Quaderno.

---

## 1. Vault layout

An example with the default settings (folder names can be changed in Settings).

```
<vault>/
├── quaderno.json              Domains and clipper rules
├── index.md                   Catalogue written by Tools → Index (do not edit by hand)
├── AGENTS.md                  Rules for agents (one more can sit in each domain)
├── Attachments/               Attachments (shared)
├── Links/
│   ├── Link inbox.md          Link inbox
│   └── Apps.md                Apps you added
├── Chronology/
│   ├── reviews.json           Chronology review data (optional)
│   └── extra.json             Extra chronology data (optional)
├── Leonardo/                  Domain "leonardo"
│   ├── Inbox.md               Task inbox
│   ├── Journal/YYYY-MM-DD.md  Daily notes
│   ├── Projects/<name>/<name>.md
│   ├── Drafts/                Ideas
│   ├── Scraps/
│   ├── Archive/
│   ├── Atlas/
│   │   ├── People/
│   │   ├── Places/
│   │   ├── Glossary/
│   │   └── Library/
│   └── Corpus/
│       ├── Paintings/
│       ├── Drawings/
│       ├── Codices/
│       └── Images/
├── Workbench/                 Domain "workbench" (same layout, without Corpus)
└── .trash/                    Trash (original paths are kept)
```

### 1.1 Skipped when reading

Folder storage (`FSStore`) does not read:

- Files and folders starting with `.` (except `.trash` at the vault root)
- `node_modules`

### 1.2 System files

These notes are kept out of the lists:

- Files named `AGENTS.md`, `CLAUDE.md`, `index.md` or `log.md`
- Notes whose front matter `type` is `links` or `launcher`

## 2. Notes (`.md`)

### 2.1 Front matter

The block between `---` lines at the top of the file is read as simple YAML.

Supported forms:

```yaml
---
key: value
quoted: "value"
list: [a, b, "c"]
block:
  - a
  - b
---
```

Nested objects and multi-line strings are not supported. All values are strings or arrays of strings.

### 2.2 Front matter keys

| Key | Where | Meaning |
|---|---|---|
| `title` | All | Title (takes precedence over the first `# Heading`) |
| `tags` | All | Tags, as an array or separated by commas or spaces. A leading `#` is ignored |
| `type` | Various | `scrap` / `work` / `person` / `place` / `term` / `book` / `links` / `launcher` |
| `status` | Various | `unverified` for scraps, `pending` for Atlas and Corpus; the stage for projects |
| `stage` | Projects | Stage (takes precedence over `status`) |
| `about` | All | Atlas or Corpus items this note is about; array of `"[[Item]]"` |
| `source` | Various | Source URL; searched by `source:` |
| `aliases` | Atlas | Other names, used to match cross references |
| `places` | People / Glossary | Related place names, used for cross references |
| `from` | In projects | `scrap` / `idea`; grouping on the project page |
| `created` / `due` | Projects | Creation and due dates (used in the sample note) |

Additional keys by kind:

| Kind | Keys |
|---|---|
| Scrap | `captured`, `kind: ai`, `session` |
| Corpus (`type: work`) | `kind` (painting / drawing / codex), `siglum`, `date`, `collection`, `inventory`, `image` |
| Person | `born`, `died`, `role`, `chronology`, `image` |
| Place | `lat`, `lon`, `stay`, `years`, `image` |
| Term | `period`, `url` |
| Book | `written`, `identified`, `recorded`, `edition` |

### 2.3 What is read from the text

| Data | How |
|---|---|
| Heading (h1) | First `# ` line after the front matter |
| Tags | `#tag` in the text (outside code and headings) plus front matter `tags` |
| Links | The name part of `[[name]]`, `[[name#heading]]`, `[[name|label]]` |
| Embeds | `![[name]]` and `![alt](path)` |
| Tasks | Lines matching `^(\s*)[-*+]\s\[([ xX/-])\]\s?(.*)$` |
| Snippet | First three lines that are not headings, tables, code or quotes (up to 220 characters) |

Nothing inside fenced code blocks is read for any of these.

### 2.4 Task lines

```
- [ ] text 📅 2026-11-10 ⏫
  │     │      │          └ priority 🔺⏫🔼🔽
  │     │      └ due date (due: 2026-11-10 also works)
  │     └ text
  └ state: space = open, x/X = done, - = cancelled, / = other
```

The completion date is `✅ YYYY-MM-DD`. Dates after `⏳` `🛫` `➕` are skipped.

## 3. Structured notes

### 3.1 Link inbox (`Links/Link inbox.md`)

Each `##` heading is one entry; `- key: value` lines below it are read. Entries without `url` are ignored.

```markdown
## Codex Atlanticus
- url: https://codex-atlanticus.ambrosiana.it/
- site: Biblioteca Ambrosiana
- category: Manuscripts
- note: Folio viewer
- lang: it
- title_from: page
- added: 2026-11-02
- status: pending
```

`category` is one of Manuscripts / Collections / Reference works / Texts and libraries / Other.

### 3.2 Apps note (`Links/Apps.md`)

```markdown
## Map view
- url: https://www.google.com/maps/@{lat},{lon},15z
- group: Maps
- note: Opens the chosen place
```

Only `url` values starting with `http(s)://` are used. `{lat}` `{lon}` `{place}` are replaced with the Sky place.

## 4. `quaderno.json`

Settings file at the vault root, created with "Save quaderno.json" in Settings.

```json
{
  "format": "quaderno-vault",
  "version": 1,
  "domains": [
    { "id": "leonardo",  "dir": "Leonardo",  "label": "Leonardo",  "color": "#9B111E" },
    { "id": "workbench", "dir": "Workbench", "label": "Workbench", "color": "#3B2F4C" }
  ],
  "clip": {
    "default": "leonardo",
    "folder": "Scraps",
    "rules": [
      { "host": "github.com",           "domain": "workbench" },
      { "host": "stackoverflow.com",    "domain": "workbench" },
      { "host": "developer.chrome.com", "domain": "workbench" },
      { "host": "docs.claude.com",      "domain": "workbench" }
    ]
  }
}
```

| Key | Meaning |
|---|---|
| `domains[].id` | Identifier (required) |
| `domains[].dir` | Folder in the vault (required; leading and trailing `/` removed) |
| `domains[].label` | Display name (defaults to `dir`) |
| `domains[].color` | Colour (defaults to `#867c78`) |
| `clip` | Routing rules for the web clipper; the URL's host name picks the domain |

Domains without `id` and `dir` are ignored. If the file cannot be read, the default domains are used and a notice is shown.
When running as an extension, the domain list is also written to `chrome.storage.local` (shared with the clipper).

## 5. Chronology data (`Chronology/`)

| File | Schema | Content |
|---|---|---|
| `reviews.json` | `ldv-chronology-reviews/1` | Review status of chronology entries (`status`, `flags` and so on) |
| `extra.json` | — | Additional entries |

Each file is taken from the vault if present, otherwise from the snapshot built into the app. Reference → Chronology can save the built-in data to the vault.

## 6. Stored in the browser

Separately from the vault, the browser keeps the following. Moving the vault to another device does not move these.

### 6.1 localStorage

| Key | Content |
|---|---|
| `quaderno:settings` | Settings (`CFG`), including pins, open folders, last note and current view |
| `quaderno:hist` | Date, calendar and weather of the historical scene |
| `quaderno:wx` | Weather cache (per place, 15 minutes) |
| `quaderno:calc` | Calculator log (up to 500 entries) |

Pins are kept per vault, keyed by `"<kind>:<name>"`.

### 6.2 IndexedDB (database `quaderno`, version 1)

| Store | Key | Content |
|---|---|---|
| `files` | `path` | Browser library files: `{ path, data, mtime, ctime, size }`; folders are `{ path: "a/b/", dir: true }` |
| `meta` | Any | Previous folder handle (`dir`) and similar |

## 7. .zip export

Settings → Export .zip saves every vault file in a ZIP archive (stored, no compression, with CRC32). No external library is used.
