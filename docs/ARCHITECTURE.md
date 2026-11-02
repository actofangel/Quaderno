# Quaderno architecture

Notes on the structure of `src/app.js` (minified, about 270 KB). Function and variable names are the ones that survive minification.
A formatter makes it readable:

```bash
npx js-beautify src/app.js > app.pretty.js
```

---

## 1. Overview

```
                ┌──────────── UI (direct DOM) ─────────────────┐
                │ renderNav / renderTree / renderList          │
                │ openNote / renderRead / renderView / Drawer  │
                └──────────────┬───────────────────────────────┘
                               │ refresh() (batched with requestAnimationFrame)
                ┌──────────────▼───────────────┐
 CFG (settings)▶│ V (index of the open vault)  │◀── parseNote()
 localStorage   │ notes / atts / dirs / trash  │
                │ names / attNames             │
                └──────────────┬───────────────┘
                               │ store.list / readText / write / move ...
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
            FSStore                      IDBStore
   File System Access API           IndexedDB "quaderno"
        (local folder)               (browser library)

 Separate: sky engine (astronomy → canvas drawing, Open-Meteo)
```

- No framework and no build-time dependencies. The DOM is built with `h(tag, attrs, ...children)` and `innerHTML` template strings.
- The single-file build (`dist/Quaderno.html`) inlines `app.css` as `<style>` and `app.js` as `<script>`.

## 2. Source map

| Lines (formatted) | Part | Main names |
|---|---|---|
| 1–60 | Helpers | `$` `$$` `h` `esc` `ymd` `today` `addDays` `sanitize` `fmtDate` `toast` `download` |
| 56–145 | Icons, setting defaults | `ICON` `ic()` `SKEY` `DEFAULTS` `CFG` `saveCfg` |
| 160–335 | Storage | `DB` `metaGet/metaSet` `IDBStore` `FSStore` |
| 338–500 | Vault index and domains | `V` `DOMAINS_DEFAULT` `VCFG` `readVaultCfg` `loadVault` `rebuildNames` `uniquePath` |
| 504–680 | Note parsing | `TAG_RE` `TASK_RE` `parseYaml` `splitFM` `setFM` `parseTask` `parseNote` `resolveNote` `resolveAtt` |
| 667–850 | Markdown rendering, editor highlighting | `inline` `wikiHTML` `embedHTML` `mdList` `md` `HLI` `hli` `hl` |
| 852–885 | .zip export | `CRC` `crc32` `makeZip` |
| 889–1440 | Astronomy, calendars, weather | `PLACES` `sunPos` `sunTimes` `moonPos` `moonIllum` `jdnJ/jdnG` `styleYear` `modernYear` `compute` `fetchWx` `stampText` |
| 1436–1670 | Sky drawing | `SKYPAL` `bands` `drawSky` `drawMoon` `drawVortex` `drawFx` `loop` `kick` `drawMini` `drawDial` |
| 1673–1900 | File operations | `refresh` `writeNote` `setNoteText` `updateLinkRefs` `moveFile` `trashFile` `restoreFile` `newNote` `dailyNote` `saveAttachment` `importFiles` `exportZip` |
| 1909–2120 | Navigation and list | `NAV` `renderNav` `folderTree` `renderTree` `setView` `matchQuery` `listNotes` `renderList` |
| 2120–2600 | Editor | `ED` `saveNow` `maybeRename` `openNote` `renderHead` `setMode` `renderRead` `insertText` `wrapSel` `AC` (autocomplete) `followLink` `toggleTask` `lightbox` |
| 2594–2900 | Dialogs, menus, Ideas, About | `modal` `ask` `confirmBox` `pickFolder` `noteMenu` `archiveNote` `newIdea` `promoteIdea` `mentionsOf` `setAbout` |
| 2900–3120 | Views | `renderView` `renderTasks` `parseQuick` `quickAddTask` `renderAtts` `renderReference` `renderTrash` |
| 3119–3420 | Drawer and Sky panel | `renderDrawer` `renderOutline` `renderLinks` `renderSkyPanel` `convert` `searchPlace` `insertStamp` `quickSwitch` |
| 3418–3630 | Settings and startup | `openSettings` `applyCfg` `welcome` `useStore` `seed` |
| 3628–3945 | Rescan, events, drag and drop | `rescan` `vaultIn` `renderOutside` `dropNotes`, global key handling |
| 3947–4470 | Links, Scraps, citations | `LINK_CATS` `LINKS` `parseLinkNote` `fetchMeta` `addLink` `createScrap` `quickScrap` `CITE_FORMATS` `citeText` `linkMaker*` |
| 4474–4550 | Tools, index.md | `renderTools` `renderToolsTab` `indexMd` `writeIndex` |
| 4551–5075 | Chronology | `CHRONO_URL` `CHRONO_GZ` `CHRONO_BUILTIN` `chronoHTML` `chronoData` `chronoDoc` `renderChronoTab` `chronoSave` |
| 5076–5275 | Apps | `APPS` `parseAppsNote` `allApps` `appHref` `renderAppsTab` `addApp` |
| 5277–5465 | Projects | `STAGES` `stageKey` `projectList` `projKind` `newProject` `addToProject` `renderProjects` `renderProject` |
| 5466–5850 | Corpus | `CORPUS_KINDS` `CORPUS_SEED` `SIGLA` `corpusText` `seedCorpus` `wikiImage` `saveImageLocal` `renderCorpus` `renderItemGallery` |
| 5850–6400 | Atlas | `ATLAS_KINDS` `atlasKind` `connections` `atlasTemplate` `newAtlas` `wikiSummary` `seedLeo*` `LEO_BOOKS` `renderAtlas` |
| 6407–end | Calculator | `CALC_FN` `CALC_CONST` `calcEval` `calcRun` `bindCalc` |

