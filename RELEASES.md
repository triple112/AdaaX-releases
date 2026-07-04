# AdaaX Release Runbook

**One-paragraph summary.** Step-by-step for cutting an AdaaX **installer** release into this public
feed so installed clients auto-update. The **portable** build is separate and is **not** published
here. Prerequisite: a GitHub token with `contents: write` on `triple112/AdaaX-releases`, exported as
`GH_TOKEN` in the shell that runs the publish.

## Prerequisites

- Build machine: Windows, Node ≥ 18, the app repo checked out at `Adaa_Utility_v2/Adaa_App`.
- `GH_TOKEN` = a **fine-grained** token scoped to **only** this repo, `contents: write`. Never commit it.
- The prebuilt `native/timerres/AdaaTimerRes.exe` present (so no `csc` JIT fallback ships).

## Steps

1. **Bump the version** in `Adaa_App/package.json` (and the `TitleBar.jsx` label). Semantic:
   patch for fixes, minor for features.
2. **Update the changelog** in `Adaa_Utility_v2/patch_notes.md` (app) — the release notes.
3. **Publish** from `Adaa_App/`:
   ```bash
   export GH_TOKEN=***          # fine-grained, this repo only
   npm run release              # build:renderer && electron-builder --win nsis --publish always
   ```
   electron-builder uploads `AdaaX-Setup-<version>.exe` + `latest.yml` (+ block-map) to a new
   **draft/published Release** on this repo.
4. **Verify the Release** shows the installer + `latest.yml` under the same tag (e.g. `v2.6.6`).
5. **Test the update** on a clean Windows VM: install the prior version, launch, confirm it detects
   and installs the new one on quit (under UAC elevation — the app runs `requireAdministrator`).

## Rollback

- If a release is bad: delete/mark it draft in the **Releases** tab. `electron-updater` clients only
  update to the latest **published** release, so unpublishing halts the rollout.

## Notes / gotchas

- **Portable builds must never auto-update.** The app gates on `!process.env.PORTABLE_EXECUTABLE_DIR`
  in `maybeCheckForUpdates()`; only the installed (NSIS) build checks this feed.
- **Unsigned today.** SmartScreen warns until an EV certificate is added (future US-LLC milestone).
- Full design + open decisions: `Adaa_platform/docs/06-packaging-and-release.md`.
