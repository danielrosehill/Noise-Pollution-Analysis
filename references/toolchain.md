# Analysis toolchain

The plugin drives established command-line tools rather than reimplementing them.
Versions and commands below were checked on 2026-10-29 (Ubuntu 26.04, ffmpeg 8.0.1,
SoX_ng 14.7.0.9, aubio 0.4.9) against synthetic test recordings with known content:
a 50/100/150 Hz hum cycling every 1.5 s, a 3 kHz whine, and 80 Hz thumps every 2 s.

## Core (required)

Apt and Homebrew names were checked; winget IDs were not.

| Tool | Role in this plugin | Install |
|---|---|---|
| **ffmpeg** | Decode any format; full-range log-frequency spectrogram; format probe | apt `ffmpeg` · brew `ffmpeg` · winget `Gyan.FFmpeg` |
| **sox** (SoX_ng) | Low-frequency zoom spectrograms; **noise profile + reduction against a background take** (isolation); band-pass to listen to one band | apt `sox` (26.04 ships the maintained SoX_ng fork) · brew `sox_ng` (maintained fork, 14.8.x) or `sox` (original 14.4.2, unmaintained since 2015) · winget `ChrisBagwell.SoX` |
| **uv** | Runs `scripts/analyse_noise.py` (numpy/scipy) in an isolated env | `curl -LsSf https://astral.sh/uv/install.sh \| sh` · brew `uv` · winget `astral-sh.uv` |

## Recommended

| Tool | Role | Install |
|---|---|---|
| **aubio** (`aubio` CLI) | Impact/thump timestamps (`onset -m specflux`); pitch tracking of a hum or whine over time (`pitch -m yin -B 4096`) | apt `aubio-tools` · brew `aubio` |

## Optional (interactive, not driven by the agent)

| Tool | Use | Get it |
|---|---|---|
| **Friture** | Real-time spectrum and spectrogram while the noise is happening; good for finding the room and position where it is worst | AppImage / dmg / msi from github.com/tlecomte/friture/releases (v0.54, 2025-09). The PyPI package is stale (0.38, 2019); don't use it |
| **Room EQ Wizard (REW)** | Calibrated measurement with a USB measurement mic | roomeqwizard.com |
| **Audacity** | Manual listening, Plot Spectrum, cutting spans | apt/brew/winget `audacity` |

## Evaluated, not used

- **phonometry** (PyPI 3.3.0, needs Python ≥ 3.13): standards-conformant acoustics library (ECMA-418-1
  tone-to-noise ratio and prominence ratio, ISO 1996 adjustments in 4.0.0rc1). On
  2026-10-29 its `tone_to_noise_ratio` / `prominence_ratio` snapped to the wrong tone and
  returned a 3,043 dB ratio on a cycling 50 Hz hum. The prominence ratio is undefined
  below ~90 Hz anyway, where most indoor hums sit. Re-evaluate at 4.0 final.
- **sox `stat -freq`**: fixed ~47 Hz bins at 48 kHz, too coarse to separate mains harmonics.
- **sonic-annotator** + Vamp plugins: capable batch feature extractor, but no Ubuntu
  26.04 package and nothing here it does better than the tools above.

## Verified commands

```bash
# Probe
ffprobe -v error -show_entries format=duration,format_name:stream=codec_name,sample_rate,channels,bits_per_sample -of json in.wav

# Full-range spectrogram, log frequency (ffmpeg)
ffmpeg -v error -y -i in.wav -lavfi "showspectrumpic=s=1200x512:mode=combined:scale=log:fscale=log:legend=1" spec_full.png

# 0-1 kHz zoom spectrogram (sox): resample to 2 kHz first so the y-axis is 0-1 kHz
sox in.wav -n remix - rate 2k spectrogram -x 1200 -y 513 -z 80 -t "0-1 kHz" -o spec_lf.png
# 0-250 Hz for drones: rate 500

# Isolation: profile the background take, subtract it from the nuisance take
sox background.wav -n remix - noiseprof bg.prof
sox in.wav isolated.wav remix - noisered bg.prof 0.3     # 0.2-0.4; higher removes more and adds artefacts

# Listen to one band only (e.g. the drone)
sox in.wav band_40_160.wav remix - sinc 40-160

# Impacts / thumps: onset times in seconds
aubio onset -i in.wav -m specflux -t 0.3

# Whine pitch over time (time, Hz); 0 = no confident pitch
aubio pitch -i in.wav -m yin -u Hz -B 4096 -H 1024
```

On the test file, `noisered` with the background profile lowered the 1 kHz and 4 kHz
bands by ~50 dB and left the 50 and 100 Hz tones within 0.1 dB.

Pitch method: `yinfft` read a pure 100 Hz tone as 106.8 Hz; `yin` with `-B 4096 -H 1024`
read 50 Hz and 100 Hz exactly and a 3 kHz whine as 3,020-3,030 Hz. Use `yin`.
