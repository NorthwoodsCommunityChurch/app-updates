# App Updates (Sparkle appcast host) — Project Context

The GitHub-Pages-hosted Sparkle **appcast feed** for every Northwoods Community Church macOS
app. It is not an app — it's a folder of static XML files served over HTTPS. Each macOS app
points its Sparkle `SUFeedURL` at one file here; Sparkle fetches the file, compares versions,
verifies the EdDSA signature, and downloads the update zip from a GitHub Release. Touch this
repo whenever you publish a new version of *any* Northwoods Mac app. Stack: static XML + GitHub Pages.

> **Read first:** org Sparkle workflow lives in `../App Updates/SPARKLE-GUIDE.md` and the release
> flow in `../RELEASE-PROCESS.md`. If you read one more thing after this file, read SPARKLE-GUIDE.md —
> it explains the two-version-field trap and the sign-the-downloaded-file rule that this repo enforces.

---

## Status — 2026-05-29
- **Stage:** production (live infrastructure — every shipping Northwoods Mac app depends on it)
- **Works:** 18 per-app appcast files served via GitHub Pages; apps auto-update against them today.
- **In progress / next:** none tracked.
- **Known issues:**
  - `README.md` is stale/wrong: it lists "Apps Using This Feed: (none yet)" (18 apps actually use
    it) and the URL has wrong casing (`northwoodscommunityChurch` — GitHub Pages hostnames are
    lowercase). Several `<link>` elements inside the XML carry the same casing typo. Cosmetic for
    `<link>` (Sparkle uses the `enclosure url`, not `<link>`), but the README should be corrected.
  - `appcast.xml` (no suffix) is the **legacy shared feed** — the org standard now requires each app
    to use its own appcast file. Do not add new apps to it.
  - `.gitignore` is currently untracked (uncommitted local change not yet pushed).

> Fresh Claude: read this Status block + the Update Protocol at the bottom and you're current.

## Out-of-session changes

### 2026-07-31 — work done from SMPTE <-> MIDI

- Added `appcast-smptetomidi.xml` (SMPTE to MIDI, bundle `com.northwoodschurch.smptetomidi`) —
  currently an empty channel skeleton. The app's v1.0.1 shipped pointing at the legacy shared
  `appcast.xml` and was never published to any feed; its `SUFeedURL` now points at the new
  per-app file for builds going forward.
- When SMPTE to MIDI's next release (v1.0.2+) is published: add the `<item>` to
  `appcast-smptetomidi.xml` AND mirror it into legacy `appcast.xml` with
  `sparkle:bundleIdentifier` so existing v1.0.1 installs migrate onto the per-app feed.
- Corrected the inventory table: `appcast-midiautomation.xml` is the separate MIDI Automation
  app (`NorthwoodsCommunityChurch/MIDI-Automation`), not SMPTE↔MIDI as the old label implied.
- Heads-up found while working: legacy `appcast.xml` items carry **no `sparkle:bundleIdentifier`**,
  so any app still pointing at it may be offered another app's update (same org signing key —
  it would install). Worth auditing which installs still use the shared feed.
- **Verify:** `curl -s https://northwoodscommunitychurch.github.io/app-updates/appcast-smptetomidi.xml | head -8`

## What it does
Static file host. There is no code, no build, and nothing to run locally. The repo is published
to GitHub Pages at `https://northwoodscommunitychurch.github.io/app-updates/`. Publishing an app
update = editing the relevant `appcast-<app>.xml` here (adding a new `<item>`) and pushing — GitHub
Pages serves the new file within a minute or so, and the next time the app polls Sparkle it offers
the update. The update zip itself is **not** stored here; it lives on the app's own GitHub Release,
and the appcast `<enclosure url>` points at it.

## Architecture
```
App (Sparkle, SUFeedURL = .../app-updates/appcast-<app>.xml)
   │  poll
   ▼
GitHub Pages  ──serves──▶  appcast-<app>.xml   (this repo)
                              │ <item> has sparkle:version, shortVersionString,
                              │ enclosure url → GitHub Release zip, edSignature
                              ▼
   App compares sparkle:version vs its CFBundleVersion, verifies EdDSA sig with its
   bundled SUPublicEDKey, downloads the enclosure zip, installs.
```
Sparkle public key used by all apps (public, safe to commit anywhere):
`VIMxKZmmRokdMcHK5d3QU4+qHgBglmkVFP5aAVvxgqM=`

