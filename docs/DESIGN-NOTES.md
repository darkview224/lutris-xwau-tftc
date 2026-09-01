# Design notes

This is the *why* behind the scripts in `scripts/`. The [README](../README.md)
intentionally doesn't explain any of this — it's just install steps. If
you're troubleshooting, curious, or maintaining this repo, read on.

## Why three separate scripts instead of one

Manually, the simplest path to a working setup is duplicating one Lutris
entry and retargeting the copies, which shares one physical prefix across
all of them (no need to reinstall fonts/DPI/etc. three times). Lutris's
installer YAML format has no "duplicate an entry" primitive, and file
pickers can only select a **file**, not a folder — so a script can't be
handed a prefix *directory* directly.

These scripts get the same practical outcome — a real duplicate, not just
three independently-built prefixes — using every Wine prefix's
`system.reg` marker file: `02-xwau2025.yml` and `03-tftc.yml` each prompt
you to select `system.reg` inside the prefix `01-xwainstallermanager.yml`
built, then run `cp -a "$(dirname <selected system.reg>)/." "$GAMEDIR/"`
as their first install step. That physically clones the whole prefix —
fonts, DPI, Gecko/Mono, everything — before the rest of the script runs,
so only `01-xwainstallermanager.yml` needs its own `create_prefix` +
`winetricks`. Scripts 2 and 3 inherit whatever prefix state actually
worked when you tested script 1, rather than re-running winetricks and
hoping it produces an equivalent result.

If you point a later script's install directory at the *same* folder you
used for `01-xwainstallermanager.yml` (i.e. you're deliberately sharing
one prefix across every entry, rather than giving each its own), the
clone step detects that the selected `system.reg` already lives in
`$GAMEDIR` and skips the copy instead of erroring — `cp` refuses to copy
a directory onto itself.

The XWAU2025/TFTC folders themselves live wherever XwaInstallerManager
put them — they don't need to be inside the Wine prefix Lutris manages
for that entry. Wine can run an executable from any path regardless of
which prefix is active.

## Adjusting for a different vanilla XWA entry

`scripts/01-xwainstallermanager.yml` declares `requires:
star-wars-x-wing-alliance` — the public lutris.net installer slug for
vanilla X-Wing Alliance (GOG). If your own vanilla install used a
different Lutris entry, edit the `requires:` line in that script to match
its `game_slug`.

## Things to verify on your first real run-through

These are documented assumptions/best guesses that couldn't be confirmed
without actually running the tools:

- The exact executable name inside `XwaInstallerManager_WIP_*.zip`
  (assumed `XwaInstallerManager.exe`).
- The exact winetricks verb set for the recommended fonts (currently
  `corefonts consolas` — `corefonts` includes Arial and Verdana). This only
  needs to be right in `01-xwainstallermanager.yml`; scripts 2 and 3 clone
  its prefix rather than re-running winetricks.
- Whether `XWAU2025_Linux.zip` extracts directly into a flat file set or
  behind a single wrapping folder (adjust the `cp` source path in
  `02-xwau2025.yml`/`03-tftc.yml` if the latter).
- TFTC's actual executable name and folder layout (assumed `Alliance.exe`,
  same as XWAU2025, since TFTC runs on the same XWA engine).

If you find any of these wrong, please fix the corresponding script and
open a PR — that's exactly what community testing is for.

## Maintenance

The download URLs for `XwaInstallerManager_WIP_2608091320.zip` and
`XWAU2025_Linux.zip` are pinned to a specific dated release tag
(`20260808_linux`) from morallo's repo. As WIP builds are updated, check
https://github.com/morallo/xwa_ddraw_d3d11/releases for newer releases and
bump the two `files:` URLs across all three scripts accordingly.

## Provenance

These installer scripts and this documentation were produced with
[Claude Code](https://claude.com/claude-code).
