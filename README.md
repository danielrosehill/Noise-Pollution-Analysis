# Noise Pollution Analysis

Agent skills for documenting a noise nuisance at home and analysing recordings of it at
the frequency level: which frequencies make it bothersome, whether it is tonal, cyclic,
impulsive or low-frequency, and what the measurements suggest about its source.

Stage 1 (this release): workspace, sample logging, spectral analysis and reports.
Stage 2 (planned): mitigation (masking, ANC, isolation) driven by the analysis.

## Skills

- `setup-noise-workspace`: create a workspace folder for noise sources, devices, samples
  and reports.
- `log-noise-sample`: add a recording with its metadata (device, app, audio source, room,
  mic position, conditions) and get advice on recording well.
- `spectral-analysis`: run the analysis script on a sample and write a frequency-level
  report into the workspace.

## Analysis script

`scripts/analyse_noise.py` runs standalone with `uv`:

```bash
uv run scripts/analyse_noise.py recording.wav --out analysis/ [--background quiet.wav] \
  [--start 5 --end 95] [--calibration-offset 94.0]
```

It writes `analysis.json` plus plots (`spectrum.png`, `thirds.png`, `spectrogram.png`,
`spectrogram_lf.png`, `level_time.png`) and reports:

- long-term average spectrum at ~0.5 Hz resolution;
- 1/3-octave band levels, unweighted and A-weighted;
- tonal peaks with prominence over the local spectral floor, grouped into harmonic
  families (mains hum, motors, fans);
- periodicity of the level envelope (compressor or pump cycling, music beat);
- L10/L50/L90, crest factor, clipping;
- per-band excess over a background recording, when one is supplied;
- warnings for lossy formats, low sample rates and phone low-frequency roll-off.

Levels are dBFS (relative) unless you pass a calibration offset.

## Recording tips

Use an unprocessed audio source. On Android, ASR Voice Recorder with the audio source
set to **Unprocessed**, WAV at 48 kHz. Place the phone on a soft surface, keep quiet,
record 1 to 5 minutes, and if possible record a background take with the nuisance
absent. Full guide: `references/recording-guide.md`.

## Requirements

`uv` and `ffmpeg`.

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install noise-pollution-analysis@danielrosehill
```

## Workspace layout

See `references/workspace-layout.md`. The workspace belongs to you and can live
anywhere; the plugin keeps only a pointer to it in
`${CLAUDE_USER_DATA:-~/.local/share/claude-plugins}/noise-pollution-analysis/config.json`.

## License

MIT
