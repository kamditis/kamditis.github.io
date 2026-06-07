# kamditis.github.io -- developer takeover guide

**Repository kind:** Static website (GitHub Pages).
**Focus:** public icon previews, add-in downloads, tutorials/guides (HTML).
**Analyzed:** 2026-06-07.

This guide is for a new sole developer taking over the repository. It assembles the user-interface, code-structure, distribution/release, and build/debugging analyses into one handoff, plus a takeover-readiness integrity check at the end.

> BIM glossary (defined once, used throughout): **BIM** = Building Information Modeling, the 3D-model-based design workflow these add-ins plug into. **Revit** = Autodesk's BIM authoring app; the Elite add-ins are DLLs that load into it. **Navisworks** = Autodesk's model-aggregation / clash-detection / model-review app (EliteClashManager, EliteIFC target it). **Add-in** = a plugin DLL loaded by Revit/Navisworks. **Point cloud** = a 3D scan of a real building (EliteScan analyzes it). **TFM** = target framework moniker, e.g. `net48` or `net8.0-windows`. None of that code runs in this repo -- this site only describes, documents, and distributes those add-ins.

---

## 0. Quick start for the new sole developer

1. **Open the repo:** `C:/Users/KevinAmditis/source/repos/kamditis.github.io`. It is a static GitHub Pages site -- no build, no framework, no package manager. Default branch is `master`; pushing to `master` deploys live in minutes (allowed for this repo only).
2. **Prerequisites:** a web browser (the only hard requirement). Optionally `git` + the `gh` CLI (authed as `kamditis`) for release/Pages inspection.
3. **First "build" = open `index.html`** in a browser, or run `python -m http.server 8080` from the repo root and visit `http://localhost:8080/` to match production paths.
4. **Start reading at `index.html`** -- the entire app (1584 lines, all HTML+CSS+JS inline). Jump to the `<script>` at line 1007: the `REGIONS` object (line 1123) is the single source of truth for all content; `customIcons` (1010), `ICON_MAP` (1060), and the `render*`/`switchRegion` functions are the dumb renderers.
5. **To change content, edit data (`REGIONS`), not markup.** Verify in browser, then `git add index.html && git commit && git push`.
6. **Trust `CLAUDE.md` + the code, NOT `README.md`** (its `.plugin-card` instructions are stale and wrong). `CLAUDE.md`'s version table is also stale -- `REGIONS` is authoritative.
7. **`whitelist.json` is live licensing** read by the add-ins (not by this site). Editing + pushing it grants/revokes users within minutes -- handle with care.
8. **Releases:** the add-in ZIPs are built in the private add-in repos' CI and cross-published to THIS repo's GitHub Releases under `EliteX-latest` tags. Here you only bump the matching `version`/`downloadUrl`/`downloadFile` strings in `REGIONS` (currently inert -- download buttons were removed).

---

## 1. User interface

This repo (`kamditis.github.io`) is a static GitHub Pages site, not a Revit/Navisworks add-in. There is no compiled code, no build step, no framework, and no server. The entire interactive surface is one file -- `index.html` (1584 lines, all HTML + CSS + JS inline) -- plus four standalone tutorial pages under `docs/` and one standalone marketing/guide page at the repo root.

The only external runtime dependencies are CDN links (no npm, no bundler):

- Google Fonts: `Space Grotesk` + `Space Mono` (in `index.html`); the `docs/` pages use `Inter` + `Fira Code`; the Coordinator Guide uses `Barlow` + `JetBrains Mono`.
- Lucide icons: `https://unpkg.com/lucide@latest/dist/umd/lucide.js` (loaded in `index.html` `<head>`, line 10). Note: `@latest`, unpinned.

### What the user actually sees

`index.html` renders a mock "Revit ribbon" UI as a product showcase. It does **not** download or run anything -- it is a styled catalog of every add-in tool. Top to bottom:

1. **Header** (`<header>`, lines ~923-939): logo + a "System Active" status pill whose dot color tracks the active add-in.
2. **Tabs** (`.tabs`, lines 942-947): four hard-coded buttons -- `ELITE SCAN`, `ELITE DRAFT`, `ELITE COORD`, `ELITE CORE` -- each carrying a `data-region` attribute (`scan` / `draft` / `coord` / `core`).
3. **Ribbon** (`#ribbon`, line 950): empty in markup; populated by JS (`renderRibbon`) with one `.panel` per panel and one `.tool-btn` per tool.
4. **Main content** (`.main-content`): a breadcrumb, a "detail card" (`#detail-main`) that shows either the add-in release/install info or a selected tool's description + embedded YouTube video, a right "Properties + Requirements + Documentation" sidebar (`.detail-sidebar`), and a "Changelog" column (`#changelog-section`).

All four tabs and every tool button are data-driven from one JavaScript object, `REGIONS` (defined at `index.html:1123`). You almost never touch HTML markup to change content -- you edit `REGIONS`.

### The data model (this is the whole content system)

`REGIONS` is keyed by region id (`scan`, `draft`, `coord`, `core`). Each region object has: `id`, `name` (e.g. `EliteScan`), `label` (tab text), `color` (hex accent), `colorVar` (CSS var), `released`, `version`, `downloadUrl`, `downloadFile`, `docsUrl`, `description`, `installation[]`, `addinRequirements[]`, and `panels[]`. Each panel has `title` + `tools[]`. Each tool object has:

```js
{ id: 'run_analysis', label: 'Run Analysis', icon: 'play',
  fn: 'Compares your Revit model against point cloud scans...',
  videoRef: '',            // YouTube video ID, or '' for none
  changelog: 'v1.3.0: ...', // free text shown in the changelog column
  requirements: ['Scan locations placed with ScanID populated', ...] }
```

The render pipeline (all in the `<script>` at the bottom of `index.html`, starting at line 1007):

- `switchRegion(regionId)` (line 1525) -- sets `activeRegion`, toggles the active tab, sets `app.dataset.activeRegion` (this drives the per-region accent color via CSS attribute selectors like `[data-active-region="scan"] .tool-btn.selected {...}`), points the Documentation button at `region.docsUrl`, then calls `renderRibbon` + `showReleaseInfo`. This is the tab handler and wires the per-region `docsUrl` into the Documentation button (`#docs-btn`).
- `renderRibbon(regionId)` (line 1359) -- builds `.panel` / `.tool-btn` DOM for every tool, attaches a click handler calling `selectTool(tool)`, then calls `lucide.createIcons()`.
- `selectTool(tool)` (line 1414) -- marks the button `.selected`, updates the breadcrumb, calls `renderDetail` + `renderRequirements` + `renderChangelog`.
- `renderDetail(tool)` (line 1454) -- renders the tool icon, `tool.label`, `ID: tool.id` badge, the `fn` description, and either a 16:9 YouTube `<iframe>` (`https://www.youtube.com/embed/${tool.videoRef}`, `index.html:1497`) or a "No video reference available" box.
- `showReleaseInfo(region)` (line 1507) -- renders the default per-region overview.
- Boot: the script ends with `switchRegion('scan')` (line 1581).

### The icon pipeline (two layers -- important)

There is **no** `EliteResources/Icons` master-TIFF / multi-DPI (32/48/64/96/128) pipeline in this repo. That TIFF pipeline lives in the add-in repos and is referenced by the workspace `CLAUDE.md`, but this website renders icons as **inline SVG**, with a Lucide fallback. The `customIcons` set is imported from the add-ins' "EliteRibbons" icon set so the website icons match the Revit ribbon. Two layers:

1. **Custom SVG icons** -- the `customIcons` array (line 1010). Each entry is `{ id, group, path }` where `path` is raw inner-SVG markup drawn on a `viewBox="0 0 64 64"`. `createIconSVG(iconId, color)` (line 1110) wraps `path` in an `<svg>` and injects the region color via `<g style="color: ${color}">` so `stroke="currentColor"` / `fill="currentColor"` in the path picks up the accent.
2. **Mapping + fallback** -- `ICON_MAP` (line 1060) maps a tool `id` to a custom-icon id. In `renderRibbon`/`renderDetail` the logic is: look up `ICON_MAP[tool.id]`; if found and the custom icon exists, render the custom SVG; **otherwise** fall back to a Lucide icon named by `tool.icon` (`<i data-lucide="${tool.icon}">`), which `lucide.createIcons()` replaces.

