---
name: spectral-analysis
description: Spectral analysis of a noise recording - dominant frequencies, hums, tones, harmonics, low-frequency drone, cycling; writes a report.
---

# Spectral analysis of a noise recording

Finds which frequencies make an indoor noise nuisance bothersome and writes a
technical report into the sample folder. Works on a logged sample or on any audio path
the user gives.

## Inputs

- **Recording**: a sample folder (`samples/<id>/`) or an audio file path. If a bare file
  is given and a workspace exists, offer to log it first with `log-noise-sample`, but
  analyse it anyway if the user declines.
- **Background recording** (optional): same spot, nuisance absent. Read
  `background_file` from `sample.md` if set.
- **Span** (optional): `--start`/`--end` seconds, to analyse only the part where the
  nuisance is present or to cut handling noise at the ends.
- **Calibration offset** (optional): dB to convert dBFS to dB SPL, from `sample.md` or
  `devices.md`. Without it, all levels are relative.

## Requirements

`uv` and `ffmpeg` on PATH. The script declares its own Python dependencies (numpy,
scipy, soundfile, matplotlib); `uv run` installs them in an isolated environment.

## Steps

1. Run the analysis script, output into the sample's `analysis/` folder (or a folder
   next to the file for a bare path):
   ```bash
   uv run "${CLAUDE_PLUGIN_ROOT}/scripts/analyse_noise.py" "<recording>" \
     --out "<sample>/analysis" [--background "<bg>"] [--start S --end S] [--calibration-offset DB]
   ```
   If `CLAUDE_PLUGIN_ROOT` is unset (skill read outside an installed plugin), run it
   straight from GitHub:
   `uv run https://raw.githubusercontent.com/danielrosehill/Noise-Pollution-Analysis/main/scripts/analyse_noise.py ...`
   The script prints a short JSON summary; the full numbers are in `analysis/analysis.json`.
2. Look at `analysis/spectrogram_lf.png` and `analysis/spectrum.png` (Read them as images)
   to confirm what the numbers say, e.g. whether a tone is steady or comes and goes, and
   whether energy at one moment is a handling thump rather than the nuisance. If a
   span is contaminated, re-run with `--start`/`--end`.
3. Interpret with `${CLAUDE_PLUGIN_ROOT}/references/frequency-interpretation.md`:
   character (tonal, harmonic, cyclic, impulsive, broadband, low-frequency), likely
   source mechanisms for the measured frequencies, and the caveats that apply. Use the
   source description in `sources/<slug>/source.md` ("cycles every few minutes",
   "worse with windows closed") to support or question the match.
4. Write `report.md` in the sample folder (template below). Every frequency claim cites a
   number from `analysis.json`. Mark interpretations as such.
5. Set `analysed: true` in `sample.md`. If the source has two or more analysed samples,
   offer to write or update `reports/<YYYY-MM-DD>_<source-slug>_summary.md` comparing
   them (consistent peaks across samples and rooms are the strongest evidence).
6. Tell the user the 2-3 headline findings and the report path.

## Report template

```markdown
# Spectral analysis: <source name>, <sample_id>

**Recording:** <file>, <duration> s, <sample rate> Hz, <device / app / audio source>
**Background reference:** <file or "none">
**Units:** <dBFS (relative) | dB SPL (calibrated, offset X dB)>

## Headline findings
1. <the frequency or band that most characterises the nuisance, with level and prominence>
2. <character: tonal / cyclic / impulsive / broadband, with the evidence>
3. <likely mechanism, marked as interpretation>

## Frequency detail
| Frequency / band | Level | Prominence or excess over background | Notes |
|---|---|---|---|

## Character
Tonality, harmonic families, periodicity, L10/L50/L90 spread, crest factor.

## Weighted vs unweighted
Loudest 1/3-octave band unweighted vs A-weighted; what A-weighting hides here.

## Data quality and caveats
Warnings from the script, recording conditions that limit the result, what a better
take would add.

## Implications for mitigation
Two or three lines pointing to the mitigation class the profile suits.

## Figures
![Spectrum](analysis/spectrum.png) ![1/3 octaves](analysis/thirds.png)
![Spectrogram 0-500 Hz](analysis/spectrogram_lf.png) ![Level over time](analysis/level_time.png)
```

## Notes

- The script uses a ~0.5 Hz FFT resolution (2 s Welch segments), so it separates mains
  harmonics and close motor tones; it needs at least a few seconds of audio.
- Tonal peaks are reported when they stand 8 dB or more above the local 1/3-octave
  spectral floor; weaker sidebands within 3% of a stronger peak are merged into it.
- A phone recording that shows nothing below ~50 Hz has not shown that nothing is there.
  Say so whenever the user describes a drone or a "felt" noise.

## Data storage

Writes `analysis/` and `report.md` into the sample folder and, on request, a summary
in `reports/`. Never modifies the audio files.
