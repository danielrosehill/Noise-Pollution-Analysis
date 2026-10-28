# Noise workspace layout

A workspace is a plain folder the user owns, usually a git repo. Every skill in this
plugin reads and writes this layout. Nothing is stored inside the plugin.

```
<workspace>/
├── WORKSPACE.md                 what this workspace is for, the home/room context, index of sources
├── devices.md                   recording devices and apps, one section per device
├── sources/
│   └── <source-slug>/
│       └── source.md            one noise nuisance: what it is, when, where it is heard
├── samples/
│   └── <YYYY-MM-DD-HHMM>_<source-slug>/
│       ├── <original audio file(s)>     copied in, never modified
│       ├── sample.md                    recording metadata (frontmatter) + notes
│       ├── analysis/                    script output: analysis.json + PNGs
│       └── report.md                    spectral findings for this sample
└── reports/
    └── <YYYY-MM-DD>_<source-slug>_summary.md   cross-sample findings per source
```

## `sources/<slug>/source.md`

```markdown
---
slug: neighbour-hvac
name: Neighbour's rooftop HVAC unit
first_noticed: 2026-09-01
status: active            # active | resolved | intermittent
---

## What it sounds like
Low droning hum, "felt in the chest", with a whine on top.

## When
Evenings 18:00-02:00, worse in summer. Cycles on and off every few minutes.

## Where it is heard
Bedroom (north wall) worst; living room faint. Louder with windows closed (suggests
structure-borne low frequency).

## Suspected source and direction
Rooftop unit on the building to the north, ~15 m away.

## Impact
Prevents sleep. Not audible to everyone in the household.
```

## `samples/<id>/sample.md`

```markdown
---
sample_id: 2026-10-28-2215_neighbour-hvac
source: neighbour-hvac
recorded_at: 2026-10-28T22:15:00+02:00
file: rec_20261028_2215.wav
background_file: rec_20261028_2230_quiet.wav   # same spot, nuisance absent; optional
device: pixel-8                                # key into devices.md
app: ASR Voice Recorder
audio_source: unprocessed                      # unprocessed | mic | voice-recognition | camcorder | unknown
format: wav 48000 Hz 16-bit mono
room: bedroom
mic_position: on bed, 1 m from north wall, mic facing wall
windows: closed
doors: closed
own_appliances_off: [fridge, AC]
duration_s: 120
calibration_offset_db: null                    # dB to add to dBFS for dB SPL, if calibrated
analysed: false
---

## Notes at time of recording
Hum steady, compressor kicked in at ~0:40.
```

## `devices.md`

One `## <device-key>` section per device: make/model, OS, app and version, audio
source setting, sample rate, known low-frequency roll-off (if measured), calibration
offset (if measured against a sound level meter).
