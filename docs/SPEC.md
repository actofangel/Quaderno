# Quaderno functional specification

This document describes the behaviour of the current build (`src/app.js` / `dist/Quaderno.html`).

- Storage formats: [DATA-FORMAT.md](DATA-FORMAT.md)
- Code structure: [ARCHITECTURE.md](ARCHITECTURE.md)

---

## 1. Overview

| Item | Description |
|---|---|
| Type | Single-page app that runs entirely in the browser; no server |
| Purpose | Keep research notes, sources and tasks on Leonardo da Vinci as Markdown |
| Storage | A local folder (File System Access API) or the browser (IndexedDB) |
| Compatibility | Notes are plain Markdown, following Obsidian conventions (wiki links, embeds, callouts, Tasks emoji) |
| Dependencies | None (plain JavaScript) |

## 2. Layout

```
┌──────────┬─────────────┬──────────────────────────┬────────────┐
│ #side    │ #list       │ #main                    │ #drawer    │
│ sidebar  │ note list   │ note / view / blank      │ drawer     │
└──────────┴─────────────┴──────────────────────────┴────────────┘
   Background: #sky (sky canvas), #fx (rain, snow and other effects)
```

### 2.1 Sidebar (`#side`)

From top to bottom:

1. **Logo**: click to switch light and dark.
2. **Domain switcher** (`#doms`): switches between the areas defined in `quaderno.json`. With none selected, everything (All) is shown.
3. **Notes outside the domains** (`#outside`): appears only when some notes belong to no domain; offers to move them into one.
4. **Navigation** (`#nav`)

   | Item | Content |
   |---|---|
   | All Notes | Ordinary notes (excludes Archive, Scraps, Ideas, Atlas, Corpus and system files) |
   | Today | Opens today's daily note, creating it if needed |
   | Pinned | Pinned notes |
   | Tasks | Tasks from all notes |
   | Projects | Project board; up to 12 projects that are not done appear as sub-items |
   | **Atlas** heading | Corpus / Places / People / Library / Glossary |
   | Ideas | Drafts (`Drafts/`) |
   | Scraps | Unverified clippings (`Scraps/`) |
   | Attachments | Attachment list |
   | Tools | Tools |
   | Archive | Archived notes |
   | Reference | Chronology, places, works and links |
   | Trash | Trash |

5. **Folders**: folder tree. Open and closed state is remembered. Drag notes onto folders to move them.
6. **Tags**: tag tree (nested `#a/b` tags shown as a tree).
7. **Capture**: Note / Idea / Scrap (from the clipboard) buttons, shown as icons or words.
8. **Footer**: small sky disc (`#mini`), sky summary, Settings button.

### 2.2 Note list (`#list`)

- Title, new note button, search box, filter chips, count, sort (Modified / Created / Title).
- Pinned notes always come first.
- Views that do not use the list (Tasks, Projects and others) hide it (`nolist`).

#### Search syntax

Space-separated terms must all match (AND). `"..."` matches a phrase, a leading `-` negates.

| Term | Matches notes where |
|---|---|
| `word` | Title or text contains it |
| `"a phrase"` | Title or text contains the phrase |
| `#tag` / `tag:tag` | The note has the tag or a subtag of it |
| `source:word` | Front matter `source` contains it |
| `path:word` | The file path contains it |
| `task:word` | An open task contains it |
| `-term` | Negation of any of the above |

### 2.3 Note pane (`#p-note`)

Header buttons:

| Button | Action |
|---|---|
| Read / Edit | Switch modes (Ctrl/Cmd+E) |
| Sky stamp | Insert the current sky line at the caret |
| Image | Choose and insert images |
| Project | Add to a project |
| Undo / Redo | Undo and redo |
| New note about / Notes about | Create and list notes about an Atlas or Corpus item (`about` property) |
| Promote | Move an idea into the working notes |
| Pin / Archive / More | Pin, archive, more actions |
| Focus | Focus mode (Ctrl/Cmd+\\) |
| Drawer | Open or close the drawer |

Body:

- **Edit mode**: a `textarea` overlaid on a syntax-coloured "mirror"; a custom editor.
- **Read mode**: Markdown rendered as HTML. Each block carries its source line (`data-line`).
- **Peek** (`#peek`): preview of a linked note on hover.
- **Source bar** (`#srcbar`) and **image gallery** (`#igal`): shown for Corpus and Atlas items.
- **Footer** (`#nfoot`): word count and other details.

