# Lutris scripts for XWAU2025 and TFTC

Lutris installer scripts that automate most of the Linux setup process for
the **XWAU2025** (X-Wing Alliance Upgrade) and **TFTC** (TIE Fighter Total
Conversion) mods for *Star Wars: X-Wing Alliance*, based on the manual
procedure documented at
https://github.com/morallo/xwa_ddraw_d3d11/wiki/XWAU-and-TFTC-in-Linux.

Everything that can safely be scripted is automated. The one part left
manual is downloading the mod content itself — that content is copyrighted
and can't be redistributed or fetched headlessly, so you'll grab a couple
of zip files by hand and point the scripts at them.

## Prerequisites

- Lutris, Wine, and winetricks installed on your system.
- **Vanilla X-Wing Alliance (GOG)** already installed via a Lutris entry.
  These scripts assume the public lutris.net slug for that entry
  (`star-wars-x-wing-alliance`). If your local install used a different
  name, see [docs/DESIGN-NOTES.md](docs/DESIGN-NOTES.md#adjusting-for-a-different-vanilla-xwa-entry).
- A modern Wine build (wine-ge, GE-Proton, or a Wine 11-class build) —
  needed for HD cutscene playback. Select this as the runner version for
  the XWAU2025/TFTC entries once they're installed.
- **The mod content itself, downloaded ahead of time:**
  - `XWAU2025_Full_1.0.0.zip` and `XWAU2025_UPD_1.1.0.zip` from
    [xwaupgrade.com](https://www.xwaupgrade.com/) — needed for step 2
    below.
  - TFTC's own installation zip, from wherever it's currently distributed
    — needed for step 3, only if you want TFTC.
- Internet access, to download one public release archive (and, if you
  use it, the mod manager tool) from
  [morallo/xwa_ddraw_d3d11](https://github.com/morallo/xwa_ddraw_d3d11/releases).

## Get this repo's scripts

You need the three files in `scripts/`. Easiest way: right-click each link
below and choose "Save Link As" (or similar) to download it.

- [scripts/01-xwa-mod-manager.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/01-xwa-mod-manager.yml)
- [scripts/02-xwau2025.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/02-xwau2025.yml)
- [scripts/03-tftc.yml](https://raw.githubusercontent.com/darkview224/lutris-xwau-tftc/main/scripts/03-tftc.yml)

Or, if you're comfortable with git:

```
git clone https://github.com/darkview224/lutris-xwau-tftc.git
```

which gives you `lutris-xwau-tftc/scripts/`.

## Installation steps

In Lutris: **+** (top left) → **Install script** (Flatpak Lutris calls
this "Install game from a local file") → browse to the script →
**Install**.

### 1. (Optional) Install the Mod Installer Manager

Run `scripts/01-xwa-mod-manager.yml` if you'd like a way to check for and
apply future mod updates later. It downloads and unpacks the native Linux
build of `XwaInstallerManager` and places it inside your vanilla XWA
prefix, so it's easy to find again. When it's done, you'll have a
permanent "X-Wing Alliance: Mod Installer Manager" entry in your Lutris
library. Click **Play** to launch it; it runs directly, with no Wine
translation layer involved.

You don't need this step to install XWAU2025 or TFTC — steps 2 and 3
below do that themselves. This entry is only for updating later.

The download includes an icon
(`XwaInstallerManager/XwaInstallerManager.ico` inside this entry's game
folder) if you want to set it as the entry's icon by hand — Lutris install
scripts can't do that part automatically.

### 2. Install XWAU2025

Run `scripts/02-xwau2025.yml`. When prompted:

- **Select `Alliance.exe`** (or `XWINGALLIANCE.EXE`) inside your existing
  vanilla X-Wing Alliance install.
- **Select `XWAU2025_Full_1.0.0.zip`**, the file you downloaded from
  xwaupgrade.com in the Prerequisites step above.
- **Select `XWAU2025_UPD_1.1.0.zip`**, downloaded the same way.

The script builds a complete, separate XWAU2025 copy of the game and
applies the Linux compatibility patch on top. This produces a playable
**X-Wing Alliance Upgrade 2025** entry — no manual mod-manager step
needed.

### 3. Install TFTC

Only if you want TFTC — it installs on top of the XWAU2025 entry from
step 2, so do that one first. Run `scripts/03-tftc.yml`. When prompted:

- **Select `Alliance.EXE`** inside the XWAU2025 entry you just built.
- **Select the TFTC zip** you downloaded.

### First launch

If either game shows a blank screen the first time you click **Play**,
use that game's **Play (Skip Intro)** option once to generate pilot data,
then launch normally after that.

---

Something not working as described above, or want to know why the scripts
are built this way? See [docs/DESIGN-NOTES.md](docs/DESIGN-NOTES.md).

An earlier, Wine-based version of these scripts (before a native Linux
build of XwaInstallerManager existed, and before XWAU2025/TFTC installed
themselves) is kept in [`archive/`](archive/) for reference — not needed
for a normal install.
