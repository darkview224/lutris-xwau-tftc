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
`XwaInstallerManager.ico` icon file. We checked whether the installer
script itself could wire that up as the entry's library icon: it can't —
[Lutris's installer YAML
spec](https://github.com/lutris/lutris/blob/master/docs/installers.rst)
has no `icon:`/`banner:` directive or any other documented way for a
script to set game art; that's strictly a per-user, Lutris-client-side
thing. The extracted `.ico` is left in place at a predictable path and
called out in the README so setting it by hand is a two-second job instead
of a file hunt.

The install deliberately does **not** auto-launch the manager as part of
the install step, unlike the old Wine-based version's `wineexec` task: an
install step that launches a long-running GUI process and waits for it to
exit is a known trap (see
[darkview224/lutris-mtgo-script](https://github.com/darkview224/lutris-mtgo-script)'s
design notes on why MTGO's client deploy was moved out of its install
step for the same reason). Click **Play** after the script finishes
instead.

`01-xwa-mod-manager.yml` asks for your vanilla XWA exe and, after
extracting, moves the manager into that prefix's `drive_c` (leaving a
symlink at the Lutris-managed path so the entry still launches). This
isn't a technical requirement — the manager is native, it doesn't touch
any prefix — it's purely so the manager's files sit somewhere findable
when you go looking for them later. See the next section for the same
`drive_c`-walk trick used for a real purpose in `02`/`03`.

## Why XWAU2025 and TFTC install themselves instead of depending on the manager

Earlier versions of `02-xwau2025.yml`/`03-tftc.yml` assumed you'd already
run XwaInstallerManager to build an XWAU2025/TFTC folder, and just pointed
a Lutris entry at whatever it produced. That meant `02`/`03` depended on
`01` having been run first, and `01` had to maintain a font-seeded prefix
purely so `02`/`03` could clone it.

We reconsidered this after establishing that XwaInstallerManager is
itself just a download manager, file copier, and archive extractor — no
installer logic beyond that. If that's true, there's nothing `02`/`03`
actually need the manager *for*: they can download/apply the same pieces
themselves. So now:

- `02-xwau2025.yml` depends only on vanilla XWA. It asks you to have
  already downloaded `XWAU2025_Full_1.0.0.zip` and
  `XWAU2025_UPD_1.1.0.zip` from xwaupgrade.com yourself (that part still
  can't be scripted — see the top of the README), then builds a complete
  XWAU2025 copy itself: clone vanilla's prefix, copy vanilla's game files
  into a new `XWAU2025` folder, layer `Full` then `UPD` on top, then
  morallo's `XWAU2025_Linux.zip` Linux patch last (its `Alliance.EXE` is
  the one actually launched).
- `03-tftc.yml` depends on `02` instead of `01` — the wiki's own procedure
  installs TFTC on top of an XWAU2025 install, not vanilla, so that's
  what it clones and copies from.
- `01-xwa-mod-manager.yml` becomes a leaf with no dependents: useful for
  checking for and applying future updates through its own GUI, not
  required for anything in `02`/`03`.

This makes the `drive_c`-walk prefix derivation (see below) load-bearing
instead of cosmetic, which is why it's fatal (`exit 1`) in `02`/`03` if it
can't find one, unlike the best-effort version in `01`.

One consequence worth calling out: because `02`/`03` build their own exe
rather than pointing at a pre-existing one, they can't use a file-picker
for `game.exe` — Lutris shows all its file-picker prompts before any
install step runs, so a script can't ask you to pick a file it hasn't
created yet. `game.exe` is hardcoded (`.../XWAU2025/Alliance.EXE`,
`.../TFTC/Alliance.EXE`) instead, relying on the last-applied archive
(morallo's Linux patch) to be the one that determines the actual filename
on disk — see "Things to verify" for the one loose end this leaves.

### The `drive_c`-walk prefix derivation

Every Wine prefix has a fixed `drive_c/` at its root, so given any file
inside one, walking up to the enclosing `drive_c` and taking its parent
finds the prefix root — no second `system.reg` picker needed. This only
became reliable once `02`/`03` pointed it at the *vanilla* XWA exe (or, in
`03`'s case, the XWAU2025 exe `02` built): a normal Lutris `wine` entry is
guaranteed to live inside a real prefix. An earlier version of this idea,
applied to wherever XwaInstallerManager's own target-folder picker put
things, was dropped for exactly that reason — that folder isn't
guaranteed to be inside any prefix at all.

## Adjusting for a different vanilla XWA entry

`scripts/02-xwau2025.yml` declares `requires: star-wars-x-wing-alliance` —
the public lutris.net installer slug for vanilla X-Wing Alliance (GOG).
If your own vanilla install used a different Lutris entry, edit the
`requires:` line in that script (and in `01-xwa-mod-manager.yml`, if you
use it) to match its `game_slug`.

## Things to verify on your first real run-through

These are documented assumptions/best guesses that couldn't be confirmed
without actually running the tools end-to-end in Lutris:

- Whether `XWAU2025_Full_1.0.0.zip` and `XWAU2025_UPD_1.1.0.zip` extract
  as flat file sets (assumed, matching `XWAU2025_Linux.zip`'s confirmed
  layout) — adjust the `cp` source paths in `02-xwau2025.yml` if either
  turns out to be wrapped in a top-level folder.
- Whether copying vanilla's game files first and layering the mod zips on
  top is actually the right model, versus `XWAU2025_Full_1.0.0.zip` being
  self-sufficient on its own. This follows the same pattern
  [psoetens/xwau-linux](https://github.com/psoetens/xwau-linux) uses
  ("installs XWAU 2025 into the game dir"), which is reasonable secondhand
  evidence but not a direct confirmation of what's inside the official
  zips.
- Whether Lutris/Wine's launch path resolves `game.exe:
  .../Alliance.EXE` case-insensitively if the actual extracted filename
  ends up differently cased on your filesystem. Wine's own file lookups
  are normally case-insensitive; whether Lutris's own pre-launch existence
  check is too hasn't been confirmed.
- TFTC's actual distribution: exact filename(s), whether it's really one
  zip, and its exe name/layout (assumed `Alliance.EXE`, same as XWAU2025,
  since TFTC runs on the same engine). `03-tftc.yml`'s prompt text is
  intentionally generic pending confirmation.
- Whether XwaInstallerManager's target-directory picker, running as a
  native Linux binary, browses the filesystem the way users expect when
  pointed at a path inside a Wine prefix's `drive_c` (relevant only if you
  use `01-xwa-mod-manager.yml` for updates).

Already confirmed by downloading and inspecting the release archives
directly (not just assumed, as the old scripts had to):
`XwaInstallerManager-linux_x86_20260824.zip` contains a native ELF binary
named exactly `XwaInstallerManager`; `XWAU2025_Linux.zip` extracts as a
flat file set (plus one `XwaInstallerRunner/` subfolder, harmless to copy
alongside the rest) and contains `Alliance.EXE`.

If you find any of the still-unverified items wrong, please fix the
corresponding script and open a PR — that's exactly what community
testing is for.

## Maintenance

The download URLs for `XwaInstallerManager-linux_x86_20260824.zip` and
`XWAU2025_Linux.zip` are pinned to a specific dated release tag
(`20260808_linux`) from morallo's repo. As builds are updated, check
https://github.com/morallo/xwa_ddraw_d3d11/releases for newer releases and
bump the `files:` URLs across whichever scripts reference them
accordingly. `XWAU2025_Full_1.0.0.zip`/`XWAU2025_UPD_1.1.0.zip`/TFTC's own
zip are never pinned URLs — they're always a manual download from
xwaupgrade.com or wherever TFTC is currently hosted, picked via a
file-picker prompt, since that content can't be redistributed.

## Relationship to `archive/`

`archive/` holds the previous, Wine-based version of this same workflow
(`create_prefix` + `wineexec` to run XwaInstallerManager's Windows build
under Wine, and depending on it to build XWAU2025/TFTC rather than doing
that itself), kept for reference and as a fallback in case the
self-installing approach in this document doesn't pan out. It's frozen —
not maintained in parallel with `scripts/`. If the approach in this
document doesn't pan out, `archive/README.md` documents the older,
more-tested-in-spirit path.

## Provenance

These installer scripts and this documentation were produced with
[Claude Code](https://claude.com/claude-code).
