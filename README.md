# AdaaX — Public Release Feed

**One-paragraph summary.** This repository is the **public update feed** for the **AdaaX** Windows
desktop app (أداء / Adaa — PC performance optimizer). It exists for one job: to host the
`electron-updater` artifacts — the NSIS **installer** `.exe` and the `latest.yml` manifest — as
**GitHub Releases**, so the installed app can auto-update without embedding a token (a public feed
needs no credentials to read). **No source code lives here.** The app's source is in the private
`Adaa_Utility_v2` repo; this repo is deliberately thin and public.

> 🤖 **For future AI agents / engineers.** This repo is intentionally minimal. Do not add source
> here. The authoritative architecture, packaging, and release design live in the umbrella
> `Adaa_platform/docs/` set — start at `docs/06-packaging-and-release.md`. When those docs and this
> README disagree, **the docs win.**

## Why a separate public repo

- The app repo (`Adaa_Utility_v2`) is **private**. `electron-updater` reading releases from a private
  repo would require a token **baked into the shipped app** — a secret-in-client anti-pattern.
- So the **installer track only** publishes here (public), and the app checks here for updates. The
  **portable track never auto-updates** (by policy — see the app's `main.js` `maybeCheckForUpdates()`).

## How releases get here (the pipeline)

Releases are produced from the app source (`Adaa_Utility_v2/Adaa_App`) by `electron-builder`, whose
`publish` config points at this repo:

```jsonc
// Adaa_App/package.json → build.publish
"publish": [{ "provider": "github", "owner": "triple112", "repo": "AdaaX-releases" }]
```

Publishing (from `Adaa_App/`, with a GitHub token in `GH_TOKEN` that can write releases here):

```bash
npm run release        # = build:renderer && electron-builder --win nsis --publish always
```

That uploads to a GitHub Release on **this** repo:
- `AdaaX-Setup-<version>.exe` — the NSIS installer,
- `latest.yml` — the manifest `electron-updater` reads to detect a newer version,
- block-map files for delta updates.

The installed app then finds the new `latest.yml` on launch and updates on quit.

## What is NOT here

- ❌ Source code, secrets, tokens, environment files.
- ❌ The **portable** build (that goes to the specialist's toolkit, not a public feed).
- ❌ Code signing (builds are currently **unsigned** — SmartScreen will warn until an EV cert is
  added under the future US LLC; see `docs/06`).

## Layout

```
AdaaX-releases/
  README.md      ← you are here
  RELEASES.md    ← the step-by-step release runbook
  (Releases tab) ← the actual installer + latest.yml artifacts live as GitHub Releases
```

## Cross-references (in the private umbrella repo)

| Doc | What |
|-----|------|
| `docs/06-packaging-and-release.md` | dual-track packaging, updater, signing, this feed |
| `docs/00-overview.md` | the platform, the two surfaces |
| `docs/08-roadmap.md` | phased backlog (P1-6 = installer/updater) |

_Maintained as part of the Adaa platform. Source of truth is the code + `docs/`._
