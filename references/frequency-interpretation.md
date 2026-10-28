# Interpreting the analysis

How to turn `analysis.json` into findings. State which findings are measured and which
are interpretation. A frequency match suggests a source; it does not prove one.

## Characterise the noise

| Signal in `analysis.json` | Character | Typical meaning |
|---|---|---|
| `tonal_peaks` with prominence ≥ 10 dB | **Tonal** (hum, whine, drone) | Rotating machinery, transformers, pumps, fans |
| `harmonic_families` present | **Harmonic series** | One machine: mains-related (50/60 Hz base) or motor/fan shaft rate |
| `periodicity.strength` ≥ 0.5 | **Cyclic / rhythmic** | Compressor or pump cycling (seconds to minutes), music beat (0.4-0.6 s), footsteps |
| `overall.crest_factor_db` > 20 and wide `l10_minus_l90_db` | **Impulsive / intermittent** | Banging, doors, impacts, footsteps, dropping objects |
| Smooth spectrum, no peaks, small L10-L90 | **Steady broadband** | Traffic hum at distance, ventilation airflow, waterfall-like |
| Most energy in `low_20_200` | **Low-frequency** | HVAC, traffic, bass music, lifts, pumps, transformers |

Tonal and low-frequency noise is far more annoying than its dB(A) figure suggests. Many
assessment methods add a penalty for tonal, impulsive or intermittent character.

## Frequency → likely sources (indoor nuisance)

| Frequency | Common culprits |
|---|---|
| < 20 Hz | Large fans, wind turbines, heavy industry, some HVAC. Phones rarely capture this. |
| 20-40 Hz | Large HVAC, ship/truck engines idling, sub-bass from music, lift machinery |
| 50 / 60 Hz and 100 / 120 Hz | Mains-frequency hum: transformers, ballasts and power supplies hum at **twice** mains (100/120 Hz) with harmonics; motors at mains rate |
| 25-50 Hz (single peak, not mains-related) | Motor or fan shaft rate (RPM / 60), e.g. 1,450 RPM ≈ 24 Hz |
| 40-120 Hz broadband, pulsing | Bass in neighbour's music, home cinema |
| 40-200 Hz with cycling | Refrigeration and AC compressors, heat pumps, pool/water pumps |
| 100-500 Hz | Traffic (tyres, engines), ventilation ducts, plumbing |
| Blade-pass tone | Fans: RPM / 60 × number of blades (often 100-500 Hz) |
| 500 Hz-2 kHz | Voices, TV, dogs, general household activity |
| 2-8 kHz whine | Electronics coil whine, ultrasonic-pest-device sidebands, small motors, bearings |
| > 8 kHz | Pest repellers, CRT/charger whine; often inaudible to older listeners |

Mains is 50 Hz in Europe, Israel, most of Asia, Africa and Australia; 60 Hz in North
America and parts of South America and Asia. Ask or infer from the user's location.

## Background comparison

If `background` is present, lead with `bands_elevated_6db_or_more`. Those bands are what
the nuisance adds; everything else is the room. An excess under ~3 dB is within the
noise of a phone recording.

## Caveats to always state

- Levels are **dBFS (relative)** unless a calibration offset was supplied. Compare
  bands and recordings made with the same device and settings; don't quote dBFS as dB SPL.
- **A-weighting** discounts low frequencies heavily (−30 dB at 50 Hz). A low-frequency
  drone can dominate perception while barely moving the A-weighted figure. Report both.
- **Phone LF roll-off:** see the `warnings` array.
- **Room modes:** a fixed position in a small room exaggerates some low frequencies and
  hides others. A second position helps confirm a low-frequency peak.

## Mitigation hints (stage 2)

Keep this short in the sample report; the mitigation stage builds on it.

- **Steady tonal or low-frequency (< ~1 kHz):** ANC headphones/earbuds work best here;
  masking works poorly (you'd need loud low-frequency masking noise). Building
  measures: mass, decoupling, sealing gaps; vibration isolation at the source.
- **Mid/high broadband or intermittent (voices, TV, traffic hiss):** masking
  (pink/brown noise, fan) is effective; earplugs; window seals and secondary glazing.
- **Impulsive (impacts, footsteps):** source-side measures (floor underlay) and
  structural decoupling; ANC and masking help little.
- **Narrow-band whine:** often a single fixable device; identify and isolate it.
