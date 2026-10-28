---
name: log-noise-sample
description: Log a noise recording into the workspace - copy the audio, capture device, app, room, mic position and conditions; give recording tips.
---

# Log a noise sample

Turns an audio file the user recorded into a sample folder with full recording metadata,
ready for `spectral-analysis`. Also the place to coach the user on how to record well.

## Inputs

- **Recording path(s)** the user gives. One nuisance take, optionally one background
  take (same spot, same settings, nuisance absent).
- **Workspace.** The path the user names, else `workspace` from
  `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/noise-pollution-analysis/config.json`.
  If neither exists, run `setup-noise-workspace` first.
- **Which noise source** it is (a slug under `sources/`; create one if new).

If the user hasn't recorded yet and asks how, give them
`${CLAUDE_PLUGIN_ROOT}/references/recording-guide.md` (device/app settings and
protocol), adapted to their device, and stop.

## Steps

1. Inspect each file with `ffprobe -v error -show_entries format=duration,format_name:stream=codec_name,sample_rate,channels,bits_per_sample -of json "<file>"`.
   Flag: lossy codec (AAC/MP3/Opus), sample rate under 44.1 kHz, duration under 30 s.
   Use the file's creation time or the user's word for `recorded_at`.
2. Ask only for the metadata you can't read from the file or `devices.md`: device and
   app, audio source setting (unprocessed / mic / voice recognition / unknown), room and
   mic position, windows and doors, own appliances turned off, anything heard during
   the take. Reuse answers from `devices.md` and earlier samples rather than re-asking.
3. Create `samples/<YYYY-MM-DD-HHMM>_<source-slug>/`, **copy** (never move) the audio
   in, and write `sample.md` per `${CLAUDE_PLUGIN_ROOT}/references/workspace-layout.md`
   with `analysed: false`.
4. If the device or app is new, add it to `devices.md`.
5. If the take was made with a processed audio source (default mic mode, voice memo,
   call app), say once that noise suppression may have removed the steady part of the
   noise and suggest a re-take with an unprocessed source. Still log it.
6. If no background take exists and the nuisance is intermittent, suggest recording
   one; it makes the analysis much more specific.
7. Offer to run `spectral-analysis` on the sample now.

## Data storage

Writes only inside the user's workspace (`samples/`, `devices.md`, `sources/`). Original
files are copied; the user's originals are left where they were.
