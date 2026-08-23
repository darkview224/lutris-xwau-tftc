# Lutris scripts for XWAU2025 and TFTC

Lutris installer scripts that automate most of the Linux setup process for
the **XWAU2025** (X-Wing Alliance Upgrade) and **TFTC** (TIE Fighter Total
Conversion) mods for *Star Wars: X-Wing Alliance*, based on the manual
procedure documented at
https://github.com/morallo/xwa_ddraw_d3d11/wiki/XWAU-and-TFTC-in-Linux.

This automates everything that can safely be scripted. The parts that
involve downloading copyrighted mod content through a GUI tool are left as
manual steps, since that content can't be redistributed or driven headlessly.

## Prerequisites

- Lutris, Wine, and winetricks installed on your system.
- **Vanilla X-Wing Alliance (GOG)** already installed via a Lutris entry.
  These scripts default to `requires: star-wars-x-wing-alliance` (the public
  lutris.net slug) — if your local install used a different installer slug,
  edit the `requires:` line in `scripts/01-xwainstallermanager.yml`.
- A modern Wine build for the final two entries (wine-ge, GE-Proton, or a
  Wine 11-class build) — needed for HD cutscene playback via Media
  Foundation. Select this as the runner version for the XWAU2025/TFTC
  entries once they're installed.
- Internet access, to download two public release archives from
  [morallo/xwa_ddraw_d3d11](https://github.com/morallo/xwa_ddraw_d3d11/releases).

## How it works

Three separate installer scripts, run in order:

1. **`scripts/01-xwainstallermanager.yml`** — depends on your vanilla XWA
   entry. Creates a dedicated Wine prefix, installs the fonts XWAU
   recommends (Arial/Consolas/Verdana via winetricks), downloads
   `XwaInstallerManager` (a WIP Linux-compatible build published by the
   `xwa_ddraw_d3d11` project), and launches it. This becomes a permanent
   "reopen the mod manager" entry in your Lutris library.

2. **Manual step (outside any script):** in the running XwaInstallerManager,
   point "Vanilla Location" at your existing XWA install, pick a target
   directory, and install "XWAU2025 1.0.0" plus the "1.1.0 update". Later,
   reopen the same manager entry, pick a *different* target directory, and
   install TFTC the same way. This step can't be scripted — the mod content
   is fetched from xwaupgrade.com / TFTC's own hosting through the tool's
   own GUI downloader.

3. **`scripts/02-xwau2025.yml`** — depends on step 1's entry. Prompts you to
   select `Alliance.exe` inside the XWAU2025 folder you just created, and
   to select `system.reg` inside the prefix from step 1 (so it can clone
   that prefix into its own — see Design notes below). Downloads morallo's
   `XWAU2025_Linux.zip` compatibility patch and copies its contents into
   your XWAU2025 folder. Sets the `ddraw.dll`/`dinput.dll` overrides to
   native-then-builtin (`n,b`), as the wiki specifies. Produces a playable
   "X-Wing Alliance Upgrade 2025" entry.

4. **`scripts/03-tftc.yml`** — identical to step 3, but for your TFTC
   folder. TFTC is confirmed to reuse the same `XWAU2025_Linux.zip` DLL
   wrapper as XWAU2025 (no separate TFTC-specific patch exists).

If you hit a blank screen on first launch of either game, use the game's
"Play (Skip Intro)" option once to generate pilot data, per the wiki.

## Design notes

- Manually, the simplest path to a working setup is duplicating one Lutris
  entry and retargeting the copies, which shares one physical prefix across
  all of them (no need to reinstall fonts/DPI/etc. three times). Lutris's
  installer YAML format has no "duplicate an entry" primitive, and file
  pickers can only select a **file**, not a folder — so a script can't be
  handed a prefix *directory* directly.
- These scripts get the same practical outcome — a real duplicate, not just
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
- If you point a later script's install directory at the *same* folder you
  used for `01-xwainstallermanager.yml` (i.e. you're deliberately sharing
  one prefix across every entry, rather than giving each its own), the
  clone step detects that the selected `system.reg` already lives in
  `$GAMEDIR` and skips the copy instead of erroring — `cp` refuses to copy
  a directory onto itself.
- The XWAU2025/TFTC folders themselves live wherever XwaInstallerManager
  put them — they don't need to be inside the Wine prefix Lutris manages
  for that entry. Wine can run an executable from any path regardless of
  which prefix is active.

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