So a tool's on-screen icon comes from `ICON_MAP[tool.id]` -> `customIcons` if present, else from `tool.icon` (a Lucide name like `play`, `search`, `layout`). See https://lucide.dev for valid names.

### Tutorials and the standalone guide

- `docs/EliteScan-Tutorial.html`, `docs/EliteDraft-Tutorial.html`, `docs/EliteCoord-Tutorial.html`, `docs/EliteCore-Tutorial.html` -- one self-contained styled "slide deck" per add-in (each ~1100-2100 lines, own inline CSS, print-friendly with `@media print`). Each opens with a `.back-nav` bar linking `../index.html` (e.g. `EliteScan-Tutorial.html:454`). They are reached from the site only via the sidebar **Documentation** button (`#docs-btn`, `index.html:991`), whose `href` is set per region from `region.docsUrl`. The button opens in a new tab (`target="_blank"`).
- `Coordination Intelligence - Coordinator Guide.html` (repo root, ~1786 lines) -- a standalone two-column guide page (its own `--accent-a` green theme, TOC sidebar). It is **not linked from `index.html`**; it is reached only by its direct URL. If you want it discoverable, you must add a link yourself.

### Deployment

GitHub Pages serves the `master` branch root directly. There is no `_config.yml`, no `CNAME`, and no `.nojekyll` in the repo. The live URL is `https://kamditis.github.io`. Per the repo `CLAUDE.md`, you push straight to `master` and changes go live within minutes. There is one GitHub Actions workflow, `.github/workflows/notifiy-teams.yml`, but it is **disabled** (its trigger is `workflow_dispatch` only; the file header says "DISABLED -- No one reads the Teams posts") and it has nothing to do with rendering the site.

### Worked example 1 -- add a new tool button end-to-end

Goal: add a "Section Cuts" button to EliteCoord's "Views" panel, with a custom icon and a YouTube demo. Every edit is in `index.html`.

1. **Add the tool to `REGIONS`.** Find the `coord` region's `Views` panel `tools` array (the panel with `title: 'Views'`, starting ~line 1265, containing `create_scopes`, `reset_scope`, `find_in_2d`). Add a new object to that array:
   ```js
   { id: 'section_cuts', label: 'Section Cuts', icon: 'scissors',
     fn: 'Creates aligned section views at chosen grid intersections for coordination review.',
     videoRef: 'YOUTUBE_ID_OR_EMPTY',
     changelog: 'v1.0.15: New command.',
     requirements: ['Active 3D view', 'Grids defined in model'] }
   ```
   `icon: 'scissors'` is the Lucide fallback used if you skip step 3. Confirm the name exists at https://lucide.dev.
2. **(Optional) draw a custom SVG icon.** In the `customIcons` array (line 1010), add an entry under the `// --- COORD ---` group. Author the path on a 64x64 grid and use `currentColor` so it inherits the region accent:
   ```js
   { id: 'SectionCuts', group: 'coord', path: '<rect x="8" y="8" width="48" height="48" stroke="currentColor" stroke-width="5" fill="none"/><line x1="8" y1="32" x2="56" y2="32" stroke="white" stroke-width="5" stroke-dasharray="4 3"/>' },
   ```
3. **(Optional) wire the custom icon.** In `ICON_MAP` (line 1060), under the `// Coord` comment, map the tool id to the custom-icon id:
   ```js
   'section_cuts': 'SectionCuts',
   ```
   If you skip steps 2-3, the button renders the Lucide `scissors` icon instead. Both are valid.
4. **Verify.** Open `index.html` in a browser, click the `ELITE COORD` tab, confirm the new button appears in the Views panel, click it, and confirm the detail card shows the description, the requirements list, the changelog text, and either the embedded video (if `videoRef` set) or "No video reference available". No build/restart -- just refresh.
5. **Ship.** `git add index.html && git commit && git push` to `master`. Live within minutes.

### Worked example 2 -- change an existing control (relabel, swap icon, change behavior)

Goal: rename EliteScan's "360 Viewer" to "Panorama Viewer", give it a different icon, and add a demo video. Only `index.html` is edited; here is every spot.

1. **Relabel** -- in `REGIONS.scan`, the `Viewer` panel, the tool with `id: '360_viewer'` (line 1160): change `label: '360 Viewer'` to `label: 'Panorama Viewer'`. This updates the ribbon button caption, the breadcrumb, and the detail-card heading (all read `tool.label`). Do **not** change `id: '360_viewer'` -- `id` is the key both `ICON_MAP` and `selectTool`/`.selected` matching rely on; renaming it silently breaks the icon mapping and the selected-state highlight.
2. **Swap the icon (Lucide route)** -- on the same tool object, change `icon: 'globe'` to e.g. `icon: 'panorama'`. This only takes effect if there is **no** custom-icon mapping for `360_viewer`. There currently *is* one: `ICON_MAP['360_viewer'] = '360Viewer'` (line 1066) -> the custom SVG `{ id: '360Viewer', ... }` (line 1016). So to actually change the rendered icon you must either:
   - **(a) edit the custom SVG** -- change the `path` of `{ id: '360Viewer', ... }` in `customIcons` (line 1016); or
   - **(b) drop to Lucide** -- remove the `'360_viewer': '360Viewer'` line from `ICON_MAP`, after which `icon: 'panorama'` takes over.
3. **Add a demo video (behavior change)** -- set `videoRef` on the tool from `'-glwNk8CL8c'` to your new YouTube id. The detail card switches from the existing embed to the new one automatically; `''` would instead show the "No video reference available" box.
4. **Update its changelog text** -- edit the tool's `changelog` string; it renders verbatim in the right-hand Changelog column.
5. **Verify** in the browser (EliteScan tab -> click the renamed button) and push to `master`.

Files edited in this example: `index.html` only (the tool object in `REGIONS`, plus `customIcons` or `ICON_MAP` depending on which icon route you chose).

### UI gotchas specific to this repo

- **No WPF/XAML, no WebView2 here.** The WPF `*_wpftmp.csproj` global-usings issue and the WebView2 dockable-pane crash documented in the workspace memory belong to the add-in repos, not this static site. Don't apply those fixes here.
- **`README.md` is stale and wrong.** It tells you to copy a `<div class="plugin-card">` block. There are no `.plugin-card` elements in `index.html` -- the real model is the JS `REGIONS` object. Trust `CLAUDE.md` + the code, not `README.md`.
- **`CLAUDE.md` version table is stale.** It lists EliteScan v1.2.4 / EliteDraft v1.0.14 / EliteCoord v1.0.4 / EliteCore v1.0.8. The actual `REGIONS` data is EliteScan `v1.3.0`, EliteDraft `v1.0.21`, EliteCoord `v1.0.14`, EliteCore `v1.0.14`. The single source of truth for what renders is `REGIONS` in `index.html`. The `CLAUDE.md` "EliteCore Panels" list is also stale -- it omits the `Sheets` panel (`elite_sheets` tool).
- **Downloads were removed but dead data remains.** `index.html:1351` and `:1544` carry comments "downloads no longer public" / "Download buttons removed". The site renders no download button, yet every region still has `downloadUrl` / `downloadFile` fields and `CLAUDE.md` still documents a download-hosting + version-bump workflow. Those fields are currently inert. If you re-enable downloads you must add the rendering code back, not just edit the data.
- **Three tools fall back to Lucide because they have no custom icon mapping.** `replace_ceilings` (line 1261, Lucide `layout-template`), `find_in_2d` (line 1269, Lucide `search`), and `elite_sheets` (line 1326, Lucide `layout`) exist in `REGIONS` but are absent from `ICON_MAP`. That's intentional-by-default (Lucide fallback works), but if you want them to match the hand-drawn icon style, add `customIcons` + `ICON_MAP` entries.
- **Orphaned / broken icon references.** `customIcons` defines `AuditHangers` (line 1029) but nothing in `ICON_MAP` or `REGIONS` uses it (dead asset). Conversely `ICON_MAP['360_pane'] = '360Pane'` (line 1067) points at a custom icon `360Pane` that does **not** exist in `customIcons`, and there is no tool with `id: '360_pane'` either -- so that mapping is dead. Don't assume an `ICON_MAP` entry means a working icon; verify the `customIcons` id exists.
- **`.gitignore` lists `CLAUDE.md` but it is already tracked.** `git ls-files` shows `CLAUDE.md`; the `.gitignore` entry doesn't untrack it. Edits to `CLAUDE.md` will still show in `git status` and can be committed.
- **Per-region accent color is driven by one attribute, not per-element classes.** `app.dataset.activeRegion` (set in `switchRegion`) flips `data-active-region` on `.app`, and CSS attribute selectors recolor selected buttons, glows, download/docs buttons, and badges. If you add a fifth region you must add: a `--newregion` CSS var block (line ~22), all the `[data-active-region="newregion"]` selectors, a `<button class="tab" data-region="newregion">` in the tabs markup, and the `REGIONS.newregion` object.