## 3. Main state

| Name | Content |
|---|---|
| `CFG` | Settings and UI state: `DEFAULTS` overlaid with `localStorage["quaderno:settings"]`. Saved by `saveCfg()` (150 ms debounce) |
| `V` | The open vault: `store`, `notes` (path → parsed note), `atts` (attachments), `dirs`, `trash`, `names` / `attNames` (lower-case file name → paths), `key` (vault identifier) |
| `VCFG` | Contents of `quaderno.json` (domains, clipper rules, whether it was found) |
| `UI` | Current view (`{ type, arg }`), pane, search query and so on |
| `ED` | The open note (`path`, `mode`, `dirty`) |
| `SKY` | Sky state (`mode: live|hist`, `hist`, weather `wx`, computed scene `st`, fetch `failed`) |

### 3.1 Parsed note

`parseNote(path, text, mtime, ctime)` returns:

```js
{
  path, name,          // path, file name without extension
  title, h1,           // title, first heading
  text,                // full text
  mtime, ctime,
  fm, fmLines,         // front matter, its line count
  tags: [], links: [], embeds: [],
  tasks: [{ path, line, indent, done, cancelled, text, due, doneOn, pri }],
  snippet
}
```

## 4. Storage interface

`FSStore` and `IDBStore` share these methods:

| Method | Description |
|---|---|
| `list(cached?)` | Every file as `{ path, dir, mtime, ctime, size, text }`. `FSStore` skips reading `.md` files whose `mtime` is unchanged (`text: undefined`) |
| `readText(path)` / `readBlob(path)` | Read |
| `write(path, data)` | Write a string or Blob; returns the new `mtime` |
| `remove(path)` / `move(from, to)` | Delete and move (`FSStore` moves by writing, then deleting) |
| `mkdir(path)` / `rmdir(path)` | Create a folder, delete one recursively |

Both have the properties `kind` (`folder` / `browser`) and `label`.

## 5. Rendering

1. File operations call `refresh()`.
2. `refresh()` batches with `requestAnimationFrame` and redraws `renderNav`, `renderTree`, `renderList`, the current view and the drawer.
3. Read mode renders with `md(text, lineOffset, note)`; `hydrate()` swaps `data-att` images for Blob URLs (`attURL`, cached).
4. Edit mode layers the `textarea` over `#mirror`, coloured by `hl()`. `#measure` is used to compute caret positions.

## 6. Sky engine

| Stage | Functions | Description |
|---|---|---|
| Time | `histMs` / `Date.now()` | The historical scene converts local mean time to UTC from the Julian day number (`jdnJ` / `jdnG`) and longitude |
| Bodies | `sunPos` `sunTimes` `moonPos` `moonIllum` | Sun and moon altitude and azimuth, sunrise and sunset, moon phase and illuminated fraction (simplified astronomical formulas) |
| Weather | `fetchWx` → `wxFromCode` | Maps Open-Meteo WMO codes to clear / cloudy / rain / snow / fog / storm |
| Scene | `compute()` | Combines the above into `SKY.st`; sun altitude sets the tone (light / dark) |
| Drawing | `drawSky` `drawFx` `loop` | Sky gradient, stars, moon, clouds, rain and snow particles, water-study vortices (`drawVortex`). `requestAnimationFrame` runs only while motion is needed; effects every 50 ms |
| Updates | `skyTimer` / `skyUpdate` | Recomputes at the chosen interval (30 seconds by default) |

Calendar conversion:

- `styleYear(style, y, m, d)`: modern year → Florentine (from 25 March), Pisan or Venetian (from 1 March) style
- `modernYear(style, y, m, d)`: the reverse

## 7. Embedded chronology

1. The chronology HTML is stored as the gzip + Base64 string `CHRONO_GZ`.
2. `chronoHTML()` expands it with `DecompressionStream("gzip")`.
3. `chronoData()` reads `Chronology/reviews.json` and `extra.json` from the vault, falling back to `CHRONO_BUILTIN`.
4. `chronoDoc()` injects the data and tone into the HTML as a `<script>`.
5. Normally shown with `iframe.srcdoc`. In the extension (`IS_EXT`), `chrono.html` is loaded and the HTML is passed with `postMessage({ qDoc })`.

## 8. Events

- A global `keydown` handles shortcuts (only while a vault is open).
- The `textarea` `keydown` handles list continuation, indentation, emphasis and autocomplete.
- Window `focus` and `visibilitychange` trigger `rescan()` (folder storage only, at most every 4 seconds).
- `resize` rebuilds the sky canvas and switches the narrow layout.
- Drag and drop has handlers on the view, folder tree, note, navigation, window and image gallery.

## 9. Network calls

| Function | Host |
|---|---|
| `fetchWx` | `api.open-meteo.com/v1/forecast` |
| `searchPlace` | `geocoding-api.open-meteo.com/v1/search` |
| `fetchMeta` | `api.microlink.io` (can be turned off in Settings) |
| `wikiImage` / `wikiSummary` | `<lang>.wikipedia.org/api/rest_v1/page/summary/...` |
| `saveImageLocal` | Image URLs such as Wikipedia's (to save them in the vault) |

## 10. Possible next steps

- Add the unminified source and split `src/` into modules (storage, Markdown, sky, views).
- Add a build script that produces `dist/Quaderno.html` from `src/`.
- Add tests for the DOM-free parts: Markdown and task parsing, calendar conversion, the calculator.
- Add `manifest.json`, `chrono.html` and icons for the extension build.
