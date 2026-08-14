# App Updates

Public release-hosting and update-feed repo for Northwoods Community Church macOS apps.
This repo stays public so app repos can be private; everything apps and users download
at runtime is served from here.

It hosts three things:

1. **Sparkle appcasts** — one `appcast-<app>.xml` per app, served via GitHub Pages at
   `https://northwoodscommunitychurch.github.io/app-updates/appcast-<app>.xml`.
   (The shared `appcast.xml` is legacy — every app must use its own file.)
2. **Release zips** — each migrated app's signed zip is uploaded as a release of THIS
   repo, tagged `<tagPrefix>-v<version>` (e.g. `canopy-v1.1.0`). The appcast enclosure
   URL points at that asset. Same bytes as the original build, so the Sparkle
   `edSignature` stays valid.
3. **`catalog.json` + `icons/`** — the Canopy catalog manifest. Canopy reads
   `catalog.json` from Pages instead of scanning org repos, so catalog apps' repos can
   be private. Each entry's `tagPrefix` names where its releases live here; apps not
   yet migrated fall back to their own repo's releases via `legacyRepo`.

## Adding an app to Canopy

Add an entry to `catalog.json` (see the `comment` field there for the schema) and,
ideally, an icon PNG under `icons/`.

## Releasing an app (new model)

1. Build and sign the zip as usual.
2. `gh release create <tagPrefix>-v<X.Y.Z> -R NorthwoodsCommunityChurch/app-updates --title "<App> v<X.Y.Z>" <zip>`
3. Download the uploaded asset, sign THAT file with the app's Sparkle key, and put the
   signature + the app-updates download URL in the app's appcast here.
4. Commit and push. Verify Pages serves the updated appcast before announcing.

## How updates work

Each app checks its own appcast for updates. Sparkle verifies the EdDSA signature
before installing any update — downloads are rejected if the signature doesn't match.
