# Working in this repo

This repo ships Lutris installer scripts (`scripts/*.yml`) for the XWAU2025
and TFTC mods for X-Wing Alliance. The intended audience for the shipped
product is a non-technical Linux gamer following steps in Lutris — keep
that audience in mind for every file a user might open.

## File layout and audience

- **`README.md`** — the only file a typical user should ever need to read.
  Numbered, procedural, no Lutris/Wine internals (prefixes, `$GAMEDIR`,
  registry files, YAML task names). If a step requires jargon to follow,
  rephrase it or push the explanation into `docs/`.
- **`docs/DESIGN-NOTES.md`** — the technical "why": root causes, design
  tradeoffs, unverified assumptions, maintenance instructions (e.g. bumping
  pinned release URLs), provenance. Written for a contributor or maintainer,
  not a first-time installer. The README links here for anyone who wants
  it; nothing in `docs/` should be required reading to complete an install.
- **`scripts/`** — the actual Lutris installer YAMLs. These are the product.
  Inline comments in the YAML should explain non-obvious choices for the
  next maintainer (as they already do), not narrate what each task does.
- **`dev/`** (create if/when needed) — debugging or recovery scripts and any
  working log from an unattended/autonomous session (see `HANDOFF.md`
  pattern below). Not part of the installer users run; keep this out of the
  README entirely.

This split is modeled on
[darkview224/lutris-mtgo-script](https://github.com/darkview224/lutris-mtgo-script):
README = plain install steps only, `docs/DESIGN-NOTES.md` = everything
technical, `dev/` = maintainer/debugging tooling most users never see.

## Editing rules

- Never move technical rationale, troubleshooting internals, or maintainer
  notes into `README.md`. If you're tempted to explain *why* something
  works in the README, that explanation belongs in
  `docs/DESIGN-NOTES.md` with a link from the README instead.
- Keep the three `scripts/*.yml` files consistent with each other — they
  share patterns (the `system.reg` prefix-clone trick, `wine: overrides:`
  for DLL settings). A fix or convention change in one usually needs to be
  mirrored in the others; check all three before considering a change done.
- If you pin a new release URL (see `docs/DESIGN-NOTES.md#maintenance`),
  update it in every script that references it and note the change in that
  file if the maintenance procedure itself changes.
- Don't write documentation files unless they serve one of the two
  audiences above (end user or maintainer) — no scratch notes, no
  duplicate summaries of what a commit already says.

## Autonomous/unattended sessions

If you're running unsupervised (e.g. via a scheduled trigger with no human
watching), work in small committed, pushed increments so progress survives
a crash or usage-limit reset. If you create a working log for this, put it
at `dev/HANDOFF.md`: state the goal, what's confirmed, what's ruled out,
and next steps, with the most recent status at the top — so a future
session (or the user) can resume from `git log` plus that one file. Fold
any confirmed fix back into the actual `scripts/*.yml` files before
declaring success; a fix that only exists as a manual workaround or a
separate throwaway script isn't done.

## Git workflow

This repo has no PR review gate on `main` in normal use — verify with the
user before assuming that for any specific push. Commit with clear,
descriptive messages; don't batch unrelated changes together.