### 2.4 Drawer (`#drawer`)

| Tab | Content |
|---|---|
| Sky | Sky details (§9) |
| Outline | Heading outline |
| Links | Backlinks and outgoing links |
| Tools | Link maker, calculator, save AI session, sky stamp |
| Apps | Launcher for research apps |

At 1180px wide or less the drawer overlays the page; wider, it sits beside it.

## 3. Library (storage)

If no storage has been chosen, a Welcome dialog appears at startup.

| Option | Implementation | Notes |
|---|---|---|
| Reopen “folder name” | Requests read/write permission again for the previous folder handle (kept in IndexedDB) | Only after a folder was used |
| Open a folder on this computer | `showDirectoryPicker()` → `FSStore` | Chromium only |
| Open a vault folder in this browser | Imported into `IDBStore` | For browsers without that API |
| Use the browser library | `IDBStore` | All browsers |

- If the chosen folder holds `index.html` and `vault/`, `vault/` is opened.
- If the browser library is empty and has not been seeded, sample notes are created (Welcome, Inbox, Sample project).
- With folder storage, the vault is rescanned when the window regains focus (at most every 4 seconds) to pick up changes made in other editors. A note with unsaved edits is not overwritten.
- Settings → Library switches storage, exports a .zip, imports files or a folder, and copies the browser library into a folder.

## 4. Notes

### 4.1 Creating notes

| Method | Location | Initial content |
|---|---|---|
| New note (Ctrl/Cmd+N) | The folder being viewed, otherwise the current domain root | `# ` plus the tag if a tag is being viewed |
| Today (Ctrl/Cmd+Shift+D) | `<domain>/Journal/YYYY-MM-DD.md` | Date, weekday, sky stamp, `## Log`, `## Tasks` |
| Idea (Ctrl/Cmd+Shift+I) | `<domain>/Drafts/Idea YYYY-MM-DD HHMM.md` | Heading only |
| Scrap | `<domain>/Scraps/` | §7.2 |
| Clicking a `[[link]]` to a missing note | Beside the linking note | A heading with the link name |

Characters not allowed in file names (`\ / : * ? " < > | # ^ [ ]` and control characters) become spaces; names are cut at 120 characters. If the name exists, ` 2`, ` 3` and so on are appended.

### 4.2 Saving and file names

- Saves automatically after typing stops, on blur and when switching notes.
- When **Rename files** is "Follow the first heading" (default), the file is renamed when the first `# Heading` changes. It waits while the caret is still on the heading line.
- Moving or renaming a note or attachment rewrites `[[old name]]` / `![[old name]]` in other notes (some moves, such as archiving and promoting ideas, do not rewrite links).

### 4.3 Archive and trash

- **Archive**: moves the note to `<domain>/Archive/<original relative path>` and unpins it. It can be restored.
- **Trash**: moves the file to `.trash/<original path>` at the vault root. The Trash view restores or deletes permanently.

### 4.4 Title

The title is, in order of precedence:

1. Front matter `title`
2. The first `# Heading` in the text
3. The file name without extension

## 5. Markdown

### 5.1 Supported syntax

