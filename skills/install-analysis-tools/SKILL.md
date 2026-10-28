---
name: install-analysis-tools
description: Check and install the CLI toolchain for noise analysis - ffmpeg, sox, aubio, uv - on Linux, macOS or Windows; verify each works.
---

# Install the noise-analysis toolchain

Every analysis skill in this plugin calls these tools. Run this first on a new machine,
or whenever a skill reports a missing tool.

## Steps

1. Read `${CLAUDE_PLUGIN_ROOT}/references/toolchain.md` for the tool list, roles and
   per-OS install names.
2. Detect the OS and package manager (`apt`, `dnf`, `pacman`, `brew`, `winget`).
3. Check each tool and record its version:
   ```bash
   ffmpeg -version | head -1
   sox --version
   aubio -V
   uv --version
   ```
4. List what is missing, split into **core** (ffmpeg, sox, uv) and **recommended**
   (aubio). Install with the system package manager after one confirmation, since it
   needs elevated rights: e.g. `sudo apt-get install -y ffmpeg sox aubio-tools`. Install
   `uv` with its official installer if no package is available.
5. Smoke-test on a generated signal, so the check exercises the actual commands:
   ```bash
   T=$(mktemp -d)
   ffmpeg -v error -f lavfi -i "sine=f=100:d=3:r=48000" "$T/t.wav"
   sox "$T/t.wav" -n remix - rate 2k spectrogram -o "$T/s.png" && echo sox-ok
   aubio pitch -i "$T/t.wav" -m yin -u Hz -B 4096 -H 1024 | awk '$2>95 && $2<105' | head -1
   uv run "${CLAUDE_PLUGIN_ROOT}/scripts/analyse_noise.py" "$T/t.wav" --out "$T/a" >/dev/null && echo script-ok
   ```
   The aubio line should print a pitch near 100 Hz. The script needs at least 1 s of audio.
6. Mention the optional interactive tools (Friture for live spectrum while the noise
   is happening, REW for a calibrated measurement mic) once. Don't install them unasked.
7. Report a table: tool, version, status (ok / installed now / missing and why).

## Notes

- On macOS, Homebrew has `sox_ng` (maintained fork) and `sox` (original, unmaintained).
  Prefer `sox_ng`, then check which binary name it installs with `brew list sox_ng`.
- If aubio can't be installed, the analysis still runs; impact timing and whine pitch
  tracking are skipped and the report says so.
- Never install with `pip` into the system Python; the analysis script runs under `uv`.