---

## 2. Code structure

This repo is the public face of the Elite/EliteCAD product family: a static GitHub Pages site that hosts the marketing/landing UI, per-add-in HTML tutorials, a "Coordination Intelligence" guide, and the license whitelist that the Revit/Navisworks add-ins read at runtime. There is no build step, no framework, no package manager, and no server-side code. Everything is plain HTML/CSS/JS served as-is by GitHub Pages off the `master` branch. For the actual add-in code, frameworks, and versioning rules see the sibling repos and `EliteResources/VERSIONING.md` / `EliteResources/HOTRELOAD-PATTERNS.md`.

### What is actually in the repo (verify before trusting the docs)

The committed `CLAUDE.md` and `README.md` are out of date: both claim the repo is only `index.html` + docs. It is not. Run `git ls-files` to see the real tracked set. As of this writing, tracked + deployed files are:

| Path | Role |
|------|------|
| `index.html` | Single-page landing/UI app. All HTML, CSS, and JS inline (1584 lines). Entry point of the site. |
| `whitelist.json` | License allowlist. Served as a static asset; fetched at runtime by the add-ins, NOT by any page in this repo. |
| `docs/EliteScan-Tutorial.html` | Standalone HTML tutorial, linked from `index.html` per-region `docsUrl`. |
| `docs/EliteDraft-Tutorial.html` | Same pattern. |
| `docs/EliteCoord-Tutorial.html` | Same pattern. |
| `docs/EliteCore-Tutorial.html` | Same pattern. |
| `Coordination Intelligence - Coordinator Guide.html` | Standalone guide (138 KB). NOT linked from `index.html`; reached only by direct URL. |
| `CLAUDE.md`, `README.md` | Docs for this repo (both stale -- fix them as part of any nontrivial change). |
| `.github/workflows/notifiy-teams.yml` | Disabled GitHub Action (`on: workflow_dispatch` only; release trigger is commented out). Note the misspelled filename. |

Untracked, present in the working tree, do NOT deploy and should be left out of commits (`.gitignore` ignores `CLAUDE.md` and `.claude/` but NOT these): `playground.html` (a local-only CSS-tuning tool that fetches `index.html` into an iframe), a stray `nul` file, and a `%TEMP%/` directory containing a leftover release ZIP. Confirm with `git status --short`.

### There is no .sln / .csproj / Python package

This is the one repo in the workspace with no compiled project. There are no target frameworks, no NuGet packages, no namespaces, and no `Views/ViewModels/Services/Models/Resources` directory convention -- those apply to the sibling Revit/Navisworks add-in repos, not here. The only "framework" dependencies are two CDN links in the `<head>` of `index.html`:

- Google Fonts (`Space Grotesk`, `Space Mono`) via `fonts.googleapis.com`.
- Lucide icons via `https://unpkg.com/lucide@latest/dist/umd/lucide.js` (note: `@latest`, unpinned).

### index.html architecture (the only "application" here)

`index.html` is a tab-based single-page app. The JS lives in one `<script>` block starting at line 1007. The architecture is a small data-driven renderer:

1. **`REGIONS` object** (`index.html:1123`) -- the master data model. One key per add-in: `scan`, `draft`, `coord`, `core`. Each region holds `released`, `version`, `downloadUrl`, `downloadFile`, `docsUrl`, `description`, `installation[]`, `addinRequirements[]`, and `panels[]`. Each panel has a `title` and a `tools[]` array; each tool is `{ id, label, icon, fn, videoRef, changelog, requirements[] }`. This object is the single source of truth for what shows on the page.
2. **`customIcons` array** (`index.html:1010`) -- inline SVG path data for hand-drawn icons, each tagged with `id` and a `group` (`scan`/`draft`/`coord`/`core`). Imported from the add-ins' "EliteRibbons" icon set so the website icons match the Revit ribbon.
3. **`ICON_MAP` object** (`index.html:1060`) -- maps a tool's `id` (e.g. `'place_scans'`) to a `customIcons` `id` (e.g. `'PlaceScans'`). `createIconSVG()` (`index.html:1110`) resolves it; if no custom icon is mapped, the renderer falls back to the Lucide icon named in `tool.icon`.

Rendering functions (all in the same `<script>`): `renderRibbon(regionId)` (`index.html:1359`) builds the tool buttons for a region; `selectTool(tool)` (`index.html:1414`) is the click handler that drives `renderDetail` / `renderRequirements` / `renderChangelog`; `renderDetail(tool)` (`index.html:1454`) renders the description + a 16:9 YouTube iframe when `videoRef` is non-empty; `showReleaseInfo(region)` (`index.html:1507`) renders the default per-region overview; `switchRegion(regionId)` (`index.html:1525`) is the tab handler and wires the per-region `docsUrl` into the Documentation button (`#docs-btn`). The app boots at the bottom with `switchRegion('scan')` (`index.html:1581`).

**Stale behavior to know about:** download buttons were removed from the public UI. `switchRegion` contains the comment `// Download buttons removed - downloads no longer public` (`index.html:1544`), and `index.html:1351` notes the download DOM elements were removed. The `downloadUrl` / `downloadFile` fields still exist in every `REGIONS` entry (e.g. `EliteScan-v1.3.0.zip` at `index.html:1132`) but nothing on the page consumes them anymore. They are dead data kept in sync by the release process; do not assume they render anything.

### whitelist.json is the license source of truth (cross-repo coupling)

`whitelist.json` is the most important file to understand because it crosses the repo boundary. It is fetched at runtime by the add-ins via `EliteResources/src/EliteCAD.Shared/Services/LicenseService.cs`, which hardcodes the URL `https://kamditis.github.io/whitelist.json` (`LicenseService.cs:26`). Nothing inside *this* repo reads it (`grep -r whitelist *.html *.js *.json` returns no consumer here).

Its shape: `authorized_users[]` (Autodesk usernames / Windows accounts), `authorized_domains[]` (e.g. `@elitecaddesign.net`), and `features{}` mapping each add-in name (`EliteScan`, `EliteDraft`, `EliteCoord`, `EliteCore`, `EliteClashManager`, `EliteCADNavisTools`, `EliteCAD.Navis.IFCExport`) to allowed users (`["*"]` = all whitelisted users). `LicenseService.IsAuthorized` checks the Autodesk username, then the Windows UPN, then the SAM account name against this file (`LicenseService.cs:58-93`). **Consequence:** editing `whitelist.json` and pushing to `master` is a live production licensing change -- a typo locks people out or lets unauthorized users in. It deploys within minutes. Recent commits like `Add 'fisnik44CHC' to the whitelist` are exactly this. This is a one-way-door-ish change; treat it with care.

### Cross-version compatibility mechanisms