| Syntax | Example |
|---|---|
| Headings | `#` to `######` (id `h-<slug>`) |
| Paragraphs and breaks | Blank line between paragraphs; line breaks inside become `<br>` |
| Emphasis | `**bold**`, `*italic*`, `~~strike~~`, `==highlight==` |
| Code | `` `code` ``, ```` ``` ```` / `~~~` fences (language as class `lang-xxx`) |
| Quotes | `> quote` |
| Callouts | `> [!note] Title` (type in the `data-c` attribute; `+` / `-` ignored) |
| Lists | `-` `*` `+`, `1.` `1)`, nested |
| Tasks | `- [ ]` open, `- [x]` done, `- [-]` cancelled, `- [/]` |
| Tables | GFM, with `:` alignment in the separator row |
| Rules | `***` `---` `___` |
| Links | `[text](URL)`, `[[Note]]`, `[[Note#Heading]]`, `[[Note|label]]` |
| Embeds | `![[image.png]]`, `![[image.png|300]]` (width), `![alt](path)`; other files appear as file links |
| Tags | `#tag`, `#a/b`. Digits alone are not a tag. Not read in headings or code. Allowed after CJK punctuation |

### 5.2 Link resolution

- `[[name]]` is matched against file names, ignoring case. Among duplicates, notes outside the archive come first, then one in the same folder as the linking note.
- A name containing `/` is matched against the end of the path.
- Unresolved links are shown faded (class `unres`).
- Attachments are found by vault path, then relative to the linking note, then by file name.

### 5.3 Editor behaviour

| Action | Behaviour |
|---|---|
| Enter at the end of a list line | Continues the marker; numbers increase by one; tasks continue unchecked. On an empty item, removes the marker |
| Tab / Shift+Tab | Indents or outdents list lines and selections; elsewhere inserts a tab |
| `[[` | Note name autocomplete (↑↓, Enter/Tab to accept, Esc to close) |
| Click a link | Opens it, also while editing |
| Option/Alt + click | Places the caret without opening the link |
| Paste or drop an image | Saves it in the attachments folder and inserts `![[file name]]` |

## 6. Tasks

### 6.1 Syntax (Obsidian Tasks compatible)

```markdown
- [ ] Check the source 📅 2026-11-10 ⏫
- [x] Define the question ✅ 2026-11-01
```

| Marker | Meaning |
|---|---|
| `📅 YYYY-MM-DD` or `due: YYYY-MM-DD` | Due date |
| `✅ YYYY-MM-DD` | Completion date (added automatically when checked, if Completion date is on) |
| `🔺` / `⏫` / `🔼` / (none) / `🔽` | Priority highest / high / medium / normal / low |
| `⏳` `🛫` `➕` + date | Removed from display (kept for other tools) |

### 6.2 Tasks view

- Source: tasks in ordinary notes and Atlas notes (cancelled and empty tasks excluded).
- Filters: Open / Due (due today or earlier) / Upcoming / No date / Done / All
- Groups: By date (Overdue, Today, Tomorrow, Next seven days, Later, No date, Done) / By note / By tag
- Order: due date, then priority (Done sorts by completion date, newest first)
- Checkboxes edit the line in the source note directly.

### 6.3 Quick add

The field at the top of the Tasks view appends a task to the task inbox (`Inbox.md` by default).

| Input | Result |
|---|---|
| `due:today` / `due:tomorrow` | Today / tomorrow |
| `due:3d` / `due:+3d` | In three days |
| `due:fri` | Next Friday |
| `due:2026-12-01` | That date |
| `!` / `!!` / `!!!` | 🔼 / ⏫ / 🔺 |

Example: `Check Pedretti on folio 263r due:tomorrow !!` → `- [ ] Check Pedretti on folio 263r ⏫ 📅 <tomorrow>`

## 7. Capture

### 7.1 Ideas

A place for quick thoughts (`<domain>/Drafts/`), kept out of All Notes. **Promote** moves the note to the same path without `Drafts/`, making it a working note.

### 7.2 Scraps

Unverified material brought in from outside (`<domain>/Scraps/`).

- **From the clipboard**: the Capture Scrap button. If the clipboard cannot be read, a dialog takes a paste instead.
- **AI sessions**: if the text contains a share link from claude.ai, chatgpt.com, gemini.google.com, grok.com, perplexity.ai and similar, the scrap gets `kind: ai` and `session:`.
- Front matter on creation: `type: scrap`, `captured`, `source` or `session`, `status: unverified`.
- The Scraps view filters by source with chips.

### 7.3 Link inbox and link maker

- **Link maker**: paste a URL; the title and site are read (via microlink.io, or guessed from the URL) and turned into a citation that can be copied, inserted into a note or added to the inbox.
- Citation formats:

  | Format | Output |
  |---|---|
  | Source line | `Source: [Title](URL), Site, accessed 2 November 2026` |
  | Markdown link | `[Title](URL)` |
  | Cite callout | `> [!cite] Title` + site, URL and access date |
  | Plain text | `Title, Site. URL (accessed 2 November 2026)` |

- **Link inbox** (`Links/Link inbox.md` by default): added links are written as `status: pending`, to be checked and merged into the built-in list. Categories: Manuscripts / Collections / Reference works / Texts and libraries / Other.

## 8. Projects, Atlas and Corpus

### 8.1 Projects

- Each folder `<domain>/Projects/<name>/` is a project. Its hub is `<name>.md` in the folder, or a note in it with `stage` / `status` in its front matter.
- A single note directly in `Projects/` with `stage` / `status` is a one-note project; adding a note turns it into a folder.
- Stages:

  | Stage | Values accepted in `stage` / `status` |
  |---|---|
  | Idea | Anything not listed below |
  | Active | active, doing, in progress, wip |
  | On hold | paused, on hold, hold, waiting, blocked |
  | Done | done and similar |

- The project page groups notes as hub / material (from scraps) / idea / note and gathers their tasks.

### 8.2 Atlas

Template notes by kind under `<domain>/Atlas/`.

| Kind | Folder | `type` | Main front matter | Sections |
|---|---|---|---|---|
| People | `People/` | `person` | born, died, role, places, aliases | Life / With Leonardo / Works and documents / Sources |
| Places | `Places/` | `place` | lat, lon, aliases, stay, years | Leonardo here / Culture and institutions / Sources |
| Glossary | `Glossary/` | `term` | places, period, url, aliases | Definition / In Leonardo's world / Sources |
| Library | `Library/` | `book` | written, identified, recorded, edition | Leonardo's entry / Identification / Edition he could have used / Sources |

- **Cross references**: when a note's `places` front matter or a `[[link]]` matches an Atlas item's title, file name or `aliases`, it is listed among that item's connections.
- **About**: front matter `about: ["[[Item]]"]` ties a note to an Atlas or Corpus item.
- Seed data: Leonardo's places, people and the book list from his notebooks can be created in one step. Summaries and images can be fetched from Wikipedia.
- New items always start as `status: pending`.

### 8.3 Corpus

Catalogue of Leonardo's works (under `Corpus/` in the Leonardo domain).

| Kind | Folder | Examples |
|---|---|---|
| painting | `Corpus/Paintings/` | Annunciation, The Last Supper, Mona Lisa and others (16) |
| drawing | `Corpus/Drawings/` | Vitruvian Man and others (3) |
| codex | `Corpus/Codices/` | Codex Atlanticus, Codex Leicester and others (9), with `siglum` |

- Each work is a `type: work` note with sections Resources / Facts / Interpretation / Sources.
- Images can be fetched from Wikipedia and saved in `Corpus/Images/` (not automatically for codices). Images can also be added from a URL or by dropping them.

## 9. Sky

### 9.1 Places

Built-in places: Vinci / Florence (default) / Fiesole / Milan / Santa Maria delle Grazie / Mantua / Venice / Imola / Rome / Amboise.
Any place can be set by name search (Open-Meteo Geocoding) or current location (Here).

### 9.2 Modes

| Mode | Time | Weather |
|---|---|---|
| Live | Now | Current values from Open-Meteo (cached 15 minutes; drawn without weather if it fails) |
| Historical | A chosen date and time (Julian or Gregorian, local mean time). Default 9 February 1495, 20:00, Julian | Clear / Overcast / Rain / Snow / Fog / Thunderstorm |

### 9.3 Sky tab

- Dial showing sun and moon altitude and azimuth
- Local time, date, weather, temperature, high and low, humidity and wind
- Sunrise and sunset, sun position, moon phase and illuminated fraction, moon position
- **Leonardo's reckoning**: Julian date, Gregorian date, Italian hour (hours from sunset), Florentine year (from 25 March), Venetian year (from 1 March)
- **Year style converter**: Florentine, Pisan and Venetian year styles to the modern year, with Julian and Gregorian dates and the weekday
- **Display**: motion, update interval (30 seconds to 15 minutes), falling rain and snow, stars, water studies (with rain and fog / always)

### 9.4 Background

- Palettes change with sun altitude: night / dawn / day / dusk.
- With Light and dark set to `auto` (default), the theme switches at a sun altitude of −4°.
- Animation stops under `prefers-reduced-motion` or with Motion off.

### 9.5 Sky stamp

A one-line record:

```
Florence · 2 November 2026 14:05 · Clear 15°C · Waxing crescent 12%
Milan · 9 February 1495 (Julian) 20:00 local mean time · Clear · Full moon 98%
```

## 10. Reference

| Tab | Content |
|---|---|
| Chronology | The Leonardo Chronology (six lanes, evidence layers, review status, automated checks). Its HTML is embedded as gzip + Base64 and shown in an iframe. Uses `Chronology/reviews.json` and `extra.json` from the vault when present; the built-in data can be saved to the vault |
| Places | Built-in places; use one for the sky, open in Google Maps or Earth |
| Works | Reference list of works |
| Links | Research sites (Codex Atlanticus viewer and others) |

## 11. Tools

| Tool | Description |
|---|---|
| Link maker | §7.3 |
| Save a session to Scraps | §7.2 |
| Research tools (Apps) | §12 |
| Index for other editors | Writes `index.md` at the vault root: per domain, Notes / Corpus / Ideas / Atlas / Scraps as wiki links (archive excluded) |
| Calculator | Log kept in the browser (up to 500 entries); can go into a note, the clipboard or a Markdown file |
| Sky stamp | Copies the stamp |
| Year style converter | Opens the converter in the Sky tab |

### 11.1 Calculator

- Operators: `+ - * / % ^`, parentheses; `× ÷ −` also accepted. Thousands separators are ignored.
- Functions: `sqrt cbrt abs exp ln log log2 floor ceil round sin cos tan asin acos atan atan2 hypot min max deg rad` (trigonometry in degrees)
- Constants: `pi` `π` `e` `phi` `φ`, previous result `ans`
- ↑↓ in the input walks the history.

## 12. Apps

Launcher for research tools.

- Built in: Velagrafia, FormaGGio, Leonardo Chronology (claude.ai artifacts), Google Maps, Google Earth and others.
- `{lat}` `{lon}` `{place}` in a URL are replaced with the place chosen in the Sky panel.
- Apps you add are written to the apps note (`Links/Apps.md` by default). Built-in apps can be hidden.

## 13. Domains

Split one vault into research areas. Defaults:

| id | Folder | Label | Colour |
|---|---|---|---|
| leonardo | `Leonardo/` | Leonardo | `#9B111E` |
| workbench | `Workbench/` | Workbench | `#3B2F4C` |

- With a domain selected, the list, tasks, Atlas and other views show only that domain's folder.
- New notes, ideas, scraps and daily notes go into the current domain.
- The definition is saved in `quaderno.json` at the vault root (created from Settings), together with the web clipper's routing rules (for example github.com → workbench).

## 14. Settings

| Section | Setting | Default |
|---|---|---|
| Appearance | Light and dark | Follow the sun (auto) |
| | Text face | Serif |
| | Text size | 17 (13 to 24) |
| | Line width (rem) | 44 (28 to 80) |
| | Open notes in | Edit mode |
| Files | Rename files | Follow the first heading |
| | Attachments folder | `Attachments` |
| | Daily notes folder | `Journal` |
| | Projects folder | `Projects` |
| | Domains | `quaderno.json` |
| | Link inbox note | `Links/Link inbox.md` |
| | Apps note | `Links/Apps.md` |
| | Capture buttons | Icons |
| | Page titles | Read online (microlink.io) |
| | Atlas / Ideas / Scraps / Archive folder | `Atlas` / `Drafts` / `Scraps` / `Archive` |
| | Task inbox | `Inbox.md` |
| | Completion date | Add ✅ date when a task is checked |
| Library | Current storage, switch, .zip export, import, copy to a folder | |
| Keys | Shortcut list | |

Settings live in the browser's `localStorage` (`quaderno:settings`), not in the vault (apart from `quaderno.json`).

## 15. Drag and drop

| Target | Behaviour |
|---|---|
| Note body | Attach images and files and embed them |
| Folder tree | Move notes into the folder |
| A project in the navigation | Add notes to the project |
| Views such as Scraps | Import text or files as notes |
| Image gallery of a Corpus or Atlas item | Save images for the item |
| Anywhere else | Import files |

## 16. Requirements

- Writing to a folder: Chrome, Edge or another browser with the File System Access API.
- Other browsers: the browser library (IndexedDB).
- The chronology needs `DecompressionStream` (gzip).
- At 1180px or less the drawer overlays the page; on narrow screens the list and the note are shown one at a time.
