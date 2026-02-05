# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Public GitHub Pages website for EliteCAD Tools. Serves as the product landing page and **download host** for installers from private repos (EliteDraft, EliteScan, EliteCoord, EliteCore).

**Live URL:** https://kamditis.github.io

## Structure

```
kamditis.github.io/
├── index.html          # Single-page app (all HTML, CSS, JS inline)
├── CLAUDE.md           # This file
└── README.md
```

Everything is in `index.html` — no build step, no framework, no dependencies beyond CDN links (Google Fonts, Lucide icons).

## How It Works

### Region/Add-in System
The site uses a tab-based UI with four "regions" defined in a `REGIONS` JavaScript object:

| Region | Key | Color | Released |
|--------|-----|-------|----------|
| EliteScan | `scan` | `#5ED5DD` | Yes (v1.2.4) |
| EliteDraft | `draft` | `#F5B878` | Yes (v1.0.12) |
| EliteCoord | `coord` | `#6EE7A8` | Yes (v1.0.4) |
| EliteCore | `core` | `#A78BFA` | Yes (v1.0.3) |

Each region has:
- `released` (boolean) — controls whether download button shows or "Coming Soon"
- `version` (string) — displayed above download button
- `downloadUrl` — points to this repo's GitHub Releases (tag-specific)
- `panels[]` — groups of tools with their descriptions, requirements, and video references

### Tool Data Structure
Each tool in a panel has:
```javascript
{
    id: 'tool_id',
    label: 'Button Label',
    icon: 'lucide-icon-name',
    fn: 'Description of what the tool does',
    videoRef: 'YouTubeVideoID',    // Empty string if no video
    changelog: '',                  // Version-prefixed notes, e.g. 'v1.0.2: Fixed bug...'
    requirements: ['Requirement 1', 'Requirement 2']
}
```

### Video Embeds
When `videoRef` contains a YouTube video ID, the detail panel renders an embedded YouTube player using a 16:9 responsive iframe. When empty, it shows "No video reference available".

### Download Hosting
Private repos can't host public downloads, so ZIP packages are uploaded to this repo's GitHub Releases with tag-specific releases:
- `EliteDraft-latest` → `EliteDraft-v{version}.zip`
- `EliteScan-latest` → `EliteScan-v{version}.zip`
- `EliteCoord-latest` → `EliteCoord-v{version}.zip`
- `EliteCore-latest` → `EliteCore-v{version}.zip`

Download URLs follow the pattern:
```
https://github.com/kamditis/kamditis.github.io/releases/download/{tag}/{filename}
```

**Installation process:** Download ZIP → Extract → Double-click `Install.bat` → Select Revit versions → Restart Revit. The batch file handles PowerShell execution policy bypass and auto-unblocks DLLs.

**Important:** When updating versions, the `downloadUrl` and `downloadFile` fields in the REGIONS JS object must be updated to match the new version number, since ZIP filenames include the version (e.g., `EliteScan-v1.2.2.zip`).

### Icons
Custom SVG icons are defined inline in the `ICON_SVGS` object, mapped to tool IDs via `ICON_MAP`. Falls back to Lucide icons when no custom icon exists.

## Updating the Site

### Adding a new tool
Add an entry to the appropriate panel's `tools` array in the `REGIONS` object.

### Adding a video reference
Set the `videoRef` field to the YouTube video ID (the part after `youtu.be/` or `v=`).

### Updating version numbers
Change the `version` property in the region object. This displays above the download button.

### Releasing a new add-in
Set `released: true` and add a `version` property to the region object.

### Requirements
The `requirements` array should list exact preconditions from the source code: shared parameter names, family names, view types, etc.

## Git Rules

- Can push directly to `master` (this is a static site, deploys automatically via GitHub Pages)
- Changes are live within minutes of pushing