These live in the add-in repos, not here, but a website maintainer should know they exist because the website advertises "Revit 2022-2026" support (`index.html:1517`): the add-ins ship a single DLL that runs on Revit 2022-2024 (.NET Framework 4.8) and 2025-2026 (.NET 8.0-windows). The runtime shims are `ElementIdExtensions.GetIdValue()` (reflection picks `Value` on Revit 2024+ vs `IntegerValue` on 2022-2023) and `AssemblyResolver.Register()` (resolves NuGet conflicts between co-installed add-ins), both in `EliteCAD.Shared`. The unsigned-assembly "bump all add-ins together / additive-only" rule for `EliteCAD.Shared` is in `EliteResources/VERSIONING.md`. None of this affects the website except that version strings in `REGIONS` must match the ZIPs the release pipeline publishes.

### Design patterns in use

- Data-driven rendering: one `REGIONS` data object, dumb render functions, no framework. To change content you edit data, not markup.
- Convention over config for icons: `tool.id` -> `ICON_MAP` -> `customIcons`, with Lucide as the fallback.
- Static-asset-as-API: `whitelist.json` is a "config endpoint" consumed by external clients (the add-ins).

There is no MVVM and no plain-C#-simulation/thin-wrapper pattern here -- those are add-in-repo patterns.

### Worked example: add a new tool button to an existing add-in (EliteScan)

Goal: add a "Cloud Reset" style tool called "Export CSV" (`id: export_csv`) to the EliteScan "Analysis" panel, with a custom icon and a tutorial link. This is the most common change. Steps:

1. **Pick the region and panel.** Open `index.html`, find `const REGIONS = {` (`index.html:1123`), then the `scan:` region, then the `panels[]` entry whose `title` is `'Analysis'` (around `index.html:1149`).
2. **Add the tool object** to that panel's `tools[]` array. Follow the exact field order other tools use (`id`, `label`, `icon`, `fn`, `videoRef`, `changelog`, `requirements`). `id` must be snake_case and unique within the region. `icon` is the Lucide fallback name. Example entry:
   ```javascript
   { id: 'export_csv', label: 'Export CSV', icon: 'download', fn: 'Exports the current analysis results to a CSV file.', videoRef: '', changelog: 'v1.3.1: New command.', requirements: ['Analysis completed (current session or saved results)'] },
   ```
   Keep `videoRef` an empty string until you have a real YouTube video ID. Make `requirements[]` mirror the real preconditions enforced in the add-in source.
3. **(Optional) Add a custom icon.** If you want a hand-drawn icon instead of the Lucide fallback, add an entry to the `customIcons` array (`index.html:1010`) with a unique `id` (PascalCase), `group: 'scan'`, and an SVG `path` string drawn on a `0 0 64 64` viewBox. Then map it in `ICON_MAP` (`index.html:1060`): add `'export_csv': 'ExportCsv',`. If you skip this step the button uses the Lucide `download` icon -- that is fine.
4. **Tutorial link is automatic.** The Documentation button uses the region-level `docsUrl` (`scan` -> `docs/EliteScan-Tutorial.html`, `index.html:1134`), not per-tool. If your tool needs docs, add a section to `docs/EliteScan-Tutorial.html`; there is nothing to wire up in `index.html`.
5. **No registration step beyond the data object.** There is no DI container, no command map, and no ribbon-builder file. `renderRibbon` (`index.html:1359`) iterates `REGIONS[regionId].panels[].tools[]` and creates a button per tool automatically; `selectTool` (`index.html:1414`) is bound on each button. Adding the object IS the registration.
6. **Verify locally.** Open `index.html` directly in a browser (or `python -m http.server` from the repo root) and click EliteScan -> Analysis -> Export CSV. Confirm the icon, description, requirements list, and (empty) video placeholder render. The `playground.html` tool can also load `index.html` in an iframe if you are tuning CSS.
7. **Update the stale docs and ship.** This is a static site that deploys on push to `master` (allowed for this repo per its `CLAUDE.md` Git Rules), live within minutes. If your change touched anything structural, also fix `CLAUDE.md`/`README.md` since both currently understate the repo contents.

To add a whole new add-in region instead of a tool, add a new top-level key to `REGIONS` (mirroring the `scan` shape), add a matching tab element in the header markup, point `docsUrl` at a new file in `docs/`, and add the add-in name to `whitelist.json`'s `features{}` so `LicenseService` will authorize it. (See also the UI section's list of all the per-region CSS/markup edits a fifth region requires.)

---

## 3. Distribution and release

This repo (`kamditis.github.io`) is the public face of distribution, but it is not where the build happens. Two distinct things ship from here:

1. **The website itself** -- a single static `index.html` (plus four tutorial HTML pages and one standalone guide), served by GitHub Pages straight from the `master` branch root. The site is the storefront: it shows version numbers, changelog text, and download buttons.
2. **The actual installable ZIPs** -- uploaded as GitHub *Release assets* on *this* repo (`kamditis/kamditis.github.io`). The private add-in repos (EliteCore, EliteDraft, EliteCoord, EliteScan, plus the Navisworks add-in EliteClashManager) build their own ZIPs in their own GitHub Actions and push them here, because a private repo can't host a public download but this public repo can.

Glossary, first use:
- **add-in / `.addin` manifest** -- Revit discovers add-ins by reading XML `.addin` files in a known folder. Each `.addin` names the DLL and the entry-point class. Navisworks uses a `.bundle` folder + `PackageContents.xml` instead.
- **GitHub Pages** -- GitHub's static-site hosting; serves files committed to a branch as a website.

### What this repo holds (verified)

```
kamditis.github.io/
|-- index.html                                  # storefront: REGIONS object holds versions + download URLs
|-- docs/
|   |-- EliteScan-Tutorial.html
|   |-- EliteDraft-Tutorial.html
|   |-- EliteCoord-Tutorial.html
|   `-- EliteCore-Tutorial.html
|-- Coordination Intelligence - Coordinator Guide.html   # standalone guide, not linked from index.html
|-- whitelist.json                              # license allow-list (read by the add-ins, not the site)
|-- .github/workflows/notifiy-teams.yml         # DISABLED (on: workflow_dispatch); kept for reference
|-- CLAUDE.md / README.md
|-- playground.html                             # NOT git-tracked (local only)
`-- nul                                          # 0-byte Windows reserved-name artifact, NOT git-tracked
```

