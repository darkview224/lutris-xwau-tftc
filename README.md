# Lutris scripts for XWAU2025 and TFTC

Lutris installer scripts that automate most of the Linux setup process for
the **XWAU2025** (X-Wing Alliance Upgrade) and **TFTC** (TIE Fighter Total
Conversion) mods for *Star Wars: X-Wing Alliance*, based on the manual
procedure documented at
https://github.com/morallo/xwa_ddraw_d3d11/wiki/XWAU-and-TFTC-in-Linux.

Everything that can safely be scripted is automated. The one part left
manual is downloading the mod content itself through a small GUI tool —
that content is copyrighted and can't be redistributed or fetched
headlessly.

## Prerequisites

- Lutris, Wine, and winetricks installed on your system.
- **Vanilla X-Wing Alliance (GOG)** already installed via a Lutris entry.
  These scripts assume the public lutris.net slug for that entry
  (`star-wars-x-wing-alliance`). If your local install used a different
  name, see [docs/DESIGN-NOTES.md](docs/DESIGN-NOTES.md#adjusting-for-a-different-vanilla-xwa-entry).
- A modern Wine build for the final two entries (wine-ge, GE-Proton, or a
  Wine 11-class build) — needed for HD cutscene playback. Select this as
  the runner version for the XWAU2025/TFTC entries once they're installed.
- Internet access, to download two public release archives from
  [morallo/xwa_ddraw_d3d11](https://github.com/morallo/xwa_ddraw_d3d11/releases).

## Get this repo's scripts

You need the three files in `scripts/`. Easiest way: right-click each link
below and choose "Save Link As" (or similar) to download it.

- [scripts/01-xwainstallermanager.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/01-xwainstallermanager.yml)
- [scripts/02-xwau2025.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/02-xwau2025.yml)
- [scripts/03-tftc.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/03-tftc.yml)

Or, if you're comfortable with git:

```
git clone https://github.com/darkview224/lutris-xwau-tftc.git
```

## Installation steps

Run these in order. In Lutris: **+** (top left) → **Install script**
(Flatpak Lutris calls this "Install game from a local file") → browse to
the script → **Install**.

### 1. Install the Mod Installer Manager

Run `scripts/01-xwainstallermanager.yml`. It creates a dedicated Wine
prefix, installs the fonts XWAU recommends, downloads the
`XwaInstallerManager` tool, and opens it. When it's done, you'll have a
permanent "X-Wing Alliance: Mod Installer Manager" entry in your Lutris
library — this is how you reopen the tool later.

### 2. Install the mods (manual step, no script)

In the XwaInstallerManager window that just opened:

1. Point **Vanilla Location** at your existing X-Wing Alliance install.
2. Pick or create a target folder, then install **XWAU2025 1.0.0** plus
   the **1.1.0 update**.
3. Close the manager, reopen the same Lutris entry, and repeat with a
   *different* target folder to install **TFTC**.

This step downloads mod content from xwaupgrade.com / TFTC's own hosting
through the tool's own interface — it can't be scripted.

### 3. Install XWAU2025

Run `scripts/02-xwau2025.yml`. When prompted:

- **Select `Alliance.exe`** inside the XWAU2025 folder you created in
  step 2.
- **Select `system.reg`** inside the prefix from step 1. In Lutris,
  right-click that "Mod Installer Manager" entry → **Configure** →
  **Advanced** → note the Wine prefix path, then browse to `system.reg`
  inside it.

This downloads the Linux compatibility patch and finishes setting up a
playable **X-Wing Alliance Upgrade 2025** entry.

### 4. Install TFTC

Run `scripts/03-tftc.yml`. Same two prompts as step 3, but pointed at your
TFTC folder instead.

### First launch

If either game shows a blank screen the first time you click **Play**,
use that game's **Play (Skip Intro)** option once to generate pilot data,
then launch normally after that.

---

Something not working as described above, or want to know why the scripts
are built this way? See [docs/DESIGN-NOTES.md](docs/DESIGN-NOTES.md).
