---
name: isolate-nuisance
description: Isolate a noise nuisance from room background - subtract a background take with sox, extract a band, make before/after spectrograms.
---

# Isolate the nuisance from background

Produces a version of the recording with the room's steady background removed, and
optionally a band-limited version that keeps only the nuisance's frequencies. These are
used to listen to the nuisance on its own, to confirm what the analysis found, and as
cleaner input for `spectral-analysis`.

## Inputs

- **Nuisance take**: a sample folder or audio path.
- **Background take**: same spot, same device and gain, nuisance absent (`background_file`
  in `sample.md`). Without one, use a span of the nuisance take where the nuisance is
  off (e.g. between compressor cycles): ask the user, or find it from `level_time.png`.
  If there is no quiet span at all, skip subtraction and offer band extraction only.
- **Band** (optional): frequencies to keep, usually from the spectral analysis
  (e.g. 40-160 Hz for a drone with harmonics).

## Steps

1. Check `sox` is available; if not, run `install-analysis-tools`.
2. Build a noise profile from the background:
   ```bash
   sox "<background>" -n remix - noiseprof "<sample>/isolation/bg.prof"
   # or from a quiet span of the same take:
   sox "<recording>" -n remix - trim <start> <duration> noiseprof "<sample>/isolation/bg.prof"
   ```
3. Subtract it:
   ```bash
   sox "<recording>" "<sample>/isolation/isolated.wav" remix - noisered "<sample>/isolation/bg.prof" 0.3
   ```
   Start at 0.3. Use 0.2 if the result sounds watery or the nuisance itself loses level,
   and up to 0.4 if background remains.
4. If a band was given, extract it:
   ```bash
   sox "<sample>/isolation/isolated.wav" "<sample>/isolation/band_<lo>_<hi>.wav" sinc <lo>-<hi>
   ```
5. Make before/after spectrograms with the same settings so they compare directly.
   Pick the `rate` from the band of interest: `rate 2k` → 0-1 kHz, `rate 500` → 0-250 Hz,
   omit it for full range.
   ```bash
   sox "<recording>" -n remix - rate 2k spectrogram -x 1200 -y 513 -z 80 -t before -o "<sample>/isolation/before.png"
   sox "<sample>/isolation/isolated.wav" -n rate 2k spectrogram -x 1200 -y 513 -z 80 -t after -o "<sample>/isolation/after.png"
   ```
6. Check the result numerically: run
   `uv run "${CLAUDE_PLUGIN_ROOT}/scripts/analyse_noise.py" "<sample>/isolation/isolated.wav" --out "<sample>/isolation/analysis"`
   and compare the bands with the original's `analysis.json`. The nuisance bands should
   be within ~1 dB of the original, and the background bands should drop. If a nuisance
   band dropped by more than 3 dB, the background take contained the nuisance too; say
   so and lower the reduction amount or pick a cleaner quiet span.
7. Note in `sample.md` what was done (profile source, amount, band) and tell the user
   which file to listen to.

## Notes

- Spectral subtraction removes **steady** background. It does not separate two
  overlapping sounds that both vary (a neighbour's TV over traffic).
- Isolated audio is for listening and identification. Quote levels in reports from
  the original recording, with the background comparison, not from the processed file.

## Data storage

Writes into `<sample>/isolation/` only. Original audio is not modified.