There is **no GitHub Pages build workflow** in `.github/workflows/` -- the only workflow file is `notifiy-teams.yml`, and it is disabled (its trigger is `on: workflow_dispatch`, the `release:` trigger is commented out). So Pages is configured in repo Settings as classic "Deploy from a branch" = `master` / root. Deployment = `git push` to `master`. The site is live within a couple of minutes; there is no build step, framework, or bundler. `CLAUDE.md` explicitly allows pushing directly to `master` for this repo (it is the documented exception to the portfolio-wide "never push to master" rule -- confirm against the repo's own `CLAUDE.md` before relying on it).

### Where version numbers live in this repo

All four add-in versions are hard-coded as string literals inside the `REGIONS` JavaScript object in `index.html`. Each region (`scan`, `draft`, `coord`, `core`) has three fields that must move together on a version bump:

| Region key | `version` | `downloadUrl` | `downloadFile` |
|---|---|---|---|
| `scan` (line ~1130-1133) | `'v1.3.0'` | `.../EliteScan-latest/EliteScan-v1.3.0.zip` | `'EliteScan-v1.3.0.zip'` |
| `draft` (line ~1185-1188) | `'v1.0.21'` | `.../EliteDraft-latest/EliteDraft-v1.0.21.zip` | `'EliteDraft-v1.0.21.zip'` |
| `coord` (line ~1239-1242) | `'v1.0.14'` | `.../EliteCoord-latest/EliteCoord-v1.0.14.zip` | `'EliteCoord-v1.0.14.zip'` |
| `core` (line ~1286-1289) | `'v1.0.14'` | `.../EliteCore-latest/EliteCore-v1.0.14.zip` | `'EliteCore-v1.0.14.zip'` |

The ZIP filename embeds the version, so if you bump `version` and forget `downloadUrl`/`downloadFile`, the download button 404s. The download URL pattern is:

```
https://github.com/kamditis/kamditis.github.io/releases/download/{tag}/{filename}
```

where `{tag}` is the fixed per-add-in tag (`EliteScan-latest`, `EliteDraft-latest`, `EliteCoord-latest`, `EliteCore-latest`). The tag is reused every release -- the add-in's CI deletes and recreates it each time (see below). Per-tool, user-facing release notes also live in `index.html`, on each tool object's `changelog` string field (format `v1.0.X: plain-language description`).

> Note: download buttons were removed from the public UI (see Section 2's "Stale behavior"), so these fields are currently inert -- but the release process still keeps them in sync, and re-enabling downloads means re-adding the rendering code, not just editing the data.

### Where the version numbers do NOT live

Versioning is **decentralized**. The authoritative version of each add-in lives in its own private repo (e.g. `EliteCore/CLAUDE.md` says `Current Version: v1.0.14`, and the build stamps it via `-p:Version=$version`). This repo only mirrors those strings into the storefront. There is no single source of truth across repos; keeping them in sync is a manual step in each add-in's `/release` flow.

### How a ZIP actually gets here (cross-repo)

The build and upload happen in the *add-in* repo, not here. Worked through `EliteCore/.github/workflows/release.yml`:

1. Trigger: pushing a git tag matching `v*` (e.g. `v1.0.15`) to the add-in repo, or manual `workflow_dispatch`.
2. `build` job: a matrix over `R22 R23 R24 R25 R26` runs `dotnet build src/EliteCore/EliteCore.csproj -c "Release R<year>" -p:Version=$version` on `windows-latest`. It restores `EliteCAD.Shared` from GitHub Packages using the `PUBLISH_TOKEN` secret (a PAT with `read:packages`). Each year's add-in folder + `.addin` manifest is uploaded as a CI artifact.
3. `package` job: downloads all five year artifacts, lays them out as `Revit2022/ ... Revit2026/`, copies in `installer/Install.bat` + `installer/Install-EliteCore.ps1`, writes an `INSTALL.md`, then `Compress-Archive` into `EliteCore-v$version.zip`.
4. Two release-publish steps run only on tag pushes:
   - `softprops/action-gh-release@v1` cuts a normal release **in the add-in's own repo** (using the built-in `GITHUB_TOKEN`).
   - A `gh` CLI step publishes the same ZIP **to THIS repo** using the `PUBLISH_TOKEN` secret (a PAT with cross-repo `contents: write`). It runs, against `kamditis/kamditis.github.io`:
     ```
     gh release delete EliteCore-latest --repo kamditis/kamditis.github.io --yes
     gh api repos/kamditis/kamditis.github.io/git/refs/tags/EliteCore-latest --method DELETE
     gh release create EliteCore-latest --repo kamditis/kamditis.github.io \
       --title "EliteCore v$version" --notes-file RELEASE_NOTES.md \
       --latest=false  EliteCore-v$version.zip
     ```
   The `--latest=false` flag is load-bearing: marking a release "Latest" excludes the *other* add-ins' releases from the unauthenticated `GET /releases` API list that EliteUpdater relies on. The `RELEASE_NOTES.md` is written by the add-in's `/release` flow before tagging; if absent, the workflow generates a generic fallback.

Implication for a new maintainer: the cross-repo upload depends on the `PUBLISH_TOKEN` secret being set in **every add-in repo**, scoped to write Releases on this repo. If that PAT expires, builds succeed but the ZIP never lands here and the website download button points at a stale or missing asset. There is no alert for this beyond a red CI run in the add-in repo.

### Packaging convention: ZIP + Install.bat (DLL unblocking)

Every Revit add-in ships the same shape: a ZIP containing one folder per Revit year (`Revit2022 ... Revit2026`), each holding the `.addin` manifest + a subfolder of DLLs, plus `Install.bat`, the PowerShell installer, and `INSTALL.md`.

`Install.bat` is a one-liner that bypasses PowerShell execution policy:
```bat
@echo off
powershell -ExecutionPolicy Bypass -File "%~dp0Install-EliteScan.ps1"
```

The PowerShell installer (e.g. `EliteScan/installer/Install-EliteScan.ps1`) does the real work:
- **Unblocks every extracted file** -- `Get-ChildItem -Path $ScriptDir -Recurse | Unblock-File`. Windows stamps a `Zone.Identifier` "mark of the web" on files extracted from an internet-downloaded ZIP; Revit refuses to load blocked DLLs. This is why a manual copy of the DLLs (skipping Install.bat) often fails silently.
- Detects installed Revit versions by checking both `C:\Program Files\Autodesk\Revit <year>` and `%APPDATA%\Autodesk\Revit\Addins\<year>`, prompts the user to pick versions (or `A` for all).
- For EliteScan only, also installs the WebView2 runtime (used by the 360 panorama viewer) via the Microsoft bootstrapper if absent.
- Copies the per-year folder + `.addin` manifest into the install location.

There is **no Authenticode code signing** anywhere in this pipeline. DLLs are unsigned. Distribution security is "unblock + trust the source," not a signing certificate. (Note: this is separate from *strong-naming* -- `EliteCAD.Shared` is also not strong-named, which drives the versioning rules below.)

### Install / settings / logs locations

- **Revit add-ins:** `%APPDATA%\Autodesk\Revit\Addins\<year>\` -- the `.addin` manifest sits here, DLLs in a sibling subfolder (e.g. `...\Addins\2024\EliteScan\`). Revit creates the `<year>` folder only on first launch; the installer creates it if missing.
- **Navisworks add-in (EliteClashManager):** `%APPDATA%\Autodesk\ApplicationPlugins\EliteClashManager.bundle\` with `PackageContents.xml` at the bundle root and DLLs under `Contents\` (single bundle, no per-year split).
- **Settings:** `%LOCALAPPDATA%\EliteTools\<scope>\<Project>\...` -- scope is `Global` for app-wide settings or the project name for per-project data. Examples: `%LOCALAPPDATA%\EliteTools\Global\Scan\settings.json`, `%LOCALAPPDATA%\EliteTools\Updater\settings.json`.
- **Logs:** `%LOCALAPPDATA%\EliteTools\Global\<Project>\Logs\` (e.g. `...\Global\Scan\Logs\`).

These paths are produced by `EliteToolsStorage` / `ProjectIdentifier` in `EliteCAD.Shared` (`EliteResources/src/EliteCAD.Shared/Services/`). The website does not touch them; they are listed here so a maintainer fielding a support request knows where a client's logs and settings are.

### EliteUpdater: how a client receives an update

Clients are not expected to re-download from the website each time. The separate `EliteUpdater` repo builds **EliteTools Updater** -- a system-tray WPF app (.NET Framework 4.8, single EXE via Costura.Fody). It polls this repo's Releases:

- `EliteUpdater/src/EliteUpdater/Services/GitHubReleaseService.cs` calls `GET https://api.github.com/repos/kamditis/kamditis.github.io/releases` (unauthenticated, 60 req/hr -- fine for an updater), filtering for the known tags `EliteCore-latest`, `EliteDraft-latest`, `EliteCoord-latest`, `EliteScan-latest`, `EliteClashManager-latest`, `EliteUpdater-latest`.
- It parses the version out of each release **name** (e.g. `"EliteCore v1.0.14"` -> `1.0.14`) and reads the ZIP asset's `browser_download_url`.
- It compares against the installed DLL's `FileVersionInfo` per Revit year, and offers Update / Install. Background check runs ~every 6 hours; Windows toast notification on a new version (deduped per version in `settings.json`).
- The updater also strips the `Zone.Identifier` block on its downloads via a `kernel32.dll DeleteFile` P/Invoke, mirroring `Unblock-File`.

So the contract this repo must uphold for the updater to work:
1. Each release **name** contains `v<major>.<minor>.<patch>` (the regex is `v?(\d+\.\d+\.\d+)`).
2. The release carries a `.zip` (or `.exe`) asset.
3. The release is **not** marked "Latest" (`--latest=false`), or it drops out of the unauthenticated list and the updater can't see the others.

EliteUpdater itself is built by `EliteUpdater/.github/workflows/release.yml` (tag `v*` -> Costura single-EXE -> published here as `EliteUpdater-latest`).

### whitelist.json -- licensing, not distribution, but it lives here

`whitelist.json` is served by this site at `https://kamditis.github.io/whitelist.json` and is fetched at runtime by the add-ins' `LicenseService` (`EliteResources/src/EliteCAD.Shared/Services/LicenseService.cs`, constant `WhitelistUrl = "https://kamditis.github.io/whitelist.json"`). It lists `authorized_users`, `authorized_domains`, and per-add-in `features`. To grant or revoke a user, edit this file and `git push` to `master` -- it is live within minutes (same Pages deploy as the rest of the site). This is the only "release" action that does not involve a ZIP or a tag. Note the add-in code references both `https://kamditis.github.io/whitelist.json` (Pages URL) and `https://raw.githubusercontent.com/kamditis/kamditis.github.io/main/whitelist.json` in templates -- the active service uses the Pages URL; the `main` raw URL in `EliteResources/Templates/` is a stale branch name (this repo's default branch is `master`), so do not rely on it.

### Versioning rules that apply to this repo (from `EliteResources/VERSIONING.md`)

This repo holds no `.csproj`, so it does not directly trigger the versioning rules -- but a website maintainer must understand them because a bump here usually accompanies a portfolio-wide rebuild:

- **`EliteCAD.Shared` is unsigned -> bump all four add-ins together.** On .NET Framework, Fusion does simple-name-only binding for unsigned assemblies: the first `EliteCAD.Shared` loaded into Revit's AppDomain wins for the whole process, regardless of what each add-in compiled against. So whenever `EliteCAD.Shared` changes, **all four** add-ins must be rebuilt and re-released against the new version, and `EliteCAD.Shared` changes must be **additive-only** (no renames/removals/signature changes). Practically, this means a Shared bump produces four near-simultaneous releases -> four version bumps in this repo's `REGIONS` object in one sitting.
- **Per-Revit-year package pinning, never `$(RevitVersion).*` wildcards.** `Nice3point.*` and `RevitMCPSDK` are pinned to exact versions per Revit year inside each add-in's csproj `<PropertyGroup>`. Wildcards caused a real Revit-startup crash (`FileLoadException` HRESULT `0x80131621`) when two add-ins resolved different versions. Same "bump all together" rule applies. This is invisible from the website but explains why releases tend to come in batches of four.

A new maintainer of *this* repo doesn't edit csprojs, but should expect: "Shared changed" -> four ZIPs arrive in Releases -> four `REGIONS` entries to update.

### GitHub Packages publishing for EliteCAD.Shared

`EliteCAD.Shared` is published as a NuGet package to GitHub Packages at `https://nuget.pkg.github.com/kamditis/` (see the root `CLAUDE.md` "Publishing Commands"). This does not flow through this repo at all -- it is published from `EliteResources` and consumed by the add-in CI via the `PUBLISH_TOKEN` secret at restore time. Listed here for completeness because it is the upstream dependency that forces the "bump all four" cascade above.

### PyInstaller / OneDrive note (Coordination Intelligence)

The `Coordination Intelligence - Coordinator Guide.html` in this repo is a **static guide page only**. The Coordination Intelligence *application* is a separate Python project (`CoordinationIntelligence/`, with its own `updater.py`) distributed via a OneDrive share using a dedicated `/release` skill -- **not** through this repo's GitHub Releases and **not** via PyInstaller-to-this-repo. Do not treat the guide HTML here as the app's distribution channel. If you need to ship a new build of that app, use the `release` skill in the `CoordinationIntelligence` repo, not anything in `kamditis.github.io`.

### Worked example: cut and release a new version

Scenario: EliteCore has a bug fix, no `EliteCAD.Shared` change, going `v1.0.14 -> v1.0.15`. The add-in repo's `/release` slash command (`EliteCore/.claude/commands/release.md`) automates most of this; the steps below are what it does, so you can do it by hand or audit it.

In the **EliteCore** repo:

1. Decide the version: increment patch from `CLAUDE.md`'s `Current Version` -> `v1.0.15` (or use a version the user specifies).
2. Commit the source fix on a branch, then merge to `master` (or commit directly to `master` if already there).
3. Replace every `v1.0.14` with `v1.0.15` in `EliteCore/CLAUDE.md`.
4. Write `EliteCore/RELEASE_NOTES.md` with a `## EliteCore v1.0.15` heading, a `### What's New` bullet list (plain language), and an `### Installation` section. The CI reads this file verbatim into the release body on this repo.
5. Commit the version bump + notes: `git add CLAUDE.md RELEASE_NOTES.md && git commit -m "Bump version to v1.0.15"`.
6. Tag and push: `git tag v1.0.15 && git push origin master --tags`. This fires `EliteCore/.github/workflows/release.yml`, which builds R22-R26, packages `EliteCore-v1.0.15.zip` with `Install.bat`, cuts a release in EliteCore's own repo, and (via `PUBLISH_TOKEN`) recreates the `EliteCore-latest` release **on `kamditis.github.io`** with the new ZIP.

In **this** repo (`kamditis.github.io`):

7. Open `index.html`, find `core: {`, and update three fields together:
   - `version: 'v1.0.15'`
   - `downloadUrl: 'https://github.com/kamditis/kamditis.github.io/releases/download/EliteCore-latest/EliteCore-v1.0.15.zip'`
   - `downloadFile: 'EliteCore-v1.0.15.zip'`
8. Add user-facing `changelog` entries on the affected tool objects (format `v1.0.15: ...`). EliteCore tool IDs: `user_workset`, `link_worksets`, `setup_wizard`, `sync_settings`, `sync_log`, `tab_color_settings`, `toggle_colors`, `batch_export`.
9. If a button/command was added or changed, update `docs/EliteCore-Tutorial.html` (new `slide-section`, bump the `.slide-title .version`).
10. Commit and push to `master`:
    ```
    git add index.html docs/EliteCore-Tutorial.html
    git commit -m "Update EliteCore to v1.0.15 with changelog and docs"
    git push
    ```
    Pages redeploys within minutes; the download button now serves `EliteCore-v1.0.15.zip` from this repo's Releases, and EliteUpdater clients see the new version on their next poll.

11. Verify: load `https://kamditis.github.io`, click EliteCore's download, confirm the ZIP downloads (no 404). This catches the most common mistake -- a `version` bump with a stale `downloadUrl`.

Cross-repo coordination required:
- **If `EliteCAD.Shared` changed** (not the case in this example), repeat the whole flow for **all four** add-ins (EliteScan, EliteDraft, EliteCoord, EliteCore) -- see the VERSIONING rules above -- which means steps 1-6 in four repos and four `REGIONS` updates (`scan`/`draft`/`coord`/`core`) in steps 7-10 here.
- **If `Nice3point.*` / `RevitMCPSDK` was bumped**, same "all four together" cascade.
- **EliteClashManager (Navisworks)** follows the same tag-driven, `gh release create ... --repo kamditis/kamditis.github.io` pattern (tag `EliteClashManager-latest`) but is *not* currently shown in the website's `REGIONS` object -- only EliteUpdater consumes it. If you add it to the storefront, mirror the four-field pattern.

### Gaps / unknowns (distribution)

- The Pages deployment mode is inferred (no workflow file, classic branch deploy) -- confirm in the repo's GitHub Settings -> Pages if exact source/branch matters.
- `playground.html` and the `nul` file are present on disk but **not** git-tracked; they will not deploy. `nul` is a Windows reserved name and should not be committed.
- The `notifiy-teams.yml` workflow (note the misspelling in the filename) is disabled and references a Power Automate / Teams webhook that the file's own header says "no one reads." Treat it as dead unless re-enabled.

---

## 4. Build and debugging

This repository has **no build step, no compiler, no test runner, and no package install**. It is a static GitHub Pages site (free static hosting served straight from a Git branch). Everything that ships is plain HTML with inline CSS/JS and a few CDN `<link>`/`<script>` tags. "Building" it means editing an `.html` file; "deploying" it means pushing to `master`.

If you came to this section looking for `dotnet build`, `/buildall`, Revit API reference assemblies, NuGet GitHub Packages auth, xUnit/pytest, C++/CLI, or CloudCompare wiring -- **none of that lives in this repo.** See "What does NOT apply here" at the end for where it actually lives.

### What is actually in this repo (build view)

Run `git ls-files` from the repo root (`C:/Users/KevinAmditis/source/repos/kamditis.github.io`). The tracked files are:

| Path | What it is |
|------|-----------|
| `index.html` | The whole landing page / download portal. ~1,584 lines, all HTML+CSS+JS inline. Holds the `REGIONS` JS object with versions and download URLs. |
| `docs/EliteScan-Tutorial.html`, `docs/EliteDraft-Tutorial.html`, `docs/EliteCoord-Tutorial.html`, `docs/EliteCore-Tutorial.html` | Per-add-in tutorial pages. |
| `Coordination Intelligence - Coordinator Guide.html` | Standalone HTML guide (~1,786 lines). Note the spaces in the filename -- quote it on the command line. |
| `whitelist.json` | License allow-list (`authorized_users`, `authorized_domains`, `features`). Served as a public static file; the **add-ins** fetch it at runtime. `index.html` does NOT read it. |
| `README.md`, `CLAUDE.md` | Docs. Note: `README.md` describes an older "plugin-card" layout that no longer matches `index.html`; trust `CLAUDE.md` and the live file. |
| `.github/workflows/notifiy-teams.yml` | A Teams release-notification workflow that is **disabled** (its trigger is `workflow_dispatch` only, and a header comment says "DISABLED -- No one reads the Teams posts"). It does not build or deploy anything. |

External runtime dependencies are CDN-only and need no install: Google Fonts (`fonts.googleapis.com`) and Lucide icons (`https://unpkg.com/lucide@latest/dist/umd/lucide.js`, loaded at `index.html:10`). YouTube tutorial videos embed via `https://www.youtube.com/embed/${tool.videoRef}` iframes (`index.html:1497`).

There are also untracked stray files in the working tree you can ignore or delete: `playground.html` (a one-off "EliteCAD Visual Polish Playground", not part of the site), plus `nul` and a `%TEMP%/` folder (Windows shell-redirect junk). `.gitignore` only excludes `CLAUDE.md` and `.claude/`.

### Prerequisites

- A web browser. That is the hard requirement.
- Git, and the `gh` CLI (GitHub CLI) authenticated to the `kamditis` account, **only** if you need to upload add-in download ZIPs to Releases or inspect Pages status. Confirm with `gh auth status` (the working machine is logged in as `kamditis` with `repo` + `workflow` scopes).
- You do **not** need to close Revit or Navisworks, and you do **not** need any .NET SDK, Visual Studio workload, or Revit API reference assembly to work on this repo. Those prerequisites belong to the add-in source repos, not here.

### How deployment works (the "build")

GitHub Pages for this repo is the **legacy branch source**: it serves the repo root of `master` directly. Verified via `gh api repos/kamditis/kamditis.github.io/pages` -> `"build_type":"legacy"`, `"source":{"branch":"master","path":"/"}`, `"https_enforced":true`. There is no Pages build/deploy GitHub Action -- the Jekyll/static build is GitHub-managed.

Consequences:
1. **Push to `master` = live.** `CLAUDE.md` explicitly allows direct pushes to `master` here (this overrides the global "never push to master" rule for this static site). Changes go live within a minute or two.
2. Check whether the latest push deployed cleanly with `gh api repos/kamditis/kamditis.github.io/pages/builds/latest` and look at `"status"` (`built` = success) and `"error":{"message":...}`.
3. Because the source path is `/`, every tracked HTML file is reachable: `index.html` -> `https://kamditis.github.io/`, `docs/EliteScan-Tutorial.html` -> `https://kamditis.github.io/docs/EliteScan-Tutorial.html`.

### How the download links work (build view)

The site does not host the add-in ZIPs in Git. They are uploaded as assets on this repo's GitHub **Releases**, under fixed `-latest` tags, and `index.html` hardcodes the URLs. Current tags (from `gh release list --repo kamditis/kamditis.github.io`): `EliteScan-latest`, `EliteCore-latest`, `EliteDraft-latest`, `EliteCoord-latest`, `EliteClashManager-latest`. The URL pattern (e.g. `index.html:1132`) is:

```
https://github.com/kamditis/kamditis.github.io/releases/download/{tag}/{filename}
```

When an add-in ships a new version you must update **both** the GitHub Release asset AND the matching `downloadUrl`/`downloadFile`/`version` fields in the `REGIONS` object in `index.html` (the ZIP filename embeds the version, e.g. `EliteScan-v1.3.0.zip` at `index.html:1132-1133`). Forgetting the `index.html` edit leaves the button pointing at a 404 or a stale version.

### How to run / preview locally

You have two options. The trivial one is fine for content edits; use a local server when you want URLs and relative paths to behave exactly like production.

- **Direct file open (quickest):** double-click `index.html`, or open `file:///C:/Users/KevinAmditis/source/repos/kamditis.github.io/index.html` in a browser. Caveat: this is a `file://` origin, so anything fetched cross-origin can behave differently than on the live `https://` origin. (This repo doesn't `fetch()` anything, so it's usually fine -- but the add-ins' `whitelist.json` fetch should be tested on the live URL, not locally.)
- **Local static server (matches production paths):** from the repo root run `python -m http.server 8080` (Python ships with a built-in static server; no venv, no `pip install`, nothing to set up) and open `http://localhost:8080/`. If you prefer Node, `npx serve` works the same way. This serves `docs/` paths and root-relative links identically to GitHub Pages.

### Debugging

Use the browser DevTools (F12). There is no debugger to attach, no Revit.exe/Navisworks.exe process, no hot-reload loop in this repo. Practical checks:
- **Console tab** for JS errors in the `REGIONS`/render code (the page builds its UI from JS starting at `index.html:1007`).
- **Network tab** to confirm the CDN font/Lucide loads and that a download button resolves to a real Release asset (200, not 404).
- **Elements tab** to inspect inline CSS (all styles live in the `<style>` block, e.g. the `:root` color tokens at `index.html:12`).

The most common latent bugs you will hit are the dead-data / orphaned-icon cases (a dead `ICON_MAP['360_pane']` mapping with no `360Pane` custom icon and no `360_pane` tool; an unused `AuditHangers` custom icon; three tools intentionally on the Lucide fallback). See Section 1's "UI gotchas" for the full list -- these are exactly the class of thing that surfaces in the Console or as a wrong/missing icon.

### Tests

There are none, and none are needed -- there is no application logic to unit-test, no xUnit project, no `pytest`, no CI test job. "Testing" a change = open it in a browser and click through. Validate before pushing: links resolve, version strings match the Release, no console errors, the page renders on a narrow viewport.

### Worked example: change a version string, preview it, and ship it

Scenario: EliteScan just released `v1.3.1` and you've already uploaded `EliteScan-v1.3.1.zip` to the `EliteScan-latest` Release. Update the site. (The `v1.3.1` here is an illustrative scenario value -- the live `REGIONS` currently holds `v1.3.0`.)

1. Branch is optional here (direct push to `master` is allowed for this repo), but check state first:
   ```
   cd C:/Users/KevinAmditis/source/repos/kamditis.github.io
   git status
   ```
2. Find the EliteScan block in `index.html`. The version/download fields are around `index.html:1130-1133`:
   ```javascript
   released: true,
   version: 'v1.3.0',
   downloadUrl: 'https://github.com/kamditis/kamditis.github.io/releases/download/EliteScan-latest/EliteScan-v1.3.0.zip',
   downloadFile: 'EliteScan-v1.3.0.zip',
   ```
   Edit all three version-bearing values: `version: 'v1.3.1'`, and replace both `EliteScan-v1.3.0.zip` occurrences with `EliteScan-v1.3.1.zip`.
3. Confirm the Release asset actually exists (so the button won't 404):
   ```
   gh release view EliteScan-latest --repo kamditis/kamditis.github.io
   ```
   The asset list should contain `EliteScan-v1.3.1.zip`.
4. Preview locally with a server so paths match production:
   ```
   python -m http.server 8080
   ```
   Open `http://localhost:8080/`, select the EliteScan region, confirm it reads "v1.3.1", open DevTools Network, click Download, and verify the request resolves (200). Stop the server with Ctrl+C.
5. Commit and push (push to `master` publishes):
   ```
   git add index.html
   git commit -m "Update EliteScan to v1.3.1"
   git push origin master
   ```
   Commit-message style here matches history, e.g. `015c8e5 Update EliteScan to v1.3.0 with changelog and docs`.
6. Confirm the deploy:
   ```
   gh api repos/kamditis/kamditis.github.io/pages/builds/latest
   ```
   Wait for `"status":"built"` and `"error":{"message":null}`, then hard-refresh `https://kamditis.github.io/` (Ctrl+Shift+R to dodge browser cache).

That is the full inner loop for this repo: edit HTML -> preview in a browser -> push to `master` -> verify the Pages build.

### What does NOT apply here (and where it lives instead)

The new owner is taking over the whole Elite portfolio, so the build/debug gotchas are worth knowing -- but **none of them apply to this repo.** They apply to the sibling add-in repos under `C:/Users/KevinAmditis/source/repos/`:

- `/buildall` and `/releaseall`, `dotnet build`, per-Revit-year configs **R22-R26**, Debug vs Release, NuGet GitHub Packages auth via `gh auth token`, attaching a debugger to `Revit.exe`/Navisworks, the EliteDev hot-reload + `IDevPlugin` loop, and the MCP dev loop (`EliteDev/docs/mcp-dev-loop.md`) -> the .NET/Revit add-in repos (EliteDraft, EliteScan, EliteCoord, EliteCore, EliteDev, EliteClashManager, EliteIFC, ClashConfig).
- WPF `*_wpftmp.csproj` GlobalUsings via `Directory.Build.props`; Nice3point per-Revit-year pinning to avoid `FileLoadException 0x80131621`; `EliteScan.Native` C++/CLI `CompileAsManaged` config for net8; `-0.0` IEEE float-hash divergence between .NET Framework 4.8 and .NET 8; CloudCompare E57 `rowIndex` crash; `IDevPlugin` `ExternalEvent.Create`-in-`Execute` crash -> these are all .NET/Revit/C++ build-time issues with **no surface here** (this repo has zero `.csproj`/`.vcxproj`/`.py` files; `git ls-files` returns only `.html`, `.json`, `.md`, `.yml`, `.gitignore`).
- Cross-cutting Revit add-in rules (package version coupling, hot-reload startup pairing) -> `EliteResources/VERSIONING.md` and `EliteResources/HOTRELOAD-PATTERNS.md`. BIM/Navisworks API reference docs -> `EliteResources/Docs/`.

This repo's only coupling to those build systems is one-directional: when an add-in build produces a release ZIP, the publisher uploads it to this repo's Releases and bumps the matching string in `index.html`.

---

## 5. Takeover readiness

Per-area verdicts (from the takeover validator), with the concrete gaps a new sole developer must close.

| Area | Verdict |
|------|---------|
| (a) Make a UI change | PASS |
| (b) Release a new version | PASS |
| (c) Add new functionality | PASS |
| (d) Fix and test a bug | PARTIAL |

**Verified strengths.** Every cited path, JS symbol, line number, version string, cross-repo file, and stale-doc warning checked out against the real repo. `index.html` is 1584 lines as documented; all render-pipeline symbols are at the exact lines cited (`REGIONS:1123`, `customIcons:1010`, `ICON_MAP:1060`, `createIconSVG:1110`, `renderRibbon:1359`, `selectTool:1414`, `renderRequirements:1435`, `renderChangelog:1450`, `renderDetail:1454`, `showReleaseInfo:1507`, `switchRegion:1525`, boot at `1581`, Lucide CDN at `10`, YouTube embed at `1497`, `docs-btn` at `991`). Icon claims verified including the dead/orphaned cases. The data-driven model is real ("adding the object IS the registration" -- no DI/command-map/ribbon-builder file exists). Cross-repo release CI verified (`EliteCore/.github/workflows/release.yml`: cross-publish step, `--latest=false`, `PUBLISH_TOKEN`, per-year `Release R` build matrix; `EliteCore/.claude/commands/release.md` exists). Commit-message style matches history. Pages is confirmed legacy branch deploy on `master`, and the only workflow (`notifiy-teams.yml`) is confirmed disabled. `whitelist.json` licensing coupling verified against `LicenseService.cs`.

**Concrete gaps the new dev must close:**

1. **No automated test harness, and no repo-specific worked debugging example.** "Testing" is manual browser click-through only. A cold-start dev fixing an actual rendering bug (e.g. a broken icon mapping or a tool that won't select) has only the DevTools-console guidance to lean on -- there is no executable verification step beyond eyeballing the page. This is the genuinely weakest area and the reason (d) is PARTIAL. Closing it would mean either adding a minimal smoke check or, at minimum, writing one worked bug-fix example against the JS render logic.
2. **Conflicting version numbers across docs.** The root `CLAUDE.md` version table (EliteScan v1.2.4, EliteDraft v1.0.14, EliteCoord v1.0.4, EliteCore v1.0.8) and the build/debug worked example's illustrative values (EliteScan v1.3.1, EliteCore v1.0.15) both disagree with the live `REGIONS` data (EliteScan v1.3.0, EliteDraft v1.0.21, EliteCoord v1.0.14, EliteCore v1.0.14). `REGIONS` is the single source of truth; update or annotate the stale tables so a literal reader isn't misled. The `CLAUDE.md` "EliteCore Panels" list also omits the `Sheets` panel (`elite_sheets`).
3. **Stale `README.md`.** It instructs copying a `<div class="plugin-card">` block that does not exist in `index.html`. Fix or delete it.
4. **Adding a fifth region is a multi-touch change.** It requires ~5 coordinated edits (a `--newregion` CSS var block, all `[data-active-region="newregion"]` selectors, a `<button class="tab" data-region="newregion">`, the `REGIONS.newregion` object, and a `features{}` entry in `whitelist.json`). Adding a tool or panel (the common case) is trivial; a new region is the one place to slow down.
5. **Cross-repo release dependency is silent.** The ZIP cross-upload depends on the `PUBLISH_TOKEN` secret being valid in **every** add-in repo. If it expires, builds go green but no ZIP lands here and the download points at a stale/missing asset -- there is no alert beyond a red CI run in the add-in repo. (Also note download buttons are currently removed from the UI, so the `downloadUrl`/`downloadFile` fields are inert until re-enabled.)
6. **Pages config is inferred from API, not a committed file.** Confirm in repo Settings -> Pages if exact source/branch ever matters. Leave `playground.html`, `nul`, and `%TEMP%/` out of commits (`nul` is a Windows reserved name).

**First blocker:** none for the UI / release / add-functionality paths -- a cold-start reader can act immediately. The first place a literal reader could briefly stumble is the conflicting version numbers (gap 2): a reader who trusts `CLAUDE.md` before `REGIONS` would be momentarily misled, though this guide tells you `REGIONS` is authoritative, so it resolves in seconds. The genuinely weakest area is bug-fix *testing* (gap 1): verifying a fix means manual browser inspection only.

**Overall verdict: PASS.** The handoff is accurate and verifiable; every cited path, JS symbol, line number, version, cross-repo file, and stale-doc warning checked out against the real repo, and a new sole developer could execute all four tasks. Bug-fix/test is PARTIAL only because the repo has no automated tests, which the docs correctly disclose.