### Where things live
```
app-updates/
├── README.md                       ← short blurb (currently stale — see Known issues)
├── appcast.xml                     ← LEGACY shared feed (Synaxis/Whisper Verses old entries) — do not extend
├── appcast-c200controller.xml      ← per-app feeds (one <channel> each, newest <item> on top)
├── appcast-dashboard.xml
├── appcast-dashboardagent.xml
├── appcast-whisperverses.xml
└── …one appcast-<app>.xml per app (full list below)
```
Per-app appcast inventory (file → channel title → # of published versions):

| File | App | Versions |
|---|---|---|
| `appcast-c200controller.xml` | C200 Controller | 35 |
| `appcast-whisperverses.xml` | Whisper Verses | 15 |
| `appcast-camerapositions.xml` | Camera Positions | 9 |
| `appcast-dashboard.xml` | AVL Dashboard | 8 |
| `appcast-dashboardagent.xml` | AVL Dashboard Agent | 8 |
| `appcast-dvdcopier.xml` | DVD Copier | 6 |
| `appcast-mirage.xml` | Mirage | 5 |
| `appcast-vocalistpositions.xml` | Vocalist Positions | 5 |
| `appcast-canopy.xml` | Canopy | 4 |
| `appcast-synaxis.xml` | Synaxis | 3 |
| `appcast-junkdrawer.xml` | Junk Drawer | 2 |
| `appcast-limbuslive.xml` | Limbus Live | 2 |
| `appcast-midiautomation.xml` | MIDI Automation (separate app — not SMPTE↔MIDI) | 2 |
| `appcast-minfancontrol.xml` | Minimum Fan Control | 1 |
| `appcast-photoingest.xml` | Photo Ingest | 1 |
| `appcast-prodcamerapositions.xml` | Production Camera Positions | 1 |
| `appcast-smptetomidi.xml` | SMPTE to MIDI | 0 (skeleton — first entry at v1.0.2) |
| `appcast-wayfind.xml` | Northwoods WayFind | 1 |

## Key identifiers
| Thing | Value |
|---|---|
| Type / stack | Static XML feed host (GitHub Pages) — no build, no runtime |
| GitHub repo | `NorthwoodsCommunityChurch/app-updates` |
| Served at | `https://northwoodscommunitychurch.github.io/app-updates/appcast-<app>.xml` |
| Bundle ID / package | n/a (not an app) |
| Current version | n/a (versioned per-app inside each appcast, not at repo level) |
| Update feed (Sparkle) | this repo *is* the feed |
| Sparkle public key (all apps) | `VIMxKZmmRokdMcHK5d3QU4+qHgBglmkVFP5aAVvxgqM=` (public, safe) |
| Secrets location | none here. The Sparkle **private** signing key lives in the project's OneDrive secrets folder — never in this repo. Appcasts hold only public signatures + public download URLs. |
| Production machine(s) | n/a (GitHub Pages) |

## Build / Run / Release
No build, no local run. "Release" here means publishing an app update by editing an appcast:

```bash
# This repo only hosts the feed. The release flow lives in the APP's repo + this one:
# 1. In the app repo: build, zip, create the GitHub Release, upload the zip.
# 2. Download the zip back FROM the GitHub Release (signatures are randomized — sign the
#    exact bytes users will download, per SPARKLE-GUIDE.md).
# 3. Sign the downloaded zip → produces sparkle:edSignature + length.
# 4. In THIS repo: add a new <item> to appcast-<app>.xml (newest item first) with
#    sparkle:version (== app CFBundleVersion / build number), shortVersionString (marketing),
#    pubDate, description CDATA, and the enclosure url/edSignature/length.
git -C "/Users/aaronlarson/Documents/VS Code/app-updates" add appcast-<app>.xml
git -C "/Users/aaronlarson/Documents/VS Code/app-updates" commit -m "Add <App> v<x.y.z>"
git -C "/Users/aaronlarson/Documents/VS Code/app-updates" push   # GitHub Pages serves it ~1 min later
```
Full canonical steps: `../App Updates/SPARKLE-GUIDE.md` and `../RELEASE-PROCESS.md`.
Ask Aaron before bumping any app's version.

## Conventions & gotchas
- **One appcast file per app** — never add a new app to the legacy shared `appcast.xml`. The org
  standard ("Every app MUST use its own appcast file — never the shared appcast.xml") is the whole
  reason the `appcast-<app>.xml` files exist.
- **`sparkle:version` must equal the app's CFBundleVersion (build number), not the marketing
  string.** If they diverge, Sparkle re-offers the same update forever. `sparkle:shortVersionString`
  is the human-facing marketing version. See the Sparkle-versioning notes in SPARKLE-GUIDE.md.
- **Sign the file users actually download.** EdDSA signing is randomized; signing a local zip and
  then uploading a "different but identical" zip makes verification fail. Upload → download → sign
  the downloaded bytes → put that signature in the appcast.
- **`enclosure url` points at the app's own GitHub Release**, not at anything in this repo. This
  repo never holds binaries.
- **Newest `<item>` goes on top** of the `<channel>`. Sparkle scans for the highest version, but
  keep them ordered for human readability.
- **GitHub Pages casing:** the served hostname is all-lowercase
  (`northwoodscommunitychurch.github.io`). The mixed-case strings in some `<link>` tags and the
  README are typos; harmless for updates (Sparkle uses the enclosure URL) but worth fixing.
- **No secrets ever in this repo** — only public signatures and public URLs. The private signing
  key stays in OneDrive.

## Update Protocol
Keep this file honest as you work:

| When you… | Update… |
|---|---|
| publish a new app version (add an `<item>`) | the inventory table version count · Status date |
| add a brand-new app's appcast file | Where things live (inventory table) |
| fix the README / casing typos | Known issues |
| change the publish/sign flow | Build / Run / Release · Conventions & gotchas |

End a work session with **`/save`** — it commits, pushes, syncs docs, and leaves cross-project notes.

## Document history
| Date | Change |
|---|---|
| 2026-05-29 | Initial CLAUDE.md to Northwoods standard (adapted for a docs/feed-host repo — no build/run) |
