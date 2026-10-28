---
name: setup-noise-workspace
description: Set up a workspace to document home noise pollution - describe each noise source, log devices, store recordings and reports.
---

# Set up a noise workspace

Creates the folder that every other skill in this plugin works in: one place for noise
source descriptions, recordings, analysis output and reports.

## Inputs

- **Workspace path.** If the current directory is empty or the user says "here" / "this
  one", use it. Otherwise ask, suggesting `~/Documents/noise-log/`.
- **Noise sources** the user already knows about (optional now; can be added later).
- **Location context** (optional): country (sets 50 vs 60 Hz mains), building type,
  floor, which rooms are affected.

## Steps

1. Read `${CLAUDE_PLUGIN_ROOT}/references/workspace-layout.md` for the layout and file
   templates.
2. If the path already contains `WORKSPACE.md`, it is already a workspace: report what is
   in it (sources, sample count, latest report) and stop.
3. Create `WORKSPACE.md`, `devices.md`, `sources/`, `samples/`, `reports/`.
   `WORKSPACE.md` holds: purpose, home context (building type, floor, country and mains
   frequency, affected rooms), and a `## Sources` index that later skills keep current.
4. For each noise the user describes, interview briefly and write
   `sources/<slug>/source.md` from the template: what it sounds like, when it happens and
   how it cycles, where it is heard and where it is worst, doors/windows effect,
   suspected source and direction, impact. Write the user's words; don't embellish.
5. Ask what they will record with. Add a section to `devices.md` per device. If it is a
   phone, give the device-and-app part of `${CLAUDE_PLUGIN_ROOT}/references/recording-guide.md`
   (on Android, ASR Voice Recorder with the audio source set to Unprocessed, WAV, 48 kHz).
6. If the folder is not a git repo, offer `git init` (useful as a dated evidence trail).
7. Save the workspace path as a pointer:
   `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/noise-pollution-analysis/config.json`
   with `{"workspace": "<absolute path>"}`. Other skills read it when the user doesn't
   name a workspace.
8. Tell the user the path and the next step: record a sample (`log-noise-sample`).

## Data storage

User-owned content (descriptions, recordings, reports) lives only in the workspace the
user chose. The plugin stores only `config.json` (the workspace pointer) under
`${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/noise-pollution-analysis/`.
