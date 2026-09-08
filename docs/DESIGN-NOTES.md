# Design notes

This is the *why* behind the scripts in `scripts/`. The [README](../README.md)
intentionally doesn't explain any of this — it's just install steps. If
you're troubleshooting, curious, or maintaining this repo, read on.

## Why the manager runs without Wine

`01-xwa-mod-manager.yml` downloads
`XwaInstallerManager-linux_x86_20260824.zip`, a native Linux build of
XwaInstallerManager (a statically-linked x86-64 ELF binary, confirmed by
inspecting the archive directly). Earlier versions of these scripts had to
`create_prefix` + `wineexec` the Windows build of this same tool, which
meant running a GUI download manager through Wine's translation layer for
no benefit — the tool itself doesn't touch DirectX/DirectSound or anything
else Wine-sensitive, it just downloads and copies files. The native build
removes that layer entirely: the Lutris entry it produces uses `runner:
linux` and launches the binary directly.

This also resolved one of the old script's "unverified" guesses: the
Windows build's executable name was assumed to be
`XwaInstallerManager.exe`; the Linux build's binary is confirmed to be
named exactly `XwaInstallerManager` (no extension), alongside an
`XwaInstallerManager.ico` icon file that isn't currently wired into
anything — Lutris banner/icon art is normally set by hand after install,
and scripting that wasn't worth the added complexity.

`01-xwa-mod-manager.yml` still creates a Wine prefix and runs winetricks
against it (`corefonts consolas`), even though the manager itself never
uses that prefix. It exists purely as a font-seeded prefix for
`02-xwau2025.yml`/`03-tftc.yml` to clone — see the next section. The
install deliberately does **not** auto-launch the manager as part of the
install step, unlike the old Wine-based version's `wineexec` task: an
install step that launches a long-running GUI process and waits for it to
exit is a known trap (see
[darkview224/lutris-mtgo-script](https://github.com/darkview224/lutris-mtgo-script)'s
design notes on why MTGO's client deploy was moved out of its install
step for the same reason). Click **Play** after the script finishes
instead.

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
you to select `system.reg` inside the prefix `01-xwa-mod-manager.yml`
built, then run `cp -a "$(dirname <selected system.reg>)/." "$GAMEDIR/"`
as their first install step. That physically clones the whole prefix —
fonts, DPI, Gecko/Mono, everything — before the rest of the script runs,
so only `01-xwa-mod-manager.yml` needs its own `create_prefix` +
`winetricks`. Scripts 2 and 3 inherit whatever prefix state actually
worked when you tested script 1, rather than re-running winetricks and
hoping it produces an equivalent result.

If you point a later script's install directory at the *same* folder you
used for `01-xwa-mod-manager.yml` (i.e. you're deliberately sharing one
prefix across every entry, rather than giving each its own), the clone
step detects that the selected `system.reg` already lives in `$GAMEDIR`
and skips the copy instead of erroring — `cp` refuses to copy a directory
onto itself.

The XWAU2025/TFTC folders themselves live wherever XwaInstallerManager
put them — they don't need to be inside the Wine prefix Lutris manages
for that entry. Wine can run an executable from any path regardless of
which prefix is active.

We considered deriving the prefix path automatically from the selected
`Alliance.exe` (e.g. walking up to the enclosing `drive_c` folder) instead
of asking for a second `system.reg` picker, and even a shared script with
an `input_menu` dropdown to pick XWAU2025 vs. TFTC and set the resulting
entry's name/icon dynamically. Both were dropped: the derivation only
works if the mod folder happens to live inside a Wine prefix (not
guaranteed — XwaInstallerManager lets you target any folder), and dynamic
`name:`/`game_slug:` from a runtime menu choice isn't a Lutris installer
YAML feature we could confirm exists. Two explicit, static scripts
(matching the existing `02`/`03` pattern) are more reliable than one
script betting on unverified behavior.

## Adjusting for a different vanilla XWA entry

`scripts/01-xwa-mod-manager.yml` declares `requires:
star-wars-x-wing-alliance` — the public lutris.net installer slug for
vanilla X-Wing Alliance (GOG). If your own vanilla install used a
different Lutris entry, edit the `requires:` line in that script to match
its `game_slug`.

## Things to verify on your first real run-through

These are documented assumptions/best guesses that couldn't be confirmed
without actually running the tools end-to-end in Lutris:

- The exact winetricks verb set for the recommended fonts (currently
  `corefonts consolas` — `corefonts` includes Arial and Verdana). This only
  needs to be right in `01-xwa-mod-manager.yml`; scripts 2 and 3 clone its
  prefix rather than re-running winetricks.
- TFTC's actual executable name and folder layout (assumed `Alliance.exe`,
  same as XWAU2025, since TFTC runs on the same XWA engine).
- Whether XwaInstallerManager's target-directory picker, running as a
  native Linux binary, browses the filesystem the way users expect when
  pointed at a path inside a Wine prefix's `drive_c`.

Already confirmed by downloading and inspecting the release archives
directly (not just assumed, as the old scripts had to):
`XwaInstallerManager-linux_x86_20260824.zip` contains a native ELF binary
named exactly `XwaInstallerManager`; `XWAU2025_Linux.zip` extracts as a
flat file set (plus one `XwaInstallerRunner/` subfolder, harmless to copy
alongside the rest).

If you find any of the still-unverified items wrong, please fix the
corresponding script and open a PR — that's exactly what community
testing is for.

## Maintenance

The download URLs for `XwaInstallerManager-linux_x86_20260824.zip` and
`XWAU2025_Linux.zip` are pinned to a specific dated release tag
(`20260808_linux`) from morallo's repo. As builds are updated, check
https://github.com/morallo/xwa_ddraw_d3d11/releases for newer releases and
bump the two `files:` URLs across all three scripts accordingly.

## Relationship to `archive/`

`archive/` holds the previous, Wine-based version of this same workflow
(`create_prefix` + `wineexec` to run XwaInstallerManager's Windows build
under Wine), kept for reference and as a fallback in case the native
Linux build turns out not to work for someone. It's frozen — not
maintained in parallel with `scripts/`. If the native-build approach in
this document doesn't pan out, `archive/README.md` documents the older,
more-tested-in-spirit path.

## Provenance

These installer scripts and this documentation were produced with
[Claude Code](https://claude.com/claude-code).
