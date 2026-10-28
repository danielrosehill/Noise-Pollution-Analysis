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

`ffmpeg`, `sox` and `uv` (core), `aubio` (recommended). If any core tool is missing, run
`install-analysis-tools` first. Commands, versions and test results:
`${CLAUDE_PLUGIN_ROOT}/references/toolchain.md`.

## Steps

1. **Probe** the file with `ffprobe` (codec, sample rate, duration) and note lossy
   formats or sample rates under 44.1 kHz. Work in `<sample>/analysis/` (or a folder next
   to a bare file). If the file is not WAV or FLAC, decode it once and use the WAV as
   `<rec>` for sox and aubio (sox can't tell the length of an m4a, which spoils its plots):
   `ffmpeg -v error -i "<file>" -c:a pcm_s24le "<out>/decoded.wav"`
2. **Spectrograms from the CLIs**, so you can see the nuisance before measuring it:
   ```bash
   ffmpeg -v error -y -i "<rec>" -lavfi "showspectrumpic=s=1200x512:mode=combined:scale=log:fscale=log:legend=1" "<out>/cli_spec_full.png"
   sox "<rec>" -n remix - rate 2k spectrogram -x 1200 -y 513 -z 80 -t "0-1 kHz" -o "<out>/cli_spec_1k.png"
   sox "<rec>" -n remix - rate 500 spectrogram -x 1200 -y 513 -z 80 -t "0-250 Hz" -o "<out>/cli_spec_250.png"
   ```
   Read the PNGs as images. Note steady horizontal lines (tones and hums), lines that come
   and go (cycling machinery), vertical stripes (impacts), and handling noise at the ends
   of the take to exclude with `--start`/`--end` in step 4.
3. **Time-domain events with aubio** (skip if aubio is missing and say so in the report):
   ```bash
   aubio onset -i "<rec>" -m specflux -t 0.3 > "<out>/onsets.txt"       # impact times (s)
   aubio pitch -i "<rec>" -m yin -u Hz -B 4096 -H 1024 > "<out>/pitch.txt"  # time, Hz
   ```
   Onsets at regular intervals mean rhythmic impacts (footsteps, bass beat, machinery).
   For pitch, summarise the stable values (ignore 0 and isolated jumps). A tone that holds
   steady is fixed-speed machinery or mains hum; one that drifts is variable-speed.
4. **Measurements** with the plugin's script: spectrum, 1/3 octaves (unweighted and
   A-weighted), tonal peaks, harmonic families, envelope periodicity, background excess:
   ```bash
   uv run "${CLAUDE_PLUGIN_ROOT}/scripts/analyse_noise.py" "<rec>" \
     --out "<out>" [--background "<bg>"] [--start S --end S] [--calibration-offset DB]
   ```
   If `CLAUDE_PLUGIN_ROOT` is unset (skill read outside an installed plugin), run it
   from GitHub:
   `uv run https://raw.githubusercontent.com/danielrosehill/Noise-Pollution-Analysis/main/scripts/analyse_noise.py ...`
   The full numbers go to `<out>/analysis.json`. Cross-check them against what steps 2
   and 3 showed. If they disagree, look again before reporting.
5. **Isolation** (optional): if a background take exists and the user wants to hear the
   nuisance on its own, run `isolate-nuisance`.
6. Interpret with `${CLAUDE_PLUGIN_ROOT}/references/frequency-interpretation.md`:
   character (tonal, harmonic, cyclic, impulsive, broadband, low-frequency), likely
   source mechanisms for the measured frequencies, and the caveats that apply. Use the
   source description in `sources/<slug>/source.md` ("cycles every few minutes",
   "worse with windows closed") to support or question the match.
7. Write `report.md` in the sample folder (template below). Every frequency claim cites a
   number from `analysis.json`. Mark interpretations as such.
8. Set `analysed: true` in `sample.md`. If the source has two or more analysed samples,
   offer to write or update `reports/<YYYY-MM-DD>_<source-slug>_summary.md` comparing
   them (consistent peaks across samples and rooms are the strongest evidence).
9. Tell the user the 2-3 headline findings and the report path.

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
Tonality, harmonic families, pitch stability (aubio), impact timing (aubio onsets), periodicity, L10/L50/L90 spread, crest factor.

## Weighted vs unweighted
Loudest 1/3-octave band unweighted vs A-weighted; what A-weighting hides here.

## Data quality and caveats
Warnings from the script, recording conditions that limit the result, what a better
take would add.

## Implications for mitigation
Two or three lines pointing to the mitigation class the profile suits.

## Figures
![Spectrum](analysis/spectrum.png) ![1/3 octaves](analysis/thirds.png)
![Spectrogram full range](analysis/cli_spec_full.png) ![Spectrogram 0-1 kHz](analysis/cli_spec_1k.png)
![Spectrogram 0-250 Hz](analysis/cli_spec_250.png) ![Level over time](analysis/level_time.png)
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
