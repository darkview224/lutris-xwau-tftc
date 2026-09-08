# Handoff

## 2026-09-08 — Pre-run brief (not started yet)

**Goal:** verify and fix `scripts/*.yml` end-to-end against a real Lutris +
vanilla XWA install, using mod content staged on disk ahead of time.
Confirm or correct every item currently listed under
[docs/DESIGN-NOTES.md#things-to-verify-on-your-first-real-run-through](../docs/DESIGN-NOTES.md#things-to-verify-on-your-first-real-run-through),
folding fixes directly into the scripts as you go — per `CLAUDE.md`, a fix
that only exists as a note here isn't done.

**Before starting, confirm these are actually true; if not, stop and ask
rather than guessing or trying to route around it:**

- Lutris is installed, and vanilla X-Wing Alliance (GOG) is already
  installed as a Lutris entry.
- `XWAU2025_Full_1.0.0.zip`, `XWAU2025_UPD_1.1.0.zip`, and TFTC's own
  installation zip are already downloaded and sitting on disk somewhere.
  This session should never attempt to fetch them itself — that content
  is copyrighted and the whole reason these are manual `files:` prompts
  in the first place.
- Internet access for the two pinned morallo release downloads
  (`XwaInstallerManager-linux_x86_20260824.zip`,
  `XWAU2025_Linux.zip`).

**Already confirmed — don't re-verify these:**

- `XwaInstallerManager-linux_x86_20260824.zip` is a native ELF binary,
  filename exactly `XwaInstallerManager` (no extension).
- `XWAU2025_Linux.zip` extracts flat (plus one harmless
  `XwaInstallerRunner/` subfolder) and contains `Alliance.EXE`.
- Lutris's installer YAML format has no `icon:`/`banner:` directive
  (checked directly against `lutris/lutris`'s own docs).

**Not yet confirmed — work through in roughly this order:**

1. `scripts/01-xwa-mod-manager.yml` — does it install, launch, and place
   itself inside the vanilla prefix's `drive_c` as designed?
2. `scripts/02-xwau2025.yml`:
   - Do `XWAU2025_Full_1.0.0.zip`/`XWAU2025_UPD_1.1.0.zip` extract flat or
     wrapped in a top-level folder? Fix the `cp` source paths if wrapped.
   - Is "copy vanilla's files, then overlay the mod zips on top" actually
     the right model, or is `Full_1.0.0.zip` self-sufficient on its own?
   - Does the `drive_c`-walk prefix derivation actually find the right
     prefix against a real GOG install layout?
   - Does `game.exe: .../XWAU2025/Alliance.EXE` actually launch (the
     open question is filesystem case-sensitivity)?
3. `scripts/03-tftc.yml` — same categories of check, plus: what does
   TFTC's distribution actually look like (exact filename(s), is it
   really one zip, exe name/layout)? Update the script's file-picker
   prompt text from its current generic wording to something specific
   once you know.
4. A real smoke test of both games: does XWAU2025 reach a working state
   (per the README's "blank screen → Play Skip Intro once" workaround)?
   Does TFTC?

**Working method:**

- Test one script at a time. Fix `scripts/*.yml` directly when something's
  wrong — don't just log the problem here.
- As each "Things to verify" item gets confirmed, move it out of that list
  in `docs/DESIGN-NOTES.md` into the "Already confirmed" paragraph there
  (matching that file's existing style), or correct the design note if
  reality differs from what's documented.
- Update `README.md` if real behavior differs from what it currently
  tells users (e.g. TFTC's actual download filename, once known).
- Commit each confirmed fix separately, with a clear message, in small
  increments — progress needs to survive a crash or a usage-limit reset.
- Before stopping, whether finished or not, add a new dated entry **above
  this one** (don't edit this one away) summarizing what's confirmed,
  what's still open, and anything you tried that didn't work.

**Out of scope:**

- `archive/` is frozen — don't touch it (see `CLAUDE.md`).
- Never invent or guess a download URL for anything not already pinned in
  the scripts.
- Never attempt to fetch XWAU2025/TFTC content from xwaupgrade.com (or
  anywhere else) yourself — it must already be staged locally.
